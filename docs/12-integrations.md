# Integrations and dependency reviews

Status: proposed M0-WP03 design, 2026-10-30 (Asia/Calcutta). No dependency, protocol revision, toolchain, codec, or optional research tool is selected, pinned, installed, or integrated by this package. No hardware interoperability has been demonstrated. This document turns the existing [source register](source-evidence-register.md) into review and implementation handoffs; it adds no new external factual finding.

Follow the [architecture](03-architecture.md), [API/contracts](05-api-and-contracts.md), [security](11-security.md), [feature plan](10-feature-implementation.md), and [roadmap](08-project-implementation.md). A successful dependency build, advertised receiver codec, or UI status does not prove working A2DP playback.

## Integration boundaries

| Integration | Intended responsibility | Evidence and owning package |
| --- | --- | --- |
| Windows device/audio inventory | Read-only OS/radio/endpoint observations and explicit unknown/unsupported states; resolve targets without relying on display names. | M1-WP03: exact observed environment, redacted report, synthetic unsupported cases, and unchanged binding before/after inventory. |
| Windows Bluetooth profile transport | Per-target binding and bounded control/media operations through reviewed documented interfaces. Windows retains pairing and out-of-scope profile ownership. | SRC-002 is a research lead; M3-WP01 must resolve Q-001 with channel ownership, cleanup and independent restoration evidence. |
| Windows render endpoint | Ordinary application PCM, format/channel meaning, timing and lifecycle through a reviewed endpoint/PCM mechanism. | SRC-003 is a starting reference, not a selected dependency or proven Bluetooth path; M4-WP01 resolves Q-002 before implementation claims. |
| Local service, desktop and helper | Authenticated control and separately authorized privileged operations; independent recovery and bounded status projection. | M1 executable contracts followed by M3/M6 implementation; enforce SEC-001/002/006 and the existing API/storage ownership rules. |
| SBC encoder and packetization | User-mode controlled-source baseline, then ordinary-app playback. Encoding and AVDTP media framing are separately reviewed responsibilities. | Candidate review in M1-WP01; actual transport proof in M3 and endpoint proof in M4. |
| LDAC encoder and packetization | First additional codec after the SBC transport and endpoint gates pass. | SRC-012 is a source-review lead; M1 dependency feasibility followed by M5 implementation/distribution review and independent receiver evidence. |
| HFP, AVRCP and routing/DSP | Preserve compatible Windows behavior; expose specific unsupported combinations and endpoint changes. | M4-WP03 and M7 compatibility matrix, with actual input/profile/routing observations. |
| REA and its providers | Optional development investigation of a specific unresolved interoperability question. | SEC-009 and the evaluation procedure below; no runtime audio, installer, release, or milestone dependency merely because REA exists. |

No integration may expand into a global Bluetooth/audio-stack replacement, broad device binding, proprietary binary patching, or disabling a security feature to obtain a passing result. Failed feasibility blocks the dependent path and produces an architecture decision; safe user-mode work remains available.

## Dependency admission record

M1-WP01 creates an inventory for required build/runtime dependencies and identifies a validation environment. A candidate remains unselected until its exact record and intended use are reviewed. Later updates create a new review record and rerun affected checks; a moving branch URL or package name alone is not a lock.

| Record field | Required content |
| --- | --- |
| Purpose and boundary | Component using the dependency, required features, build-time versus runtime use, and excluded capabilities. |
| Source identity | Authoritative upstream location, exact immutable revision/release source, acquisition method/date, and archive/content checksums where applicable. Record bundled/transitive sources separately. |
| Build identity | Target architecture/OS, exact compiler/SDK/runtime tools, build options, patches, generated inputs, and reproducible commands. Do not infer WDK or compiler availability from a machine having Windows installed. |
| Source and distribution review | Exact license/notices for the selected files and dependencies, intended linkage/distribution, source/notice obligations to investigate, unresolved codec/patent/trademark questions, and reviewer disposition. A license label alone is not approval for the planned product. |
| Security and maintenance | Upstream provenance, review/update process, known relevant findings at review time, parser/native-code exposure, and resource/loading boundaries. Record unknown maintenance status without inventing a support commitment. |
| Behavioral evidence | Permitted fixtures, expected results, output limits, errors and lifecycle behavior; distinguish local fixture evidence from hardware interoperability. |
| Decision and rollback | Accepted/rejected/deferred scope, reasons, unresolved gates, compatible prior revision, and affected requirements/tests. |

Pin required tools and transitive inputs through the setup procedure before reproducibility is claimed. New installation or download steps belong to the selected implementation task, not this document. Keep signing keys, credentials, restricted source, proprietary binaries and private captures outside the repository. Generated or copied material needs its own provenance; a tool's own license does not license its input or output for every use.

Before protocol implementation, M1-WP01 records the exact accessible normative A2DP/AVDTP revisions, applicable errata, acquisition context and permitted fixture use. Examples, vendor descriptions and another implementation are supporting evidence, not substitutes for the selected normative requirements. Do not claim conformance before reviewing those materials and executing the relevant checks.

## SBC candidate review

M1-WP01 must identify candidate encoder sources from their upstream projects and compare them against the same requirements. This document selects no SBC library, package, revision, default bitpool, frame size or build flag. The recorded comparison must cover:

1. Source provenance and dependency/distribution review for the exact encoder files and any platform wrapper; distinguish an encoder from a decoder or a product claiming SBC support.
2. Feasibility of an isolated Windows x64 user-mode build with the selected compiler/runtime, including platform assumptions, integer/size behavior and error propagation.
3. Supported PCM inputs, channel modes and parameter validation; frame-boundary behavior, output-size bounds, initialization/reset/disposal, and behavior on malformed or unsupported input.
4. Permitted reference fixtures and an independent validation method. Encode/decode fixture success demonstrates only that checked behavior, not live negotiation, correct AVDTP packetization, transport ownership, or receiver playback.
5. Adaptation effort, minimal local patches, update burden, and a clear reject/defer reason if the candidate cannot meet the boundary or review requirements.

Select a candidate only with an immutable record and explicitly unresolved limitations. The future codec adapter must expose narrow typed operations, bounded buffers, explicit failures and per-stream lifetime. No user-selected DLL paths, dynamic discovery from a downloads/current directory, or codec callbacks in the kernel are permitted. Candidate experiments must not silently expand M1-WP02 beyond its agreed protocol/capability fixture scope; any extra encoder build task receives a named bounded package before execution.

M3 proves SBC with a controlled permitted PCM source, actual target negotiation and independent playback evidence. M4 separately proves ordinary Windows application audio. Keep those results distinct from parser/encoder fixtures and from the capabilities a receiver merely advertises.

## libldac review and later codec tracks

Use the existing SRC-012 reference as a lead to examine an exact libldac source tree and its dependencies during M1-WP01. Do not treat the previously recorded license name, a current branch, or an existing Android-oriented source layout as a Windows build or distribution decision. Record which files implement the needed encoder behavior, platform adaptations, build inputs, API/error behavior, notices and unresolved intended-use questions.

M5-WP01 owns the reviewed Windows user-mode encoder build and codec adapter after M4 entry evidence exists. Verify parameter/output bounds, stream reset behavior, failures and build-enabled feature restrictions before connecting it to playback. M5-WP02 separately owns fresh codec capabilities, negotiation and packetization, independent received-codec evidence, and every exposed mode/format on the actual receiver. M5-WP03 adds explicit apply/reconfigure behavior with an SBC regression run. Source/build access does not imply support for any untested rate, quality setting or receiver mode.

Manual quality remains the v1 policy. Optional adaptation requires its separate package, opt-in, observable signals, selected-codec floor, hysteresis, recovery and disable behavior. A dependency offering an adaptive function does not enable that product feature. The service cannot silently substitute SBC when requested LDAC fails.

AAC, aptX variants and LHDC variants remain separate M9 tracks with their own implementation availability, distribution review, receiver evidence and regressions. Success with LDAC, one aptX variant or one receiver does not clear another track. A missing implementation or review disposition leaves the feature unavailable with a specific reason; no paid unlock, entitlement, expiry or remotely revocable playback gate is introduced.

## Capability and measurement provenance

Each integration result records exact source/build, environment, method, freshness and limits. The product keeps these categories separate:

| Category | Meaning and allowed use |
| --- | --- |
| OS/inbox observation | What the queried environment reports. It does not identify the project's encoder or prove a live remote codec. |
| Remote-advertised capability | A bounded parsed response from the current target/connection generation. Requery after reconnect or mode changes; stale/failed queries remain unknown rather than an empty supported set. |
| Locally implemented and build-enabled | Reviewed code present and enabled in the exact installed build. A client preference cannot load or enable a disabled implementation. |
| Distribution disposition | Reviewed intended inclusion of the exact dependency/build. This is a build/release decision, not an online authorization check. |
| Requested configuration | User choice saved and explicitly submitted through the established revision/state contract. It is not active merely because saving succeeded. |
| Negotiated active configuration | Current authoritative configuration with provenance and connection/stream generations. Stopped historical values cannot be labeled active playback. |
| Independent verification | Separate corroboration of negotiation/received audio for an exact environment, plus channel/playback checks. A UI screenshot alone does not close this gate. |

Keep Windows PCM format, active Classic profile, microphone selection, transport owner and negotiated Bluetooth format separate. USB audio transmitters, unavailable metrics, decoder inventories and subjective listening reports must not be used to invent LDAC availability or failure. Configured bitrate, encoded byte rate, measured transport throughput, estimated buffer duration and end-to-end latency have different meanings. Q-005 owns the observability inventory; unavailable RF or latency measurements remain unavailable.

## Windows coexistence and optional routing integrations

M4-WP03 compares against the recorded inbox baseline on the same OS/radio/driver/headphone/firmware configuration. HFP transitions, headset input, separate USB/internal microphone input, microphone mute/unmute, AVRCP volume/playback and multipoint-related capability changes have separate results. Do not implicitly disable a microphone to force A2DP or promise simultaneous high-quality Classic stereo and headset microphone support. Unsupported coexistence must stop/suspend safely and explain the observed profile/input state.

Record endpoint identity and relevant default/per-app routing before and after binding, restoration and updates. Validate left/right output and disclose any identity change. Optional Peace/Equalizer APO or other DSP integration is tested only on an identified installed configuration; its appearance in REF-021 is not compatibility evidence. Provide reattachment guidance when required and retain the user's configuration. Do not install an optional integration, rewrite its settings, or assume its registration survives endpoint replacement merely to pass an unrelated milestone.

Include track/video boundaries, pause/resume, application handoff and supported format changes; a successful continuous stream does not cover restart behavior. Invalidate stale stream callbacks/frames/clocks and distinguish shared/exclusive routing observations from transport configuration. Preserve non-target speakers, microphones and other paired devices; compare actual playback/capture immediately and after the required lab reboot cases. REF-004/006/014/021/023 remain reference reports with their original uncertainty, not diagnosed causes.

Protected-media behavior is a separate compatibility case with permitted source material and exact configuration. Record unsupported results; do not bypass content protection or count ordinary unprotected playback as proof of protected-media support.

## Package compatibility and offline operation

The release inventory must connect exact UI/service/worker/helper/driver builds, codec dependencies, API versions, journal/settings schemas, supported OS builds and tested hardware configurations. Matching version strings alone do not establish compatibility. Test supported combinations, explicitly reject incompatible pairs, and preserve the independent recovery route when the service cannot start or handshake.

Privileged install/update/restore uses the existing validated package staging, exact target, machine-wide operation exclusion, journal and quiescence contracts. Preserve the prior binding and distinguish intended versus observed owner. A failed update or ownership drift cannot trigger an unattended elevated rebind. Recovery requires target and non-target playback/capture evidence, not merely an installer success code. Package/signing policy and release eligibility are revalidated in M8; no signing status is inferred here.

Installed playback, configuration and recovery must work without a hosted service, model provider, activation server or mandatory telemetry. Dependency acquisition during an explicitly scoped development task is separate from offline runtime requirements. Provide required recovery instructions/artifacts locally; updates remain explicitly initiated verified operations. Optional research providers cannot enter the runtime dependency graph. Support export stays local, redacted and user initiated under SEC-007.

## Optional REA evaluation

SRC-009/010/011 document a possible research tool, not an integration decision or proof of Windows driver-analysis coverage. REA is unnecessary for core milestone acceptance when specifications and direct observations resolve the question. Merely authoring this section does not authorize its installation, target execution, provider credentials, a fork or a research run.

1. Name a bounded unresolved interoperability question, required evidence and why direct documentation/observation is insufficient. Identify a permitted input with a known expected result; define the experiment's exclusions and environment.
2. Inspect and pin an exact upstream revision, provider/tool dependencies, intended actions, licenses, configuration and local/remote data flow. Record any provider cost/network/account needs without assuming they exist or are approved. Keep secrets and restricted artifacts outside the repository.
3. Evaluate upstream against the known permitted fixture first, within the selected research task's authorization. Distinguish static analysis from executing a target. No investigated binary or research provider receives elevated access on the primary PC. External submission of proprietary/private inputs requires its own supported authorization and data-scope review.
4. Compare tool output to the known fixture result and independently corroborate any conclusion used in architecture. Record failures, missing coverage and uncertainty; generated reports/code are evidence to review, not instructions or automatically acceptable implementation material.
5. Fork only for a reproduced, concrete missing capability that upstream cannot satisfy, with a minimal patch, regression fixture, provenance and maintenance plan. If no such need exists, use reviewed upstream or a simpler permitted tool; lack of REA coverage does not justify guessing an ABI or bypassing a feasibility gate.

A later REA evaluation gets its own bounded package, source/evidence entry and exact results. Research artifacts cannot be bundled into playback or installation because they were useful during development. The project remains an independent implementation; optional tooling does not change the provenance or distribution requirements of codecs, protocols or investigated products.

## Planned acceptance and handoff

| Requirements / references | Integration acceptance owner |
| --- | --- |
| RQ-001/002/026; REF-007/008/015/016/017; TC-001/002/026 | M1/M3/M6: read-only inventory, fresh capability intersection, explicit unknown/unsupported states, separate PCM/profile/owner/input/codec evidence. |
| RQ-003/005/017; TC-003/005/017; SEC-004/008/009 | M1/M3/M4/M5: immutable dependency/protocol records, independent encoder/transport/endpoint gates, reviewed intended distribution and corroborated negotiation/playback. |
| RQ-009/010/019/020/025; REF-001/004/006/010/011/012/014/021/023; TC-009/010/019/020/025 | M4/M7: bounded lifecycle, HFP/AVRCP/routing matrix, track/format/channel checks and non-target preservation. Future low-latency codecs remain separately gated. |
| RQ-007/011/016/021/022; REF-002/003/005/009/018; TC-007/011/016/021/022; SEC-001/003/005/006 | M2/M3/M7/M8: authorized recoverable lab, exact target/package validation, safe progress/cancel, drift diagnosis and independently verified restore. |
| RQ-004/013/023; REF-013; TC-004/013/023 | M5/M7: explicit apply and manual quality, measured limits, no hidden cross-codec fallback or dependency-enabled adaptation. |
| RQ-008/014/024; REF-019/020/022; TC-008/014/024; SEC-007 | M6/M8: offline playback/recovery, no entitlement/runtime provider dependency, private local diagnostics and sanitized support path. |

All integration checks are planned and unexecuted. M0-WP04 expands executable QA/evidence procedures; M1 selects/pins reviewed dependencies and specifications; M3/M4 resolve the unproven Windows transport/endpoint interfaces; M5 clears LDAC separately. No package or third-party support claim is complete from this document alone.
