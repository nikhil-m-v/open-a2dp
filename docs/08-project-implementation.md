# Project implementation and session roadmap

Baseline: 2026-10-21. This is the authoritative task/checkpoint record. Scope is defined in the [PRD](01-project-requirements.md); technical gates follow the [system design](15-system-design.md) and [security controls](11-security.md).

## Execution rules

One implementation session selects **one bounded work package**. A milestone is not one prompt. If a package cannot reasonably be finished/reviewed in a session, split it into named subpackages before implementation; do not claim partial delivery completes its exit condition.

For each package, record objective, prerequisite evidence, exclusions, intended changes, acceptance checks, actual results, risks, recovery notes, and the next task. Tests that were not run are labeled unexecuted. A mock or fixture result is never hardware evidence. Keep existing synced files and `sources/` unchanged.

Only the primary PC is available. M0 and safe M1 work may proceed. Driver installation/testing waits for M2. Research uncertainty is resolved by a specific experiment and ADR rather than by blindly implementing dependent features. A failed gate stops only dependent tasks; safe documentation/offline work can continue.

## Current status

| Milestone | State | Exit condition |
| --- | --- | --- |
| M0 Documentation and reference behavior | In progress: batch 1 authored | Four batches reviewed, pending user defects explicitly tracked, reference claims distinguished from observations, requirements/tests linked. |
| M1 Development foundation | Not started | Reproducible safe user-mode build, pinned inventory, specification/fixture baseline, read-only device inventory. |
| M2 Recoverable hardware lab | Waiting for separate target | Exact hardware baseline and demonstrated offline recovery with deployment authorization. |
| M3 Bluetooth transport proof | Blocked by M1/M2 evidence | Controlled-source SBC playback, target/channel ownership, disconnect cleanup, and inbox restoration demonstrated. |
| M4 Windows system-audio proof | Depends on M3 | Ordinary-app SBC playback through the project endpoint; lifecycle, HFP/AVRCP behavior, and restore validated. |
| M5 LDAC vertical slice | Depends on M4 and distribution review | Independently verified negotiated LDAC playback and supported quality settings. |
| M6 Usable desktop alpha | Depends on M5 for LDAC alpha | Accessible UI, persistent preferences, diagnostics, explicit apply, and independent recovery discoverable. |
| M7 Stability/security beta | Depends on M6 | Measured thresholds, broader matrix, stress/security review, update/recovery tests pass. |
| M8 Public release | Depends on M7 and signing eligibility | Signed clean-target installation under required protections; licensing/build/recovery evidence and release notes published. |
| M9 Additional codecs | Separate gated tracks | Each codec independently satisfies availability, distribution, implementation, receiver, security, and regression gates. |

Blocked dependencies here describe milestone prerequisites, not a request to mark the whole project blocked or stop safe work.

## M0 — Documentation, four sessions

| Package | Deliverable / bounded work | Acceptance and handoff |
| --- | --- | --- |
| M0-WP01 | Batch 1: PRD, roadmap, system design, security, index, evidence and defect/risk registers. | Links/IDs/status verified; no false runtime claims or synced-file changes. Next: M0-WP02. |
| M0-WP02 | Batch 2: architecture, UI/UX, database/storage, and API/contracts. | Define authorized operations, version/error policy, target and session identity, requested/active model, preference migration, accessible recovery flow. Identify experimental data-path decisions without inventing a settled kernel ABI. Update index/checkpoint. |
| M0-WP03 | Batch 3: setup, feature implementation, integrations, development agents and workflows. | Describe safe build commands and version selection/pinning procedure, component task ownership, one-package session template, dependency reviews, and optional REA evaluation. No tool installation, forks, driver deployment, or agent launch merely to author these files. |
| M0-WP04 | Batch 4: QA/testing, debugging/logging, and consistency review. | Every RQ and SEC has planned coverage; define evidence templates, log limits/retention, hardware matrix, performance-baseline protocol, failure injection, and release gates. Record missing defect input explicitly. |

When the user's defects arrive, intake them as the first step of the active documentation package and update affected documents. Do not wait indefinitely for them or call them verified. If no local reference-app observation exists by M0 close, state that plainly and carry behavior study as an optional evidence task rather than claiming it happened.

## M1 — Safe development foundation

Entry: documentation baseline M0 complete; no driver deployment needed.

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M1-WP01 | Inventory available build tools read-only; choose/pin compatible .NET/WPF, C++/WDK and test tools; establish repository/build layout and exact target OS builds; review SBC candidates and libldac dependencies; record project-license proposal and normative protocol revisions. | Dependency/provenance ledger with immutable references; tool installation needs are explicit; clean user-mode build instructions have an identified validation environment. No unsupported promise of WDK availability. |
| M1-WP02 | Implement only the user-mode protocol/capability core and permitted fixture corpus, with bounds and planned IPC/driver-contract validation. | Meaningful malformed/edge/state fixtures and a reproducible offline build/test run; no hardware or encoder support claims from fixtures. |
| M1-WP03 | Implement read-only OS/radio/audio inventory and an environment-report format. | TC-001 on actual available hardware, synthetic unsupported cases, redaction, and before/after device binding confirmation. Do not substitute advertised headset specifications for queried capabilities. |

M1 addresses Q-004/Q-006 in part. Exact dependencies become pinned here; documentation links to moving branches are not dependency locks. Optional research tooling is not required to pass this milestone.

## M2 — Recoverable lab gate

Entry: a separate test target can be arranged. The primary PC is not the target.

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M2-WP01 | Inventory dedicated Windows 11 x64 target, actual Bluetooth radio/driver, SBC and LDAC receiver support/firmware, debugger route, security state, and working inbox playback. Prepare offline backup/recovery media and readable target identification. | Baseline record and recovery rehearsal restoring working inbox playback; physical access and recovery actions are available even if Bluetooth/audio/service fails. |
| M2-WP02 | Define and authorize a target-specific deployment/debug procedure, permitted lab security configuration, cleanup, and stop conditions. | Recorded target/scoped authorization, reviewed recovery procedure and package contracts. No deployment onto primary PC; VM access only if physical-radio forwarding and recovery are demonstrated. |

If hardware is unavailable, remain in safe M1/offline work. A build-only proof is still useful but does not satisfy M2/M3.

## M3 — Bluetooth transport proof

Entry: M1/M2 gates and SEC-001/002/003/006 design review. Lab only.

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M3-WP01 | Minimal narrow profile-driver binding and documented Bluetooth L2CAP channel ownership experiment, plus independent install/restore helper. No full audio endpoint or additional codec. | Q-001 ADR: exact target binding, no global stack patch, no competing ownership, cleanup/removal, and successful inbox playback after restoration (TC-007). If impossible, stop and revise architecture. |
| M3-WP02 | Implement AVDTP discovery/configuration and bounded transactions in the transport proof; expose authoritative negotiation/status to the test harness. | Remote capabilities and selected SBC parameters corroborated independently; malformed/timeout/generation tests pass; Q-005 measurement availability recorded. |
| M3-WP03 | Stream a permitted controlled PCM source through the reviewed SBC encoder and packetizer to the headphones; no Windows endpoint yet. | At least 10 minutes playback plus channel/format verification, bounded queue and resource observations, stop/reconnect/restore evidence; distinguishes transport from system-audio proof. |

No production UI is required to prove M3. Preserve the exact successful revision and environment before adding an endpoint.

## M4 — Windows system-audio proof

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M4-WP01 | Evaluate endpoint integration using reviewed Windows samples; implement bounded PCM exchange with the user-mode worker. Set numeric capacities/timeouts from measured experiment before finalizing its ABI. | Q-002 ADR includes endpoint format/timing, buffer layout and ownership, IRQL/cancellation behavior, flow control, security review, and measured proof. A moving UI meter alone fails. |
| M4-WP02 | Connect ordinary Windows application rendering end to end over SBC; implement explicit start/stop and crash handling. | TC-003: 30-minute playback; channel/sample checks; UI-independent playback; worker failure stops/cleans correctly; successful return to inbox. |
| M4-WP03 | Validate removal, sleep/resume, HFP/microphone transitions and AVRCP on the lab configuration. | TC-009/010 initial coverage including 20 reconnect and 10 power cycles; Q-003 ADR states safe coexistence or an explicitly approved limitation before alpha. |

## M5 — LDAC vertical slice

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M5-WP01 | Pin and build the reviewed LDAC encoder in user mode; complete notices/source/distribution review and codec adapter contract. | Encoder fixture/bounds tests, no unreviewed DLL loading, distribution decision recorded independently of technical success. |
| M5-WP02 | Add codec-specific capabilities, AVDTP negotiation, packetization and supported quality controls. | Independent negotiated LDAC evidence plus 30-minute headset playback (TC-005); validate every exposed mode/format and reject unsupported combinations. |
| M5-WP03 | Add explicit requested/active apply workflow with stop/reconfigure/restart and selectable SBC. | TC-004 failures/reconnect cases; no silent cross-codec fallback; SBC path still passes baseline regression. |

Do not infer aptX/LHDC support from success here. Adaptive quality is deferred until its measurements and recovery policy are validated.

## M6 — Desktop alpha

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M6-WP01 | WPF device/capability/status UI and service connection; session ownership; requested versus active settings; explicit unavailable states. | TC-002/018 including keyboard/screen reader, service-down view, stale events, and unauthorized session controls. |
| M6-WP02 | Per-user/device settings with atomic persistence/migrations and explicit apply actions. | TC-004/015 including interrupted write, corrupt/newer settings, permissions, and restart resynchronization. |
| M6-WP03 | Bounded diagnostics, redacted local support export, and discoverable independent recovery/install flow. | TC-006/007/014; export privacy checks; offline use; recovery tool works with the normal service unavailable. No automatic upload/update. |

Alpha is explicitly lab-tested software. Its label does not imply signed public deployment or broad headset compatibility.

## M7 — Stability and security beta

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M7-WP01 | Broaden the matrix to at least two radio/driver families and two headphone models where available; include interference, contention, protected-media and profile-transition results. | TC-009/010 with exact limits. If hardware is unavailable, publish only the narrower tested scope and keep broad-compatibility acceptance open. |
| M7-WP02 | Compare inbox/project baselines; lock approved CPU/memory/queue/latency/discontinuity thresholds and measurement methods; run stress/soak. | TC-013: at least a two-hour SBC and two-hour LDAC run on the primary lab combination; record observable dropouts/errors, resource trends, counter limitations, and exact thresholds before declaring beta. |
| M7-WP03 | Independent security review, parser/IPC fuzzing, focused lab driver verification, crash/cancellation tests, and install/update/restore interruption tests. | SEC coverage and TC-008/011/012; five install/restore cycles; critical/high findings closed; recovery works under the failure cases. |

If adaptive quality is justified, add a separate M7-WP02A package: use only measured signals; specify hysteresis/limits, test sustained poor conditions and recovery, and rerun codec regression. Otherwise retain manual quality settings for v1.

## M8 — Public release

| Package | Deliverable | Exit evidence |
| --- | --- | --- |
| M8-WP01 | Revalidate current signing/WHCP eligibility, applicable HLK requirements, supported OS builds, costs, and controlled signing process. | Organization/certificate/submission route and required test evidence available; security review covers key custody. No assumption that test signing satisfies release. |
| M8-WP02 | Build signed installer/helper/driver/service set; verify clean-target install, manual update, intentional compatible rollback, standard-user offline playback, and restoration. | TC-007/011/012/014 with Secure Boot and Memory Integrity enabled on supported targets. No installer-driven disabling of security features. |
| M8-WP03 | Complete source/build publication, approved project license, dependency notices, matrix, known defects, recovery guide, private security contact, and versioned release notes. | Every applicable RQ/SEC has a disposition and evidence. Manual publication/distribution occurs only within the user's explicitly authorized publishing scope. |

Open-source intent does not resolve dependency rights or funding. If release eligibility is unavailable, publish an accurately labeled source/build status within authorized scope and keep the public-binary milestone incomplete.

## M9 — Additional codec tracks

Use a separate `<codec>-WP01` feasibility/distribution package, `<codec>-WP02` implementation package, and `<codec>-WP03` receiver/security/regression package for AAC, aptX Classic, HD, LL, Adaptive/Lossless, and each relevant LHDC variant. The order after LDAC is not committed; select one based on evidence and user priority when entering M9.

Every track requires a pinned usable implementation, compatible terms, explicit advertised/active variant identity, actual receiving hardware, independently verified negotiation, bounds/failure tests, and SBC/LDAC regression. A failed availability gate leaves that codec research-only; do not advertise unsupported controls as delivered features.

## Package record template

```text
Package / date / responsible role:
Objective and exclusions:
Prerequisites and exact environment:
Authorized target/actions (if applicable):
Expected files/interfaces and bounded change:
Acceptance scenarios and required evidence:
Actual changes and observed results:
Validation run / failures / unexecuted checks:
Risks, decisions, recovery notes, and artifacts:
Status (complete / partial / waiting) and reason:
Next bounded package:
```

## Session checkpoint — M0-WP01

- Objective: implement documentation batch 1 and supporting navigation/evidence/risk records.
- Prerequisites: user-approved plan; empty local reference directory; root instructions read.
- Exclusions: remaining documentation batches, runtime code, installation, tool setup, REA fork/use, device probing, hardware tests, publishing, and security/boot changes.
- Delivered: seven Markdown files, stable RQ/TC/SEC/risk/source/decision/question IDs, complete future document inventory, explicit unknowns and staged work packages.
- Evidence: primary documentation referenced in the source register; reference vendor pages marked as offline copies; no local runtime findings.
- Validation: passed local Markdown link resolution, balanced code-fence checks, and unique-definition/reference checks for 18 requirements, 18 planned test scenarios, 9 security controls, 15 risks, 12 sources, 8 decisions, and 7 research questions. The root instruction file's SHA-256 is unchanged and `sources/` remains empty. No runtime tests are applicable to this documentation-only change.
- Status: **M0-WP01 complete**. M0 remains in progress; batches 2 through 4 are pending.
- Risks: user's defect list is pending; target hardware and all driver/codec feasibility remain unverified.
- Recovery: no system settings changed; documentation is additive under `docs/`.
- Next: **M0-WP02**, documentation batch 2. Intake user defects first if available; otherwise proceed with their absence recorded.
