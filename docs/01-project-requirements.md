# Project requirements

Status: batch 1 design baseline, 2026-10-21. No runtime requirement has been verified. Read the [evidence register](source-evidence-register.md) for source limits and the [roadmap](08-project-implementation.md) for sequencing.

## Purpose and users

Create an independent open-source Windows A2DP source implementation that lets users choose compatible audio codecs and understand the settings actually in use. Start with SBC to validate the transport and system-audio path, then deliver LDAC. Users should retain a reliable route back to Windows audio.

Primary users are Windows 11 x64 headphone/speaker owners seeking control over Bluetooth Classic audio. Secondary users are contributors investigating compatibility and maintainers diagnosing failures. The initial product serves one interactive user controlling one active output. Contributor workflows may use analysis tools; playback itself requires no AI or network service.

Success means an ordinary Windows application can render audio through our endpoint and Bluetooth path, the headphones receive the negotiated codec, and installation/recovery are demonstrated on a published test configuration. A functioning dashboard, encoded file, or virtual audio meter alone does not satisfy this goal.

## Scope and constraints

| Area | Initial decision |
| --- | --- |
| Platform | Windows 11 x64. Exact supported builds are recorded after toolchain and hardware validation, not inferred from the product name. |
| Profiles | Bluetooth Classic A2DP source playback. Windows remains responsible for pairing and out-of-scope profiles. |
| Device scope | One active target; store preferences for multiple paired devices without simultaneous streaming. |
| Codec sequence | SBC baseline, LDAC first added codec; other codecs individually gated later. |
| Application | Local WPF desktop UI, native C++/WDK components as needed, user-mode encoding where feasible. |
| Operation | Offline after installation; no account, activation, subscription, expiry, or runtime AI dependency. |
| Storage | Local versioned settings; no database server. Detailed schema belongs to batch 2. |
| Development safety | Only the primary PC is currently available. Experimental drivers require a separate recoverable lab. |
| References | Existing synced material is read-only. Behavior-first research; preserve implementation provenance. |

Deferred: Windows 10, ARM64, browser control panel, LE Audio/LC3, A2DP sink, broadcast/group audio, simultaneous outputs, headset firmware changes, vendor-app features, multipoint management, high-quality simultaneous Classic microphone/stereo promises, and automated driver deployment. Unmodified protected-media playback is a separate compatibility case, not an initial guarantee.

Windows codec support depends on OS and hardware. Detect rather than assume an SBC/AAC-only baseline. [Microsoft Bluetooth Classic audio](https://learn.microsoft.com/en-us/windows-hardware/drivers/bluetooth/bluetooth-classic-audio)

## Requirements and acceptance traceability

All scenarios are planned and unexecuted. IDs remain stable as later batches expand them. A milestone can pass only with a recorded result and environment; documentation is not runtime evidence.

| ID | Required behavior / acceptance | Milestone | Planned scenario |
| --- | --- | --- | --- |
| RQ-001 | Enumerate relevant local audio/radio devices read-only; display OS build and adapter identity. Clearly distinguish detected, unknown, and unsupported configurations, including USB audio transmitters and LE-only devices. No binding changes during inventory. | M1 | TC-001: inventory actual target and synthetic unsupported/unknown cases; inspect before/after driver state. |
| RQ-002 | Separate advertised, implemented, distribution-eligible, requested, and active codec state. Requery capabilities on a fresh connection. Mark unqueried/stale data and reject a request outside the supported intersection. | M3/M6 | TC-002: capability fixtures, changing capabilities, stale connection generation, cross-session control, and unsupported codec requests. |
| RQ-003 | Send a controlled source over SBC, then render an ordinary Windows app through the endpoint to headphones. Audio reception and negotiated configuration need evidence beyond the UI. | M3/M4 | TC-003: controlled-source transmission, ordinary-app playback, channel/format checks, 30-minute continuous playback; record protected-media limits separately. |
| RQ-004 | Accept valid codec-specific settings, preserve requested versus active values, and show whether reconnection is required. Apply explicitly; on failure retain the last verified state or report stopped playback. Never report success solely because settings were saved. | M5/M6 | TC-004: valid/invalid/unsupported parameters, failed renegotiation, interrupted apply, explicit fallback, and reconnect-required status. |
| RQ-005 | Negotiate and stream LDAC using a reviewed implementation. Offer only validated supported configurations. SBC remains an explicit selectable compatibility option. | M5 | TC-005: LDAC-capable and non-LDAC devices; supported quality modes; independent negotiated codec verification; 30-minute LDAC playback. |
| RQ-006 | Expose connection/stream state, actual negotiated parameters, queue/transport counters where measurable, and bounded diagnostic export. Distinguish configured rate, measured throughput, estimated buffering, and unavailable metrics. | M3/M6 | TC-006: unavailable metrics, counter discontinuity, UI/service restart, export redaction, and log rotation. |
| RQ-007 | Install/change binding only for the intended target and restore the previous Windows path through an independent privileged recovery tool. Uninstall must not remove unrelated packages or require a functioning audio worker. | M3/M8 | TC-007: target identity checks, denied elevation, binding failure, interrupted operation, service unavailable, successful standard-driver playback after recovery. |
| RQ-008 | Enforce user/service/kernel boundaries and protect audio and identifiers. Unauthorized requests, malformed inputs, and untrusted paths must not gain privilege or disclose another session's data. | M1/M7 | TC-008: SEC-001 through SEC-009, hostile local callers, malformed buffers, cross-user access, and support-bundle privacy checks. |
| RQ-009 | Handle loss/reconnect, radio removal, sleep/resume, worker failure, and HFP/microphone transitions without stale streaming or resource leaks. Any unsupported coexistence must be visible, safe, and bounded. | M4/M7 | TC-009: 20 disconnect/reconnect cycles, 10 sleep/resume cycles, radio removal, microphone activation, crash injection, and pending-I/O cancellation. |
| RQ-010 | Publish compatibility only for tested OS/radio/driver/headphone/firmware combinations. Record AVRCP/volume, HFP, multipoint-related capability changes, and protected-media results separately. | M4/M7 | TC-010: matrix entries with evidence and exclusions; compare inbox behavior under the same configuration. |
| RQ-011 | Maintain driver-installation state sufficient for recovery. Make repeated operations safe, detect partial installation, and test update/downgrade compatibility before release. | M7/M8 | TC-011: five install/restore cycles, upgrade/downgrade, power/process interruption, mismatched driver/service versions, and rollback revalidation. |
| RQ-012 | Release binaries load with required Windows security features enabled. Playback does not require administrator rights, test signing, or disabling protections. | M7/M8 | TC-012: standard-user playback, Secure Boot and Memory Integrity enabled, applicable driver verification, signing checks, and absence of unresolved critical/high security defects. |
| RQ-013 | Bound memory and latency under backpressure. Define measured performance thresholds from the lab baseline before beta; no unmeasured latency/quality promises. | M4/M7 | TC-013: encode/queue timing, CPU/memory, stressed/interfered playback, saturation and monotonic memory-growth investigation. |
| RQ-014 | Playback and configuration work without network access after installation. Releases include source/build instructions, dependency notices, tested compatibility, and a secure manual installation/update path. | M6/M8 | TC-014: offline playback/configuration, clean-machine build reproduction, package signatures, notices, and release checklist. |
| RQ-015 | Store per-user/device preferences with versioning, safe atomic replacement, and explicit migration handling. Corrupt or newer unsupported settings cannot silently select unsafe values or corrupt other users' data. | M6 | TC-015: interrupted write, malformed file, unsupported version, stale device ID, file permissions, and cross-user isolation. |
| RQ-016 | Limit current-machine work to approved documentation, offline/user-mode builds and tests, and read-only inspection. Driver installation, driver verifier, binding/boot/security changes, and destructive recovery wait for the lab and explicit target authorization. | M0/M2 | TC-016: evidence of isolated target, recovery rehearsal, baseline inventory, and authorization before each deployment task whose scope requires it. |
| RQ-017 | Track source/observation provenance and exact dependency/tool revisions. Clear distribution terms for each bundled codec; optional REA conclusions require independent corroboration. | M0/M1/M5/M9 | TC-017: provenance review, pinned dependency inventory, license/notices review, and reproducible investigation fixture if REA is used. |
| RQ-018 | Provide keyboard-accessible device/codec controls, meaningful error messages, explicit changes, and persistent requested/active distinction. Recovery guidance remains accessible if the service is down. | M6 | TC-018: keyboard/screen-reader checks, unsupported-device explanation, service-down view, interrupted apply, and recovery-tool discovery. |
| RQ-019 | Survive track/video boundaries, pause/resume, application handoff and supported format changes without stale frames/clocks or reconnect requirements. Preserve intended left/right mapping. | M4/M7 | TC-019: at least 50 combined track/video/start-stop transitions, 20 alternating 44.1/48 kHz requests where supported, shared/exclusive app handoff, and known left/right fixtures; verify recovery without manual reconnect. REF-011/014. |
| RQ-020 | Installation, configuration and restoration preserve unrelated internal/wired/USB outputs, separate microphones and other paired devices. Compare against the recorded baseline immediately and after reboot. | M3/M4/M7 | TC-020: non-target inventory and playback/capture before/after install, apply, restore, reboot and sleep; separate USB/internal mic mute/unmute during calls. Require recorded user changes to explain intentional baseline differences. REF-004/006/023. |
| RQ-021 | Honor the user's explicit driver-owner choice. Detect intended-versus-observed binding drift after restart/reconnect/update and explain it; never silently reinstall/rebind or tie audio to activation/expiry. | M3/M6/M7 | TC-021: select inbox path, restart UI/service/OS, reconnect and simulate update-induced drift; no project takeover and no hidden repair; explicit authorized action required for rebind. REF-018. |
| RQ-022 | Show privileged-operation stage, correlation ID and progress; detect stalls using documented measured deadlines. Permit cancellation only at a safe operation boundary and retain recoverable pending state rather than forcing an in-flight PnP operation to stop. | M3/M7/M8 | TC-022: injected slow/hung helper/PnP completion, denied elevation, close/reopen UI, cancel at each supported boundary, interrupted journal, and recovery with service unavailable. REF-009. |
| RQ-023 | Manual codec quality is the v1 default. Optional later adaptation is opt-in, stays within the selected codec, exposes cause/current quality, and uses tested minimum quality, hysteresis and recovery rules. Disable it restores the explicit manual preference. | M5/M7 | TC-023: manual quality remains fixed; if adaptation is delivered, test brief/sustained poor conditions, unavailable/stale measurements, recovery to configured quality and disable/restart. Otherwise confirm the adaptive feature remains absent/disabled and mark algorithm tests deferred. REF-013. |
| RQ-024 | Supply offline onboarding/recovery, a structured public bug-report route, private security disclosure contact and support-status policy before release. Diagnosis and recovery cannot depend on a maintainer reply, purchase flow or hardware entitlement. | M6/M8 | TC-024: follow offline docs on a clean target; submit a sanitized sample report through the documented route; verify disclosure contact and unsupported-version handling. Do not promise unstaffed response times. REF-019/020/022. |
| RQ-025 | Track endpoint identity before/after binding and updates. Preserve identity/settings where feasible; otherwise disclose changes and provide device-specific app-routing/EQ/APO reattachment instructions without rewriting third-party configuration. | M4/M6/M7 | TC-025: record endpoint IDs, default/per-app routing and optional Peace/Equalizer APO attachment before/after binding/restore/update; verify left/right output. Publish optional integration support only after that configuration is tested. REF-021/014. |
| RQ-026 | Explain Windows PCM format, transport owner, active Classic profile, negotiated Bluetooth codec/format and actual selected microphone separately. Give specific LDAC-unavailable reasons; show unknown when evidence is missing. | M3/M6 | TC-026: absent A2DP, HFP-only active mode, USB audio transmitter, PCM 96 kHz with other transport format, unsupported 96 kHz receiver, stale capabilities, no decoder-list entry and separate microphone selection. None alone falsely proves or disproves LDAC. REF-007/008/015/016/017. |

The playback and cycle-count targets are project acceptance targets, not vendor performance claims. Passing them is necessary but insufficient for beta: M7 adds longer soak and broader matrix testing. Audible improvement is not guaranteed; independent codec/channel verification comes first. Optional listening comparisons must control source material, level and processing and cannot replace objective transport evidence.

TC-009 additionally requires resume without a mandatory reboot, separate headset versus USB/internal microphone selection/mute/unmute, and diagnostic identification of the selected input/profile. TC-007/011 must confirm restored target endpoint and non-target playback immediately and after reboot, including interrupted installation and service-unavailable recovery. TC-014 includes clock/date advancement and rediscovery after hardware changes without entitlement or expiry behavior. TC-010 records exact Windows updates and endpoint/DSP state; the Reddit mention of 24H2 is not a measured version-specific failure rate.

## Codec availability policy

| Codec | Planned position | Evidence needed before offering it |
| --- | --- | --- |
| SBC | Transport and endpoint baseline | Reviewed encoder, applicable specifications, interoperable packetization, and supported parameters. |
| LDAC | First added codec | Exact source/license review, working Windows build, negotiation, quality configuration, and receiver evidence. |
| AAC | Later independent track | Encoder and distribution review, supported A2DP configuration, and headset interoperation. |
| aptX Classic / HD | Later individual tracks | Exact implementation/provenance, variant-specific capabilities, distribution review, and hardware tests. |
| aptX LL | Separate later track | Same gates plus independently measured latency; no inference from Classic/HD. |
| aptX Adaptive / Lossless | Research only | Implementation/interface availability, required hardware, rights, and variant-specific tests. |
| LHDC variants | Research only | Version-specific specifications/implementation access, distribution eligibility, and compatible receivers. |

An installed library does not imply a distribution decision, working transport, or headphone support. Support for one aptX or LHDC variant does not imply another. Do not label research-only codecs as selectable or promise release dates.

## Reference behavior and defects

The reference vendor documents per-device driver selection, codec settings, and the codecs listed in SRC-007/008. These are research leads. No reference binary/version has been inspected, and no feature parity claim is made.

The supplied [Reddit discussion](https://www.reddit.com/r/SonyHeadphones/comments/112zjf9/alternative_a2dp_driver_for_pc/) is SRC-013, with REF-001 through REF-023 classified in the [register](defect-risk-register.md). Reports are externally sourced and unverified locally; severe allegations, ambiguous product attribution, expected HFP limitations, subjective impressions, and integration caveats are explicitly distinguished. Positive replies prevent treating the thread as universal failure evidence. No root cause or reference-version defect is established.

Every reference entry maps prevention to requirements and planned scenarios. Additional intake appends stable IDs with source and environment; actual project reproductions use `BUG-*` with build-specific evidence. Runtime testing and local reference-app observation remain unexecuted.

## Release acceptance and change control

M3 must prove transport ownership and reversal; M4 must prove ordinary-app playback; M5 must prove LDAC reception. Failure of an earlier gate blocks dependent implementation and triggers a recorded architecture change.

Before beta, approve exact performance thresholds and test durations using the recorded inbox/project baseline. Before public release, close applicable test scenarios, resolve critical/high security findings, publish supported configurations, confirm signing eligibility, and approve dependency/project licensing. Reduced scope must update this PRD and affected tests before declaring success.

Unknowns Q-001 through Q-007 remain explicit research tasks. This baseline approves staged investigation; it does not assert that the proposed driver design or codec redistribution is already feasible.
