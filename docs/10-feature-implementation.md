# Feature implementation slices

Status: proposed M0-WP03 implementation guidance, 2026-10-30 (Asia/Calcutta). No feature, runtime test, toolchain, driver deployment or hardware result is delivered by this document. The [roadmap](08-project-implementation.md) owns package order, status, numeric acceptance targets and exit conditions; this document explains implementation boundaries and failure-oriented handoffs. Read the [architecture](03-architecture.md), [API contracts](05-api-and-contracts.md), [storage](04-database-and-storage.md), [UI design](02-ui-ux-design.md) and [security](11-security.md) before implementing their components.

## Selecting a slice

Complete M0, including batch 4 test/logging procedures and consistency review, before entering M1. Select one named roadmap package per session, with an explicit input/output contract and acceptance record. If its changes exceed a reviewable session, record named subpackages and their handoffs before coding; completing a subpackage does not close the parent exit gate. Roles below are responsibilities, not additional runtime processes or an instruction to launch development agents.

Use [setup and initialization](09-setup-and-initialization.md) for the toolchain procedure, [development roles](06-development-agents.md) for ownership and review, [session workflows](07-agent-workflows.md) for execution records, and [integrations](12-integrations.md) for external dependencies. Existing `sources/` and synced project files stay read-only. Proposed layouts and semantic payloads are not executable schemas or a settled kernel ABI.

The dependency chain is SBC transport proof, Windows system-audio proof, then LDAC. A mock control response, encoded file or active UI meter cannot bypass that order. Only safe user-mode/offline work and read-only inspection are permitted on the primary PC; all experimental installation, binding, driver verification and recovery experiments require the separate M2 lab and scoped authorization.

For every accepted slice, record the exact commit/environment, owned files/interfaces, fixed numeric limits where applicable, positive and failure cases, actual results, unexecuted tests, and unresolved risks. Review changes across a privilege or kernel boundary independently. A failed prerequisite stops dependent work and creates a concrete follow-up; it does not justify widening privilege or silently reducing acceptance.

## M1: establish safe inputs before device control

Entry: M0 complete. These slices have no driver-installation side effects. No dependency version or codec support is inferred from a source link or an installed tool name.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M1-WP01; build/dependency owner | Inventory existing tools read-only; select compatible OS/tool targets and normative protocol revisions; review SBC candidates and LDAC dependencies; record immutable provenance, license proposal and reproducible build layout. | RQ-014/017, TC-014/017, SEC-008/009: ledger and identified validation environment. Missing tools become explicit setup needs, not assumed successful builds or automatic installation. Handoff the chosen revisions and unresolved distribution questions to M1-WP02/M5-WP01. |
| M1-WP02; protocol/service owner | From WP01's reviewed inputs, create the bounded user-mode capability/protocol core and executable control schemas; add permitted fixtures before connecting OS or driver operations. Define parser, enum, size, timeout, concurrency, replay and event-cursor behavior. | RQ-002/008, TC-002/008, SEC-001 through SEC-005: reproducible offline fixtures exercise malformed data, wrong identity/state, duplicate requests, expired results and stale generations. Passing fixtures is not encoder, Bluetooth or hardware evidence. Failed bounds block exposure through a privileged boundary. |
| M1-WP03; inventory owner | Consume the bounded core/report schema; implement read-only OS/radio/endpoint observations, target-key resolution and redacted reports. Separate unknown, unsupported and detected values; test ambiguous or changed identities. | RQ-001/026, TC-001/026, SEC-007: actual read-only inventory plus synthetic unsupported cases and before/after binding comparison. Do not probe remote codec capability through an assumed privileged path or substitute advertised headset specifications. Unavailable observations remain unknown. |

M1-WP02 owns executable fixture contracts, including synthetic authorization state; actual pipe authentication, driver ACLs and deployment evidence remain in their integration packages. Keep test seams explicit so an offline fake cannot accidentally open a production device.

## M2: prove the recovery precondition

Entry: a separate recoverable Windows 11 x64 target is available. M2 does not turn the primary PC into a fallback test machine.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M2-WP01; lab/recovery owner | Record exact OS/radio/driver/receiver baseline and working inbox playback; prepare offline backup/recovery and private recovery material; rehearse recovery with local access independent of Bluetooth/service availability. | RQ-007/016, TC-007/016, SEC-006: actual recovery rehearsal and baseline artifacts, with secrets excluded. An untested restore point or inaccessible recovery key leaves the gate open. |
| M2-WP02; lab owner with independent security review | Use WP01 evidence to bound the target, permitted deployment/debug operations, stop conditions and cleanup procedure; record scoped authorization and interface review. | RQ-008/016, TC-008/016, SEC-001/002/003/006: reviewed target-specific procedure. Missing hardware, recovery or authorization blocks deployment; continue only independent safe work. |

## M3: prove target ownership, then controlled-source SBC

Entry: M1/M2 exit evidence and relevant security review. Each package consumes the previous package's measured result; the independent helper is part of transport proof, not deferred until a polished UI exists.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M3-WP01; driver/helper owner | Resolve the exact target; capture target/non-target baseline; establish machine-wide install exclusion and quiescence; implement protected journal/stage projection and minimal binding/channel-ownership experiment; restore the prior path. | RQ-007/011/020/021/022, TC-007/011/020/021/022, SEC-001/002/003/005/006: Q-001 ADR, scoped ownership and actual restored playback. Journal-before-side-effect failure prevents mutation; uncertain completion is reconciled, not replayed. No global stack patch, blind PnP kill or unattended rebind. |
| M3-WP02; protocol/transport owner | On proven ownership, implement bounded AVDTP discovery/configuration transactions; refresh remote capabilities per connection; expose authoritative negotiation, profile/PCM/owner distinctions and available counters to the harness. | RQ-002/006/026, TC-002/006/026, SEC-002/003/004/005: independent capability/selected-SBC evidence and Q-005 observability record. Reject malformed/timeout/stale results; unknown telemetry is not zero, and an unsupported request cannot silently choose another codec. |
| M3-WP03; codec/transport owner | Use the reviewed SBC encoder with a permitted controlled PCM fixture, current negotiated limits and bounded packetization; exercise stop/reconnect/restore on the lab. No ordinary Windows endpoint yet. | RQ-003/013, TC-003/013, SEC-003/004/005: roadmap's controlled playback duration, channel/format checks and bounded resource observations. Encoder output exceeding MTU/format limits fails before submission; queue pressure cannot allocate indefinitely or discard arbitrary encoded bytes. |

Keep the successful transport revision and environment as the comparison baseline for M4. Failure to establish narrow reversible binding resolves Q-001 by revising the architecture, not by expanding device matching or permission scope.

## M4: connect ordinary Windows audio without stale stream state

Entry: M3 SBC transport proof. Endpoint mechanics remain experimental until M4-WP01 records measured ownership, timing and resource limits.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M4-WP01; endpoint/audio owner | Review sample/dependency provenance; implement the minimum endpoint and bounded PCM exchange; document formats, channel mapping, buffer lifetime, cancellation and measured numeric budgets before finalizing the ABI. | RQ-003/008/013/025, TC-003/008/013/025, SEC-003/005: Q-002 ADR and measured exchange/endpoint proof. Unfinished callbacks retain their resources; cleanup timeout isolates state rather than freeing reachable memory. Endpoint identity preservation requires evidence. |
| M4-WP02; audio/worker owner | Connect ordinary-app PCM through the existing SBC path; implement start/stop, worker death and track/app transitions with fresh stream generations, cleared partial frames and protocol-correct clock reset. Preserve a valid Bluetooth connection across unchanged-format stream boundaries. | RQ-003/009/019, TC-003/009/019, SEC-004/005: ordinary-app duration, repeated boundaries/format alternations and left/right checks from the roadmap/PRD, plus crash cleanup and inbox restore. Stale completions may finish cleanup but cannot alter a new stream; no manual reconnect requirement is hidden by the harness. |
| M4-WP03; coexistence owner | Exercise removal, reconnect, sleep/resume, Windows HFP/input ownership and AVRCP; compare independent microphones/non-target outputs; record endpoint/routing changes and known limitations. | RQ-009/010/020/025/026, TC-009/010/020/025/026, SEC-005/006: Q-003 ADR, exact cycle counts and coexistence matrix. No silent microphone disabling or third-party routing rewrite. An unsafe profile interaction blocks dependent alpha claims until resolved or an explicit scope change is approved. |

Connection and stream generations solve different lifetimes: reconnect invalidates channels/capabilities; a track boundary can invalidate only stream frames/clocks. Suspend invalidates pre-suspend assumptions. Validate both fences independently rather than using one counter for every event.

## M5: add LDAC without weakening the SBC baseline

Entry: ordinary-app SBC and relevant M4 lifecycle evidence. Encoder availability, distribution eligibility and received-codec proof are separate gates.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M5-WP01; codec/dependency owner | Pin and review the LDAC source/build inputs and notices; implement its user-mode adapter against bounded contracts; record the distribution decision independently. | RQ-005/017, TC-005/017, SEC-004/008: source/provenance review and encoder bounds fixtures. A successful build does not clear distribution or activate an unreviewed adapter. No arbitrary DLL path from preferences. |
| M5-WP02; codec/protocol owner | Add variant-specific capabilities, negotiation, packetization and only validated quality/format choices using the existing transport/endpoint ownership model. | RQ-005/006/026, TC-005/006/026, SEC-003/004/005: independent negotiated LDAC evidence and roadmap playback target for every exposed mode. Missing independent evidence leaves the gate open; neither Windows PCM rate nor a dashboard label proves LDAC reception. |
| M5-WP03; service/audio owner | Implement explicit captured-request stop/configure/restart and manual quality policy, with selectable SBC and honest post-failure state. Exercise service-level requests through the harness; production preference persistence arrives in M6-WP02. | RQ-004/023, TC-004/023, SEC-002/005: failed/interrupted apply, explicit retry and SBC regression. Invalid preflight leaves valid playback alone; failure after stop reports stopped/faulted. No automatic cross-codec fallback or hidden adaptive quality. |

M5's harness validates control semantics; it does not claim the M6 desktop Save-and-apply workflow exists. Any later adaptive policy requires the separately approved M7-WP02A measurements and tests.

## M6: make the proven path usable and recoverable

Entry: M5 evidence for the LDAC alpha and available independent helper. Implement [UI flows](02-ui-ux-design.md) against authoritative observations; never make a fixture-backed screen a compatibility claim.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M6-WP01; desktop/service owner | Implement authenticated service connection, exclusive session ownership, accessible inventory/status and requested/active presentation. Keep independent recovery/help reachable when service handshake fails. | RQ-002/008/018/026, TC-002/008/018/026, SEC-001/002: keyboard/screen-reader, fake-server/unauthorized-session, stale-event and service-down cases. UI closure leaves healthy audio running; logoff/release cleans before reassignment. |
| M6-WP02; desktop/storage owner | Implement per-user atomic preference persistence/migrations, then explicit Save/Apply orchestration using the sequence below. Reconcile corrupt/newer schemas and ambiguous target identity without changing binding. | RQ-004/015/021, TC-004/015/021, SEC-002/007: interrupted writes, concurrent windows, full disk, version conflicts and restart resynchronization. Save failure prevents dispatch; a preference cannot restore a stream lease or authorize rebind. |
| M6-WP03; diagnostics/helper/UX owners | Add bounded local diagnostics, inspectable redacted exports, journal-backed progress and offline recovery entry points; explain routing reattachment and support status. | RQ-006/007/014/020/021/022/024/025, TC-006/007/014/020/021/022/024/025, SEC-006/007: service-down recovery, safe cancel/stall, no-expiry/offline checks and seeded export redaction. Pending verification stays pending; no automatic upload or third-party configuration rewrite. |

The M6-WP02 acceptance seam must exercise this complete sequence:

1. UI saves validated preferences under the per-user lock with revision/content conflict checks; only successful replacement advances `settingsRevision`.
2. UI checks and captures the complete saved request under that lock, then releases it before contacting the service. Unsaved or failed-save drafts cannot be dispatched by the v1 product flow.
3. Service validates authenticated ownership, `serviceEpoch`, `snapshotSequence`, target, connection/stream generations and current capabilities. `settingsRevision` only correlates input; the service cannot read or detect a later local Save.
4. On acceptance, service captures immutable values and advances its decision-state sequence. A later Save changes displayed saved preferences, not the accepted operation. UI displays saved/applied differences; it does not invent a service stale-state error from an unseen file edit.
5. Terminal result and a fresh authoritative snapshot establish negotiated/stream state. Reopening UI observes the existing operation; late events, a replaced service epoch and expired deduplication results never cause automatic redispatch with refreshed preconditions.
6. Exercise failed post-stop negotiation, conflicting windows, lost responses, event gaps and old-generation callbacks. A persisted request, accepted command and active configuration must remain distinguishable in every result.

## M7 through M9: broaden evidence before release or another codec

Entry to M7 is the lab-tested alpha, not broad platform support. Each row remains a separate package; none combines release qualification with feature implementation.

| Package and owner | Change sequence and bounded handoff | Exit proof and failure path |
| --- | --- | --- |
| M7-WP01; compatibility owner | Expand exact hardware/OS matrix; repeat profile, routing/endpoint, channel and non-target checks after relevant updates. | RQ-009/010/019/020/025, TC-009/010/019/020/025, SEC-005/006: tested matrix and explicit exclusions. Unavailable hardware keeps broad-compatibility acceptance open. |
| M7-WP02; performance owner | Compare matched inbox/project baselines, approve numeric thresholds and measurement methods, then run specified SBC/LDAC stress/soak. | RQ-013/019/026, TC-013/019/026, SEC-005: measured resources, discontinuities and metric limitations. Do not attribute dropouts to RF without evidence or claim unmeasured latency/quality gains. |
| Optional M7-WP02A; audio owner | Only if separately justified, define within-codec adaptive floor/signals/hysteresis and explicit opt-in; exercise downgrade, recovery, stale measurements and disabling back to manual preference. | RQ-023, TC-023, SEC-005: measured policy results and codec regression. Otherwise manual quality remains the only delivered policy and adaptive algorithm tests stay deferred. |
| M7-WP03; independent security/recovery reviewers | Review privilege/kernel boundaries; fuzz parsers/IPC; inject worker/helper/process interruptions; verify safe cancellation, ownership drift and restored target/non-target state on the lab. | RQ-008/011/012/020/021/022, TC-008/011/012/020/021/022, SEC-001 through SEC-009: required cycles, documented failures and closure of critical/high findings. Unresolved serious failures block affected release qualification. |
| M8-WP01; release owner | Revalidate signing/certification eligibility, applicable tests, supported OS builds and controlled key custody against current official requirements. | RQ-012/017, TC-012/017, SEC-008: exact qualification route and evidence needs. Test signing does not satisfy public-release eligibility. |
| M8-WP02; release/helper owner | From WP01 and beta evidence, assemble the compatible signed set; verify clean-target install, manual update/rollback, offline standard-user playback and restoration. | RQ-007/011/012/014, TC-007/011/012/014, SEC-006/008: required protections remain enabled and clean-target results are recorded. Incompatible versions or unsigned/unqualified packages cannot be relabeled release-ready. |
| M8-WP03; maintainer/release owner | Finalize approved license/notices, source/build/recovery publication, tested matrix, known defects, private disclosure contact and support policy within authorized publishing scope. | RQ-010/014/017/024, TC-010/014/017/024, SEC-007/008/009: evidence-linked release checklist and inspectable sanitized report route. No invented support staffing, codec rights or feature parity. |
| M9 `<codec>-WP01`; codec/dependency owner | Select one codec variant only after its track is authorized; establish exact implementation/interface, provenance, distribution eligibility and available receiver. | RQ-002/017, TC-002/017, SEC-004/008/009: track-specific gate record. Missing availability leaves it research-only; success for another variant is insufficient. |
| M9 `<codec>-WP02`; codec/protocol owner | After that track's WP01, implement only the reviewed adapter, variant-specific capabilities and negotiation/packetization through existing bounded contracts. | RQ-002/004/013, TC-002/004/013, SEC-003/004/005: malformed/unsupported/failed-negotiation tests and scoped technical proof. Do not expose controls for unverified variants. |
| M9 `<codec>-WP03`; compatibility/security owners | Independently verify receiver negotiation/playback, applicable latency/artifact behavior, boundary/security failures and existing SBC/LDAC regression. | RQ-009/010/013, TC-009/010/013, SEC-004/005/008: variant-specific evidence and support disposition. Do not infer low latency from an encoder label or one successful mode. |

## Reference-report handoff checklist

These are unverified reference reports, not confirmed project bugs. Every recipient carries the named scenario into its package record; exact procedures are expanded in M0-WP04. Ambiguous attribution, subjective listening and missing versions remain recorded in the [risk register](defect-risk-register.md).

| Reference reports | First implementation owner and subsequent handoff | Requirement / planned test and required distinction |
| --- | --- | --- |
| REF-001/002/003 | M3-WP01 helper; M4-WP02/03 audio/coexistence; M6-WP03 diagnostics | RQ-003/006/007/009/011; TC-003/006/007/009/011: link versus endpoint/stream readiness, resume and real restored playback, including required reboot checks. |
| REF-004/006/023 | M3-WP01 baseline; M4-WP03 coexistence; M7-WP03 review | RQ-020; TC-020: unrelated speakers/inputs and separate-mic mute/capture immediately and after reboot; preserve REF-023's product-attribution uncertainty. |
| REF-005 | M2-WP01 recovery; M3-WP01 helper; M7-WP03 interruption review | RQ-007/011/016; TC-007/011/016: offline recovery independent of OS/service startup; no primary-PC reproduction or assumed driver root cause. |
| REF-007/008 | M3-WP02 capabilities; M4-WP03 profiles; M5-WP02 codec; M6-WP01 UX | RQ-002/005/009/026; TC-002/005/009/026: fresh support and selected input/profile explain unavailable LDAC without disabling microphones. |
| REF-009 | M3-WP01 helper; M6-WP03 UX; M7-WP03 interruption review | RQ-022; TC-022: durable operation stages, measured watchdog and safe cancellation; stale progress is not permission to kill PnP. |
| REF-010/011/012 | M4-WP02 audio; M7-WP02 performance; future LL track separately | RQ-004/013/019; TC-004/013/019: matched baseline, repeated boundaries/formats and artifacts; an unspecified low-latency report does not establish aptX LL causation. |
| REF-013 | M5-WP03 service; optional M7-WP02A audio | RQ-023; TC-023: manual policy first; adaptation needs measured recovery/disable behavior rather than a hidden quality reduction. |
| REF-014/021 | M4-WP01/03 endpoint/coexistence; M7-WP01 compatibility | RQ-010/019/025; TC-010/019/025: channel fixtures and endpoint mapping, OS-update evidence and explicit routing/EQ/APO reattachment. |
| REF-015/016/017 | M3-WP02 observations; M5-WP02 codec; M6-WP01 UX | RQ-001/005/013/026; TC-001/005/013/026: distinguish PCM/profile/owner/codec, unavailable timing and supported rates; no guaranteed audible improvement or decoder-list inference. |
| REF-018/019 | M3-WP01 ownership; M6-WP02/03 persistence/recovery; M7-WP03 review | RQ-014/021; TC-014/021: explicit owner survives restart/reconnect/update; hardware changes trigger rediscovery, never entitlement failure or autonomous takeover. |
| REF-020/022 | M6-WP03 offline UX/diagnostics; M8-WP03 maintainer | RQ-008/017/018/024; TC-008/017/018/024: inspectable provenance, accessible recovery and honest support/disclosure routes independent of purchase or maintainer response. |

Each package closes only against its recorded exit evidence. Documentation coverage, fixture success, local observation, independent hardware verification and release qualification remain separate result types; no row in this plan is a passed test.
