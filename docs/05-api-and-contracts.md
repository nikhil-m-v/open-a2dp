# Local API and contracts

Status: proposed M0-WP02 design, 2026-10-30 (Asia/Calcutta). These are semantic contracts for future implementation, not an implemented API or frozen wire/driver ABI. The [PRD](01-project-requirements.md), [system lifecycle](15-system-design.md), and [security controls](11-security.md) remain authoritative. See also [architecture](03-architecture.md), [UI flows](02-ui-ux-design.md), and [storage](04-database-and-storage.md).

## Contract surfaces

| Boundary | Permitted purpose | Excluded authority |
| --- | --- | --- |
| Desktop to local control service | Authenticated inventory, ownership, state, explicit apply/stop, bounded diagnostics. | Driver installation, arbitrary commands/files, another user's settings or audio. |
| Service to encoder worker | Start/stop one configured stream, bounded codec input/output and counters. | Package installation, loading a client-selected DLL, changing binding. |
| Service/worker to driver | Narrow validated control and scoped media exchange, subject to M3/M4 experiments. | General kernel access, arbitrary pointers, global stack modification. |
| User to independent elevated helper | Explicit target/package plan, install/update/restore, safe cancellation and recovery inspection. | Generic elevated proxy; trusting service/UI claims as evidence of binding or package integrity. |

The control plane uses local named pipes, with no HTTP/TCP listener. PCM, encoded frames, and high-rate telemetry do not use JSON control messages. Media buffer format, transport IOCTLs, endpoint exchange, numeric capacities and exact identities require the M1/M3/M4 gates; this document creates no kernel structures or opcode values.

## Authentication and compatibility

The service owns pipe creation and explicit DACLs; reject remote callers and authenticate the peer's Windows token, logon identity and session. A client-supplied SID, process ID, target name or lease token never establishes identity. Prefer explicit rights: Microsoft documents that generic write access also grants pipe-instance creation rights. [Named pipe security](https://learn.microsoft.com/en-us/windows/win32/ipc/named-pipe-security-and-access-rights)

Derive caller identity through a checked OS authentication path. If impersonation is used, check success and revert on every exit path; failed authentication denies the request. No work proceeds as the service merely because impersonation failed. [Named pipe impersonation](https://learn.microsoft.com/en-us/windows/win32/ipc/impersonating-a-named-pipe-client)

Before disclosing preferences or sending commands, the client verifies the pipe server against the installed service identity and service-owned endpoint. A pipe server PID is an available observation, not proof by itself: the implementation must handle process replacement and validate the service association without a check/use race. The concrete mechanism and fake-server fixtures must pass SEC-002 before shipping IPC. [Server process observation](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-getnamedpipeserverprocessid)

Proposed handshake: `Hello` exchanges supported protocol major/minor ranges, build identity, and feature identifiers after peer validation. `Welcome` returns the selected version, supported operations/features, `serviceEpoch`, and the enforced limit profile. Limits are server-owned, cannot be raised by a client, and must be known before allocating the body of an unauthenticated/initial message. Exact values and the executable schema belong to M1-WP02.

- A major mismatch permits only a bounded incompatibility response and closure. No mutations, optimistic downgrade, or partial interpretation.
- A minor version adds optional fields/operations; negotiate the common subset. New required behavior needs a major change or an explicitly negotiated feature. Missing features disable the affected action with a reason.
- Requests use a strict schema: reject unknown operation names/enums, missing required fields and unexpected mutation fields. Optional extension fields are allowed only where the negotiated schema explicitly defines them. Unknown response fields may be ignored only if doing so cannot alter action availability; unknown state values render as unavailable and disable mutations.
- API version, preference schema version, helper journal version, package compatibility and driver interface version are distinct. A compatible desktop handshake does not prove a compatible driver/codec set.
- The independent helper has its own supported package/journal versions and local interface. Recovery remains inspectable with the service absent or incompatible; unsupported protected state allows diagnosis, not guessed privileged replay.

## Identity and ordering

| Field | Meaning and validation |
| --- | --- |
| `targetId` | Opaque local key mapped to a canonical OS device by the service/helper. Never a display name, raw client path, authorization token or claim of immutable hardware identity. Re-resolve before mutation; ambiguous/missing mappings fail closed. |
| `serviceEpoch` | New opaque identity on each service start. Reject earlier epochs; old leases, operations and active state cannot be adopted after restart. |
| `controllerLease` | Service-issued reference bound to the authenticated user/logon/session and selected target. It authorizes only while that OS identity still owns the lease. Never persisted or accepted as a bearer credential. |
| `connectionGeneration` | Service-owned generation invalidated by disconnect, removal, suspend or new connection. Capability observations carry this generation and freshness. |
| `streamGeneration` | Distinguishes individual streams within a connection, including track/app boundaries and reconfiguration. Late completion cannot mutate a later stream. |
| `settingsRevision` | Version of the user's saved/requested preferences, with compare-and-replace semantics in storage. Correlates an apply result to its input; not an authorization or global service-state token. |
| `snapshotSequence` | Monotonic, nonreused revision of decision-relevant state within the service epoch. Ownership, lifecycle, capability or negotiated-state changes advance it. Exhaustion requires a fresh epoch and resynchronization, never wraparound. High-rate counter samples use a separate telemetry sequence and do not continuously invalidate Apply. |
| `requestId` | Client-created unique correlation identity scoped to authenticated caller and service epoch. The client must never reuse it; the server detects changed-payload reuse while the request record is retained. Stale-state checks still apply after record expiry. |
| `operationId` | Service/helper-issued identity for one accepted operation. Helper operation identity is durable in its protected journal; service operation identity lasts only for its service epoch. |

Commands that mutate stream/ownership state include expected `serviceEpoch`, `snapshotSequence` and applicable target/generations; all except initial lease acquisition include the current lease. The service validates these and the caller immediately before accepting the transition. It serializes competing transitions and advances the snapshot before admitting another conflicting command. A capability refresh in progress makes configuration unavailable.

The lease belongs to the authenticated OS session, not a particular UI window or pipe handle. Closing/crashing the UI does not stop healthy playback. A replacement UI in the same authenticated logon/session can resynchronize and recover its lease reference. Different sessions, including the same account in another logon, cannot seize it. Logoff or explicit release stops and drains the old stream before reassignment; session locking/switching alone does not imply release. Service failure invalidates the lease and triggers orphan cleanup. A service restart reconstructs observations, never persisted active playback.

## Operations and authorization

Names below identify proposed operations; no callable endpoint exists yet. Read access is not universal: use the caller's visibility policy, omit other users' preferences and diagnostics, and expose only a generic ownership-conflict reason to nonowners.

| Operation | Caller and prerequisites | Result / side effect |
| --- | --- | --- |
| `GetInventory` | Authenticated permitted local session; compatible handshake. | Read-only target/radio/endpoint observations with unknown/unsupported reasons; no binding or capability negotiation side effect. |
| `GetSnapshot`, `SubscribeState` | Authenticated permitted local session, filtered by ownership. | Consistent current snapshot and bounded events. No raw OS IDs from another user's scope or historical audio. |
| `AcquireController` | Permitted local interactive session; expected epoch/state; no current controller. | Lease for one selected target. If occupied, return conflict; never implicit takeover or installation. |
| `ReleaseController` | Current controller and matching expectations. | Stop, drain and release; incomplete cleanup blocks reassignment. |
| `RefreshCapabilities` | Current controller; validated project binding, compatible idle/discovery state. | Bounded fresh protocol query; stale data stays labeled until completion. Does not bind a device. |
| `ApplySettings` | Current controller; full saved requested values with revision correlation, matching service-state expectations, fresh remote capability intersection, compatible driver/build. | Accepted stop/configure/restart operation or typed rejection. Saving is separate; no implicit cross-codec fallback. |
| `StopStream` | Current controller and current target/generations. | Idempotent stop of that stream; a stale stop cannot stop a replacement stream. |
| `GetOperation`, `RequestCancel` | Initiating authenticated owner and operation identity. | Current result or cancellation request, honored only at declared safe boundaries. |
| `GetDiagnostics` | Current controller or own permitted diagnostic scope. | Bounded sanitized records/counters. User-side export saves only after inspection; never asks the service to write a client-selected path. |

`ApplySettings` sets the explicit selected codec/format/manual quality policy. It does not start an arbitrary audio source. If the endpoint still requests playback after successful reconfiguration, the service may start a fresh stream under the existing authorized playback session; otherwise it reports configured and stopped. Initial controlled-source playback remains a separate lab harness contract in M3. Clearing a saved preference or changing a UI selection is never an installation action.

## Requested, observed and active models

Every relevant observation has a value or explicit `unknown`/`stale`/`unsupported` status, origin, observation time, and applicable generations. Wall-clock timestamps aid diagnosis; monotonic clocks govern deadlines. An unavailable query is not an empty list of codecs and not evidence of absent support.

The UI combines client-local preferences with an authoritative service snapshot. Keep the origins explicit:

- Client-local user-requested codec, transport format and manual quality, plus saved revision and whether edits are unsaved. The UI owns these fields and local persistence; the service snapshot contains only the captured submitted request/revision for its operation. The service does not independently read or validate the current preference file.
- Remote-advertised capabilities; locally implemented codecs; installed-build distribution eligibility; supported intersection and reason for exclusion. Disabled/research codecs cannot be enabled by a client override.
- Windows PCM format and endpoint identity; selected microphone; observed Classic profile; intended and observed transport owner. Each can independently be unknown. HFP, a USB transmitter, a general decoder list or a PCM setting does not establish LDAC support.
- Current negotiated configuration and its provenance/generations; actual stream lifecycle and observable progress. Store last verified configuration separately for diagnosis; when stopped/disconnected it is never labeled current active audio.
- Routing/endpoint identity changes, verified mapping when available, and instructions needed to reattach app/EQ/APO settings. Do not rewrite third-party configuration.
- Counters with units, sample interval, availability and reset identity. Configured bitrate, encoded-byte rate, transport throughput and end-to-end latency remain distinct; missing metrics are not zero.

Example semantic payload, using synthetic placeholders rather than a wire-schema fixture:

```json
{
  "operation": "ApplySettings",
  "requestId": "example-request",
  "expected": {
    "serviceEpoch": "example-service",
    "snapshotSequence": 12,
    "targetId": "example-target",
    "connectionGeneration": 3,
    "streamGeneration": 7
  },
  "controllerLease": "example-lease",
  "settingsRevision": 4,
  "requested": { "codecId": "sbc", "parameters": {} }
}
```

The example makes no SBC-parameter, packet-layout or quality-support claim. An empty parameter object is illustrative, not permission to invent negotiation defaults. Implementation replaces placeholders with the bounded schema and codec-specific validated configuration. Only a terminal success plus an authoritative snapshot confirms application of settings; receipt, saving and operation acceptance do not.

## Apply, retry and event rules

1. V1 Apply uses successfully saved settings; an unsaved draft requires the explicit Save-and-apply action. Before dispatch, the UI checks revision/content through its persistence component and captures the complete saved request. Saving failure prevents dispatch. The service receives these immutable values and revision correlation against a fresh service snapshot; it cannot observe a later local Save. A later Save leaves the accepted operation unchanged and the UI displays saved/applied divergence.
2. Service validates identity, lease, binding, compatibility, freshness, bounds and the supported intersection before changing the stream. Rejection leaves existing playback untouched when it is still valid.
3. Once accepted, the service captures immutable request values and enters the lifecycle's stop/drain/configure/restart path. A second conflicting mutation returns `Busy`; it never interleaves two configurations. Capability changes/removal invalidate the operation before further work.
4. After negotiation, publish the actual configuration. A post-stop failure reports stopped/faulted state and retains the user's request plus last verified history. No automatic SBC switch and no false active-state rollback. Explicit retry uses fresh observations.
5. Cancellation is a request, not a terminal success. Report `cancelRequested`, `canCancel` and the current stage; complete only after cleanup at a safe boundary. A disconnected UI or response timeout does not imply cancellation.

Deduplicate requests within a bounded retention window by caller, epoch, request ID and canonical payload. An exact repeat returns its original operation/result, with no repeated side effect. While the record is retained, a changed payload with the same ID returns `RequestConflict`. Retain accepted in-flight requests until terminal; bound admission rather than evicting active operations. The server advertises result-retention limits. Once a result has expired, its query returns `OutcomeUnknown`; replay still carries the original expected snapshot, which an accepted mutation has already advanced. Never silently refresh those preconditions on a retry. Reject stale replays, resynchronize, and require a new explicit action. Never blindly resend after service restart. A fresh action is revalidated against current state, not treated as permission to repeat an uncertain privileged step. M1 fixtures must exercise expiration and duplicate side effects as well as the cache-hit path.

Subscribe with a full snapshot and a defined event cursor, then accept only increasing events for the same epoch. The executable contract must make snapshot/subscription acquisition race-free. Gap, overflow, out-of-order event, generation mismatch or reconnect requires a fresh snapshot; queued old events cannot make controls appear ready. Per-subscriber queues are bounded and a slow viewer may lose diagnostics or be disconnected without blocking audio. Operation completion remains queryable within its retention window; privileged completion comes from the durable helper journal.

## Errors and uncertainty

Errors carry a stable code, user-facing explanation key, affected operation/stage, applicable epoch/generations, and an allowlisted next action. Optional underlying OS codes are diagnostic and redacted. Do not expose pointers, arbitrary paths, another user's identity, secrets or audio. Localization uses message keys; clients branch on codes rather than English text.

| Code family | Required response |
| --- | --- |
| `AccessDenied`, `OwnershipConflict` | No mutation; no cross-session details; do not retry with elevated playback UI. |
| `VersionMismatch`, `FeatureUnavailable` | Disable affected actions; show compatible package/recovery guidance. |
| `InvalidRequest`, `LimitExceeded`, `UnsupportedSetting` | Reject before side effects; identify the bounded field or reason without echoing unsafe content. |
| `StaleState`, `StaleGeneration`, `CapabilitiesUnknown` | Refresh snapshot/capabilities through permitted operations; never apply a stale request automatically. |
| `Busy`, `RequestConflict` | Keep current operation; query its state or require a new explicit action. |
| `DeviceRemoved`, `WorkerFailed`, `NegotiationFailed` | Stop/clean up as required, invalidate active state, expose the actual result. |
| `TimedOut`, `OutcomeUnknown` | Inspect authoritative operation/state; do not infer rollback or repeat side effects. |
| `BindingDrift`, `RecoveryRequired` | Block streaming/configuration until safely resolved; offer independent helper and offline instructions. |
| `StorageUnavailable` | Show that persistence/journaling failed. Failed required journal write prevents the next privileged mutation. |

Unknown errors use a conservative unavailable/faulted presentation with a correlation ID. They never convert to success or trigger a privileged repair.

## Independent helper contract

The helper supports fixed plan/inspect/execute/restore/cancel operations for an exact validated target and reviewed package identity. It resolves package material from protected staging and checks current OS binding itself; no arbitrary command, registry key, INF path or DLL name is accepted from preferences or the service. Elevation is requested only for an explicit privileged action, with the affected target and planned change visible.

The plan records prior binding, intended owner, observed owner, target and non-target endpoint baselines, package versions, compatibility and required recovery prerequisites. Plan identity is bound to those inputs. Revalidate at execution; a changed target/package/binding invalidates the plan. Machine-wide serialization and the service/driver quiescence handshake prevent install/restore from racing playback. If the service is unavailable, the independent path must prove quiescence through validated driver/OS mechanisms before mutation; M3 must establish these mechanisms.

Operation progress exposes `operationId`, operation kind, current stage, stage sequence, start/last-progress observations, measured deadline policy, `canCancel`, `cancelRequested`, reboot requirement and outcome certainty. A UI timer or lack of bytes is not invented percent completion. The [protected journal](04-database-and-storage.md) records intent before a side effect and verified outcome afterward. A watchdog may classify stalled/unknown, but cannot kill an in-flight PnP change or declare rollback. Restart reconciliation compares durable intent/outcome with live OS state before choosing a permitted next step.

Completion distinguishes **binding restored**, **restart required**, **playback verification pending**, and **recovery verified**. Successful package API exit alone cannot produce the final state. TC-007/020/022 verify the target and non-target playback/capture, including the required post-reboot cases, on the lab. The public projection omits private raw device identities and recovery material. Cancellation/query authorization is bound to the initiating authenticated session; later recovery by an explicitly elevated operator is a new recovery action, not adoption of the former user's stream lease.

## Implementation gates and traceability

| Gate / planned check | Owning package and coverage |
| --- | --- |
| Executable control schema, bootstrap/body/depth/string limits, ID encoding, bounded in-flight counts, deduplication retention, event cursor and gap behavior; malformed, replay and stale fixtures. | M1-WP02, SEC-001/002/003/005; RQ-002/004/008; TC-002/004/008. No transport hardware result implied. |
| Actual pipe ACL/server verification, token/session lifecycle and helper authorization implementation. | M3/M6/M7 as applicable; SEC-001/002/006; TC-007/008/018/021/022. |
| Driver/media ABI, safe cleanup and install-versus-playback exclusion; transport ownership and independent restoration evidence. | M3/M4, Q-001 through Q-003; SEC-003/004/005/006; TC-003/007/009/011/016. |
| Track/app boundaries fence stream callbacks; manual quality has no hidden adaptation; fresh capability checks explain PCM/profile/codec separately. | RQ-019/023/026; TC-019/023/026; REF-007/008/010/011/012/013/014/015/016/017. |
| Helper baseline and intended/observed owner, persistent stages, safe cancel, independent offline recovery and endpoint routing guidance. | RQ-020/021/022/024/025; TC-020/021/022/024/025; REF-001/002/003/004/005/006/009/018/019/020/021/022/023. |
| Exact log/export retention, stage deadlines and numeric resource budgets before their implementation; privacy, storage and stalled-client procedures. | M0-WP04 procedure design, M3 measured install deadlines, M4 media budgets, M6 persistence/export, M7 performance gates; SEC-007; TC-006/013/015. |

All listed acceptance checks are planned and unexecuted. Batch 2 resolves contract responsibilities and error semantics; it does not demonstrate codec support, implement a parser, select dependency versions or authorize driver deployment.
