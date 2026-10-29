# Database and local storage

Status: proposed M0-WP02 design, 2026-10-30 (Asia/Calcutta). No storage code, migration, ACL, recovery operation, or runtime test has been implemented. This document defines storage responsibilities; exact serialized schemas and limits must be reviewed before their implementation packages. See [architecture](03-architecture.md), [API contracts](05-api-and-contracts.md), [UI/UX](02-ui-ux-design.md), [system design](15-system-design.md), and [security](11-security.md).

## Storage decision and ownership

Use local versioned files. A database server, cloud account, activation database, and network service are unnecessary for the planned single active output. Local files must still have explicit concurrency, durability, validation, and recovery rules; choosing JSON does not provide these properties by itself.

The following are proposed logical locations, resolved through the owning Windows identity's known folders. They are not caller-supplied paths and are not created by this documentation package.

| Store | Proposed location and owner | Permitted contents and readers |
| --- | --- | --- |
| Preferences | `%LOCALAPPDATA%\OpenA2DP\settings.json`; per-user UI persistence component | UI choices and per-target requested settings. Only that OS user's authorized processes write; another user's UI cannot read or mutate them through the service. |
| Preference recovery copy | Same protected user directory; one bounded previous valid revision plus a bounded diagnostic quarantine under the future retention policy | Recovery from interrupted writes or an explicit reset. Same privacy and ACL rules as preferences; no indefinite backup chain. |
| Installation/recovery journal | `%PROGRAMDATA%\OpenA2DP\Recovery\`; short-lived elevated helper | Exact target/package/binding records and operation progress. Only the helper's authorized machine role writes; the service and user receive a filtered status projection through authenticated interfaces where needed. |
| Local diagnostics | Separate per-user UI and protected service/helper diagnostic directories under their owned roots | Bounded structured events and counters. No raw audio, credentials, link keys, unrestricted packet captures, or globally correlatable export identifiers. |
| Support export | Destination selected and written by the unelevated UI | Explicitly generated, redacted, inspectable bundle. No automatic upload and no elevated write to an arbitrary user-selected destination. |

The codec worker has no authority to write preferences or the privileged journal. The service receives validated requested settings through the control API; it neither opens a client-supplied settings path nor persists user files on the caller's behalf. The helper does not import a user preference file as an installation plan. Each component validates data crossing its own boundary even when another component already parsed it.

No store contains reusable administrator credentials, signing keys, disk-encryption recovery material, Bluetooth pairing keys, entitlement checks, or a hardware activation fingerprint. Offline recovery media and sensitive recovery material remain privately managed outside application logs, exports, and this repository.

## Per-user and per-target preferences

`schemaVersion` identifies the on-disk settings schema independently of control API versions. `settingsRevision` increases when a complete preference file is committed successfully. It correlates a requested settings snapshot with later apply/status results; it is neither authorization nor a service concurrency token. API mutations additionally check the current service state and generations.

| Field/group | Meaning and validation |
| --- | --- |
| `schemaVersion` | Positive supported schema identifier. A newer unsupported version is preserved without rewriting. Migration rules are explicit for every older accepted version. |
| `settingsRevision` | Per-user file revision, changed only on a successful complete write. Range/overflow behavior is fixed before implementation; never wrap into a previously used revision. |
| UI preferences | Allowlisted display/accessibility choices and optional last-selected target. No executable, DLL, command, registry, service, or privileged path fields. |
| `devices[].targetId` | Opaque local target key issued by the inventory/resolution boundary. It is a lookup hint for preferences, not a driver binding identity or proof of ownership. No display-name matching or automatic reassignment to a similar device. |
| `devices[].requested` | Explicit codec identity and typed codec-specific requested parameters. Values are checked against the installed implementation and fresh queried capabilities again at Apply. Unsupported saved choices remain visible as unavailable. |
| Manual quality | Store the explicit manual preference where supported. V1 has no enabled adaptive-quality policy. Future adaptation requires its own versioned schema, opt-in, and tests; disabling it retains the saved manual choice. |

Illustrative JSON only; `example-target` is synthetic, the values assert no available codec or receiver, and the example is not an executable schema or test fixture:

```json
{
  "schemaVersion": 1,
  "settingsRevision": 3,
  "ui": {
    "lastSelectedTargetId": "example-target"
  },
  "devices": [
    {
      "targetId": "example-target",
      "requested": {
        "codecId": "sbc",
        "parameters": {}
      }
    }
  ]
}
```

The implementation package must define required parameters and explicit defaults for each supported codec; an empty example parameter object does not authorize implicit negotiation choices. No codec selection or capability can be inferred merely from the presence of this file.

The inventory implementation must define how opaque target keys are resolved across restarts and rediscovery, record the evidence needed for a match, and test ambiguity and identity changes. Windows/device identifiers are not assumed permanently stable. If the current target cannot be matched unambiguously, mark the saved preference orphaned and ask the user to select a current target. Reassociation may copy editable requested choices after explicit user selection, but does not transfer installation authority, a stream lease, or proof of capabilities. Changing hardware causes rediscovery, never activation failure.

Do not persist `controllerLease`, `serviceEpoch`, `connectionGeneration`, `streamGeneration`, or an active configuration as resumable authority. These belong to current runtime state. A UI restart obtains a fresh authenticated snapshot. Historical diagnostics may retain bounded, appropriately scoped correlation values, but cannot be replayed to adopt a stream. Bluetooth link, endpoint, active profile, selected input, PCM format, and negotiated codec remain separate live observations; a saved request is not any of them.

## Write, migration, and read rules

The unelevated persistence component serializes writes for the same OS user using a user-scoped cross-process lock with an explicit ACL. It uses a bounded lock wait and reports a conflict instead of blocking audio or retrying forever. The service does not participate in this file lock or treat its revision as global state ordering.

For a normal edit:

1. Read the current file within parser limits and retain its revision and content identity. Validate the whole candidate against the exact supported schema before writing. Duplicate keys, duplicate target entries, unknown mutation-like fields, invalid types/ranges, and excessive size/depth/counts are rejected under the schema's explicit policy.
2. Acquire the per-user write lock and re-read. If the revision or content changed, return a conflict and reload; do not silently merge incompatible device requests or overwrite another window's edit. A content digest can detect same-revision manual edits; it is not a security credential and need not appear in logs.
3. Create a uniquely named temporary file under the verified same directory, apply the intended ACL, write the entire validated next revision, flush as required by the selected durability implementation, and close the write handle. Neither a pre-existing temporary path nor a reparse target is trusted.
4. Replace the destination using a tested same-volume replacement strategy and keep at most the bounded prior valid recovery copy. Never truncate the live destination before the replacement is ready. The chosen Windows API, metadata/ACL behavior, and crash windows must be documented and tested in M6-WP02.
5. Report Save success only after the replacement succeeds. A write failure leaves the last valid file available where the filesystem permits and reports unsaved edits. Recovery and disk-full paths must not trigger Apply or change driver ownership.

Save and Apply are distinct operations. Save records a request. In v1, Save and apply must save successfully before dispatching Apply. The UI checks the complete saved requested values, `settingsRevision`, and content identity under the per-user write lock when preparing the request. A conflicting edit requires reload and a new explicit action; unsaved values cannot be dispatched. The UI captures the matching saved request for submission, releases the lock without waiting on a service operation, and sends its full requested values and revision with the API's current state/generation preconditions.

The service captures the supplied request immutably when accepting Apply. It validates that request but neither reads the preference file nor independently discovers a later save. A later successful save creates a visible difference between the latest saved request and the accepted/applied request; it does not alter an operation already accepted by the service. The UI compares its current local revision/content with the submitted values and the authoritative operation result. If Apply fails after Save, preferences remain requested while the UI shows the actual stopped/faulted or other authoritative state. An Apply result never rewrites a different preference revision that another window saved meanwhile. Only the user-side persistence component can assert which request is currently saved.

| Read/migration case | Required result |
| --- | --- |
| File absent on first use | Load conservative editable defaults. No automatic Apply, binding change, or claim of active codec. A file is created only through the defined save flow. |
| Current valid schema | Load requested values; still validate target, capabilities, and caller when applying. |
| Supported older schema | Transform a validated copy through explicit migration steps, validate the final schema, and replace atomically with the concurrency check. Preserve the bounded prior version. Migration cannot install, bind, start a stream, or enable a previously absent feature. |
| Migration failure or unsupported old version | Preserve the source and warn. Offer supported read-only information or explicit reset; do not partially write the upgraded file. |
| Newer unsupported schema | Open in a safe unavailable/read-only settings state. Do not downgrade, drop unknown fields, or overwrite it on shutdown. Explain which application version/schema mismatch occurred. |
| Malformed, truncated, oversized, or duplicate-key file | Do not treat partial data as authoritative. Warn, preserve bounded private evidence, and offer conservative editable defaults or an explicit reset. Do not overwrite the original simply by opening the UI. |
| Valid recovery copy available | Inspect/validate it and offer explicit restoration as a requested-settings revision. It cannot restore active runtime state or installation authority. |
| Access denied, unsafe path, or full disk | Surface a storage error and preserve known state; do not elevate the UI or relax permissions to make Save work. |

Migration includes a dry in-memory validation phase and an exact supported source-to-destination version path. A supported newer format cannot be inferred from a successful JSON parse. The schema owns the unknown-field policy so that a future feature cannot silently become enabled or disappear through an older writer. Actual maximum bytes, nesting, arrays, strings, backup counts beyond the stated bounded copy, and migration work budgets are fixed and fixture-tested before storage code lands.

## Protected recovery journal

The journal is independently readable by the recovery helper with the normal UI/service/worker unavailable. It is the record of privileged intent and observed progress, not a script to replay. A valid journal cannot override live target verification or make an untrusted package safe.

| Record group | Required information |
| --- | --- |
| Format and operation identity | Journal schema version, `operationId`, monotonically ordered record sequence, operation kind, helper/build compatibility, and validated caller scope. Record wall-clock time for human context; ordering and deadlines do not rely on wall-clock continuity. |
| Exact target | `targetId` correlation plus the independently resolved OS/device/binding identity needed for that operation. Keep sensitive exact identifiers protected; presentation/export uses aliases. |
| Prior state | Observed pre-change driver/package/binding and endpoint identities, baseline reference, and verification method. Unknown prior state is explicit and may block the operation; never assume the inbox binding is identical on all systems. |
| Intended and observed owner | Explicitly chosen owner, pending operation intent, last verified owner, and observation provenance. Preserve the last completed owner choice while a new operation is pending; disagreement or uncertainty produces a drift/recovery diagnostic, not automatic rebinding. |
| Package provenance | Allowlisted staged package identity, expected integrity/signature evidence, and compatibility information. Never consume an arbitrary path or command from user settings. |
| Stage and progress | Last durable stage, stage result, pending external action, progress source, last observed progress, deadline-policy identifier, and any stalled/uncertain state. No invented completion percentage. |
| Cancellation and restart | Whether cancellation is requested, whether the current stage is a tested safe boundary, whether external completion is outstanding, reboot requirement if actually known, and available next recovery action. |
| Verification and outcome | Fresh observed binding/endpoint results and separate playback/capture checks for target and non-target devices. Record whether immediate and post-reboot checks are passed, failed, unavailable, or not run; helper process success alone is not restored audio. |

Before a privileged external change, append and durably record its intended step. After completion, append the observed result. If the pre-change journal write fails, do not start that change. If a post-change write fails, preserve uncertainty and require reconciliation against live state; do not fabricate success or immediately repeat the action. Logical append-only records protect the history from accidental rewriting; the implementation must also validate record framing/order and detect a torn final record. Checksums detect accidental damage, not malicious administrator modification.

V1 serializes privileged helper mutations machine-wide, covering both the selected target and shared package resources. A per-target lock alone does not make package removal safe. Repeated requests with an existing `operationId` return or reconcile the recorded operation; changed intent under the same identifier is rejected. Requests for a new operation do not bypass an unresolved predecessor. Lock acquisition, ownership recovery after helper death, and journal parsing must be bounded.

After a crash or reboot, the helper validates the journal version, protected location, package, target and current OS state before proposing or performing a separately authorized reconciliation step. A truncated, incompatible, missing, or ambiguous journal yields an honest recovery-required state. Do not guess the previous binding, delete broad classes of drivers, or reconstruct authoritative records from untrusted diagnostics. Compatible offline helper/recovery instructions remain part of the release package; service API incompatibility must not make recovery inaccessible.

A stalled stage retains its operation identity and pending state. A cancel request is recorded and executed only at a tested safe boundary; it does not kill an in-flight PnP operation or erase its journal. UI closure does not cancel the helper's operation. A later UI can obtain a filtered projection of the same journal progress. Measured stage deadlines and the tested restart/cancel behavior are prerequisites to the installer implementation, not guessed by this document.

## Identity, routing, and unrelated devices

Before install/restore/update, capture protected observations of the selected target's endpoint identity and relevant non-target outputs/inputs. Distinguish identity/enumeration observations from actual playback/capture results. Record deliberate user changes so that a changed baseline is not misdiagnosed as a project failure. Avoid storing application names or paths unless necessary for a specifically consented diagnostic case.

Track old/new endpoint identity associations and whether the mapping is verified, ambiguous, or unavailable. A changed identifier invalidates dependent cached handles. The UI provides app-routing/EQ/APO reattachment guidance when needed; the journal does not grant permission to rewrite third-party configuration. Default routing, selected microphone, channel/spatial processing, Windows PCM, active profile, transport owner and codec are distinct observations, each with provenance/freshness. No saved codec preference can disable another microphone or change unrelated speakers.

Stream boundaries do not generate persistent active-state checkpoints. `streamGeneration` and its clocks/partial frames reset in runtime ownership; reloading a preference must not revive old packet sequence, buffers, or channel mapping. This addresses stream-transition reports without turning the settings file into a media checkpoint.

## Privacy, limits, and lifecycle

Resolve owned roots without accepting traversal, arbitrary absolute paths, or caller-controlled environment roots across a privilege boundary. Verify the actual opened objects and their permissions, including parent directories and reparse/link behavior; string-prefix checks alone are insufficient. Temporary files, replacement files, backups, diagnostics and journal projections receive the same access policy as their originals. Do not repair a malicious path by following it with elevated rights.

The standard user cannot edit the privileged journal. The helper exposes only the minimum authorized projection and treats every record as untrusted parser input despite its protected location. Removing per-user preferences does not remove recovery state. A user-facing Reset preferences action cannot change binding or uninstall packages. Completed journal history is compacted only under a reviewed policy that preserves the current recovery baseline; unresolved operations and the only usable recovery record are never silently rotated away.

Normal diagnostics use structured enums, stages, counts and bounded errors, with device names, addresses, OS user paths and exact target identifiers excluded or redacted. Export aliases are created afresh for each bundle and applied consistently across its files; no stable hashed address is used as a public identity. A bundle includes a schema/version manifest and clear unavailable/not-run fields, allowing offline investigation without pretending that missing data are zero or success. Raw journal files, dumps, audio and captures are excluded from the default bundle. The unelevated UI previews the export and writes it only after the user's explicit request.

M0-WP04 must specify diagnostic byte caps, retention periods, export limits, opt-in duration, cleanup behavior and full-disk policy before logging implementation. M6-WP02 fixes settings parser/write limits; M3-WP01 fixes journal/stage limits before privileged operations. High-rate events may be dropped with a counter rather than blocking audio; recovery-critical records cannot be silently dropped to continue a privileged change. These ownership assignments are implementation gates, not permission to ship unbounded storage.

## Acceptance handoff

All checks below are planned and unexecuted. Document review does not prove filesystem durability, ACL correctness, recovery, or audio behavior.

| Requirement / references | Storage responsibility and planned evidence |
| --- | --- |
| RQ-004, RQ-015; TC-004, TC-015 | Atomic replacement, same-revision external edit and multi-window conflict, full disk, migration interruption, malformed/newer schemas, orphaned targets, and Save/Apply divergence. |
| RQ-008; SEC-001, SEC-002, SEC-006, SEC-007; TC-008 | Cross-user access denial; hostile paths/reparse points; permission inheritance; parser limits; no secrets/audio in default stores or redacted exports. |
| RQ-007, RQ-011; REF-003/005; TC-007, TC-011, TC-016 | Protected prior state, independent helper, torn/missing/incompatible journal, interrupted external action, repeated request reconciliation, and recovery with service unavailable on the authorized lab. |
| RQ-019; REF-011/014; TC-019 | Preferences never revive stale stream clocks/frames/generations; repeated track/format transitions remain runtime tests with separate left/right evidence. |
| RQ-020; REF-004/006/023; TC-020 | Protected target/non-target baseline, recorded intentional changes, playback/capture comparison immediately and after reboot. Preserve ambiguous reference-product attribution. |
| RQ-021; REF-018/019; TC-021, TC-014 | Separate intended/pending/observed owner across restart/update; no auto-rebind, entitlement, or clock-expiry behavior. |
| RQ-022; REF-009; TC-022 | Durable stage/progress/cancel records, helper crash/full-disk cases, safe cancellation boundaries, and reopening the same operation after UI closure. |
| RQ-023; REF-013; TC-023 | Preserve explicit manual quality, keep adaptation absent/disabled in v1, and prohibit migration from silently enabling it. |
| RQ-024; REF-020/022; TC-024 | Offline readable recovery guidance and sanitized structured reporting; no automatic export/upload or dependence on maintainer availability. |
| RQ-025; REF-021/014; TC-025 | Endpoint identity changes/ambiguity invalidate caches and produce reattachment guidance without third-party writes. |
| RQ-026; REF-007/008/015/016/017; TC-026 | Persistent requests cannot establish active PCM/profile/codec/input state; unavailable/stale evidence remains explicit. |

Before implementation, review the exact schema, parser limits, target-key resolver, writer locking/replacement API, service/helper projections, and journal reconciliation with the API and security owners. All kernel buffer/IOCTL layouts, codec parameter validity, and actual Windows binding/recovery mechanics remain owned by their later experiments.
