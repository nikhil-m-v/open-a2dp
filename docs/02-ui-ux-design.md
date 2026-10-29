# UI and UX design

Status: proposed M0-WP02 design, 2026-10-30 (Asia/Calcutta). This document specifies future WPF behavior; no screen, accessibility test, codec control, installer, or recovery flow has been implemented. Read the [requirements](01-project-requirements.md), [architecture](03-architecture.md), [storage](04-database-and-storage.md), and [API contracts](05-api-and-contracts.md) together. The [system design](15-system-design.md) owns lifecycle invariants and [security](11-security.md) owns authorization.

## Product principles

The main task is to select one supported output, understand what audio path is actually active, and explicitly apply supported settings. The desktop UI runs as a standard user. Pairing and out-of-scope profile management remain Windows responsibilities. An elevated helper handles a separate, explicitly initiated install or restore operation on the exact target.

Saved preferences, current requests, verified active settings, and historical observations have visibly different labels. Saving a choice is never presented as successful negotiation or playback. A Bluetooth connection alone is never presented as proof that sound reaches the headphones. The UI reports the evidence available to the service and retains uncertainty where evidence is absent.

The application opens without a network connection, account, activation, subscription, or runtime AI service. Closing its window leaves a healthy stream running. An explicit Stop action stops playback; ownership release or Windows session logoff stops the session's stream. A newly opened window resynchronizes instead of replaying commands from its previous lifetime.

## Window structure

Use one resizable main window with four named destinations. Recovery remains reachable from the initial screen and every failure state, including when the service is unavailable.

| Destination | Primary content | Primary actions |
| --- | --- | --- |
| Devices | Relevant radio/output inventory; supported, unsupported, unknown and disconnected states; one selected target. | Refresh inventory, select target, open local getting-started guidance. |
| Playback | Current path and readiness; requested settings beside active settings; reason for unavailable controls. | Save preferences, Apply, Stop, Refresh capabilities when permitted. |
| Diagnostics | Observed errors and stage, exact local component versions, available counters and freshness, routing/endpoint changes. | Inspect details, create and preview a redacted local support export. |
| Recovery and help | Intended/observed driver owner, protected-operation summary, offline recovery instructions and compatibility/support status. | Open independent recovery tool, inspect operation, open local help. |

A persistent header shows selected device, current service availability, and a short text status. Never expose another session's device name, settings, audio activity details, or diagnostics through the header. Display only a generic ownership conflict where access is denied. Full machine-private recovery details require the helper's authorization.

Advanced identifiers and counters belong in an expandable details view. User-facing labels use ordinary language; internal field names in this document are contract references, not default interface copy. A copy-details action prepares a redacted preview, not an unredacted clipboard dump.

## Inventory and first use

1. Authenticate the local service and negotiate a compatible control version. Show locally packaged help and recovery access even if that fails.
2. Request a fresh, authorized read-only inventory. Show the detected OS build/radio and relevant outputs with explicit unknown fields; inventory does not install, bind, disconnect, pair, or change defaults.
3. Let the user select one target. Duplicate display names must be distinguishable using safe local labels/details. The service resolves the opaque `targetId`; the display name is never an installation identity.
4. Explain whether the target currently uses the Windows path, the project path, another owner, or an unknown owner. Showing saved codec preferences does not enable project controls on a Windows-owned device.
5. If the installed build and tested configuration permit a project binding operation, show its separate target-specific entry point and scope. Experimental deployment is unavailable on the primary-PC development path; M2/M3 must first establish the recoverable lab and permitted workflow.

| Inventory/capability result | Display and permitted action |
| --- | --- |
| Not yet queried | "Capabilities have not been checked." Allow an authorized refresh; do not label codecs unsupported. |
| Query failed or data stale | Show the reason and last observation as historical. Block Apply until the service confirms current applicability. |
| Fresh query excludes a codec/configuration | Explain the specific unsupported choice and show validated alternatives, without selecting one automatically. |
| Codec unavailable in this build | Explain that the build does not enable that implementation. Never offer a purchase or activation path. |
| USB audio transmitter owns the codec path | Explain that its firmware controls the transport and that this project cannot configure it through the planned Classic profile path. Do not offer ineffective driver installation. |
| Unsupported or unclassified transport | Explain the observed transport or missing evidence. Do not infer Classic A2DP capability from the device's product name. |

Codec support has distinct advertised, implemented, build-enabled/distribution-reviewed, requested and active states. The compact picker lists only choices permitted by the current supported intersection. A details view may explain unavailable choices, including research-only codecs, but must not render them as working features or give a release promise.

## Playback status and settings

Show these observations separately rather than compressing them into a single "quality" value:

| Display item | Meaning and fallback |
| --- | --- |
| Transport owner | Observed Windows, project, external firmware, other, or unknown ownership; intended owner is a separate field. |
| Bluetooth link | Observed connected/disconnected/unknown link state, with freshness. |
| Active profile | A2DP, HFP, another observed mode, or unknown. This is independent of whether a headset microphone exists. |
| Windows audio format | Observed PCM format at the Windows endpoint, or unavailable. It is not the negotiated Bluetooth codec/rate. |
| Bluetooth codec and format | Verified current negotiated configuration, or no active configuration/unknown. Historical values are labeled "Last verified". |
| Endpoint and stream | Endpoint availability and stream progress are separate. A visible endpoint or submitted bytes do not prove audible reception. |
| Selected microphone | Observed headset, separate input, or unknown, without silently changing/muting it. Explain profile effects where known. |
| Metrics | Distinguish configured bitrate, measured throughput, estimated queue duration and unavailable measurements. No invented RF strength, latency or reception counters. |

If Windows PCM is 96 kHz and the active transport format is 48 kHz, display both values with those labels. Neither the Windows setting nor absence from a general decoder list establishes LDAC activity. Do not promise an audible improvement from a codec name, bitrate, or bit depth.

The requested-settings panel has a codec selector and only the validated parameters relevant to that codec. V1 quality is manual. Adaptive quality is absent or explicitly unavailable until its separate implementation and test gates pass. Choosing a lower quality or SBC is a user action; the UI never treats degraded measurements as permission to switch codecs.

### Save and Apply flow

1. Editing creates a visible unsaved draft. Validate locally for useful feedback, then rely on service validation for authoritative eligibility. Display errors beside the relevant field and in an accessible summary.
2. Save persists a per-user/device preference through the storage contract. Show "Saved; not applied" with the returned `settingsRevision`. A failed write leaves the draft visible and reports that it was not saved.
3. Apply uses a successfully saved request. If the draft has unsaved changes, first save successfully as part of the explicitly labeled "Save and apply" action; failed Save prevents dispatch. Immediately before dispatch, the UI checks the saved revision and complete content against the request the user selected. A detected storage conflict requires refresh/reconciliation and another explicit action, not overwriting another window's saved values.
4. Before dispatch, explain any expected interruption: "Playback will stop while these settings are applied." For a configuration that requires reconnection, state that explicitly. Bind the operation to the current selection; switching the visible device cannot redirect it.
5. While applying, show the actual reported stage and retain requested versus last-verified values. Disable competing mutations for that target. Request acceptance is "Applying", not success.
6. On confirmed negotiation, show the new active settings. Report playback resumed only when the service reports the corresponding stream state. Separate negotiation success from independent hardware verification used for milestone evidence.
7. If negotiation or restart fails, retain the saved request, show stopped/faulted state and the reason, and label any previous configuration historical. Offer explicit Retry after cleanup or an explicit alternative choice. Never claim the previous codec remains active without fresh evidence.

`settingsRevision` correlates the operation with the user's saved request; it is not a service-wide concurrency token. The service receives the complete requested values and checks its own `snapshotSequence`, ownership, capabilities and generations before accepting them. A rejected stale service expectation means the request was not applied and the UI must refresh. The service cannot independently detect a later Save by another window, and a local file change alone cannot produce a service `StaleState` result.

Once accepted, Apply uses the captured complete request. A later Save never changes that operation or its target. If the current saved revision differs from the submitted/applied revision, show the saved request separately with "Saved changes have not been applied"; do not call the accepted operation stale or failed solely for that difference. Closing and reopening the UI during Apply resumes observation of the existing operation; it does not issue a second Apply.

## State presentation and session ownership

These labels map to the application lifecycle; they are not normative AVDTP protocol states.

| State or condition | Required presentation and next action |
| --- | --- |
| `StandardDriver` | "Using the Windows audio path." Project codec preferences are inactive; inventory/help remain available. |
| `Ready` | "Project device ready; playback stopped." Explain available explicit actions based on fresh capabilities and ownership. |
| `Discovering` | "Checking device capabilities." Mark old results stale and wait for this generation's response. |
| `Configuring` | "Applying requested settings." Show reported stage; no success toast on mere command acceptance. |
| `Streaming` | Show the authoritative active settings and available progress observations; avoid claiming end-to-end reception from local progress alone. |
| `Stopping` | "Stopping playback." Wait for confirmed cleanup; do not re-enable conflicting actions from a UI timer. |
| `Faulted` | "Playback stopped" plus the actionable reason and safe retry availability. Retain details and recovery access. |
| `RecoveryRequired` | Explain the observed binding/package uncertainty. Block streaming/configuration and offer the independent recovery route. |
| Service unavailable or incompatible | "Current audio state cannot be confirmed." Mark previous observations stale; disable service mutations, retain local help and independent recovery. |
| Another session owns control | "This device is controlled by another Windows session." No forced takeover or disclosure of that session's state. |
| Intended/observed ownership mismatch | Show both states with freshness and "Driver ownership changed." Offer a scoped explicit recovery/binding action; never silently reinstall. |

The service validates the Windows caller and `controllerLease`. A control connection is not itself stream ownership. Closing a window does not release ownership; explicit release and logoff require cleanup before another session acquires it. A second authorized window in the same session follows the API's serialization/conflict policy. A storage revision conflict requires reloading/reconciling saved preferences; a service snapshot conflict requires refreshing service state. Neither permits assuming the current window's draft wins.

The UI tracks `serviceEpoch`, `targetId`, `connectionGeneration`, `streamGeneration`, `settingsRevision`, `requestId`, `operationId` and `snapshotSequence`. A service restart or event gap requires a full snapshot. Older or out-of-scope events cannot restore an old active codec, update the newly selected device, or replace the current operation's error. Duplicate responses are observational only; the UI does not retry mutations automatically after an uncertain disconnect.

Track/video boundaries and app handoff may change stream generation while the Bluetooth connection remains intact. The display must not flash a new connected-device success message or tell the user to reconnect for each normal stream boundary. Sleep/resume invalidates stale observations until the service has revalidated binding and capabilities. Profile transitions explain why A2DP may stop or become unavailable; the UI never disables a microphone to force stereo playback.

## Independent install and recovery experience

The main window links to an independently launchable local helper with its own compatible read/validation path. Its executable location is installer-controlled, not taken from preferences or a user-supplied command. An offline shortcut and packaged instructions make it discoverable when the UI or service cannot start. Help explains where to find the prearranged recovery material if Windows cannot start; the helper itself is not claimed to recover an unbootable OS.

The helper flow has these stages:

1. Inspect protected state and live OS state. Show the exact resolved target, current binding, intended owner, previous recorded binding and any uncertainty. Reject an ambiguous target rather than guessing from its display name.
2. Present the fixed action and scoped changes, such as restoring the recorded prior Windows path. Explain any validated package/version constraints and required reboot, without a generic "repair everything" action.
3. Request elevation for that explicit action. Denied or canceled elevation leaves state unchanged and returns a readable result. The ordinary UI stays unelevated.
4. Show journal-backed `operationId`, actual stage, most recent progress and cancellation availability. Use indeterminate progress when completed work cannot support a percentage. A stall means "Progress has not advanced; state needs inspection", not automatic failure or completion.
5. A cancel request is acknowledged separately from completed cancellation. During a noncancelable PnP step show "Cancellation requested; waiting for a safe stopping point". Do not kill the helper/PnP operation or offer an unsafe force-stop button. Closing the view does not cancel the operation; reopening reads its current journal state.
6. After mutation, report binding verification, endpoint enumeration, and playback/capture validation separately. "Binding restored; playback not yet checked" is distinct from completed recovery. Reboot-pending work remains pending until the journal and live state have been reconciled after reboot.
7. Guide target playback plus unrelated output/input checks against the recorded baseline, immediately and after a required test reboot. Record failed, pending and user-reported checks honestly; a user confirmation is not independent measured playback evidence.

The helper must expose an interrupted-operation state even when the service is down. It revalidates rather than blindly replaying the last command. Unknown journal versions/corruption keep the uncertainty visible; there is no broad driver deletion, pairing reset, security-setting change, or automatic privileged takeover as a shortcut. Exact stage deadlines and cancelable boundaries must be experimentally defined before implementing this flow; this document invents no timeout.

## Routing, diagnostics and help

When endpoint identity changes, show the old/new local endpoint relationship and explain the affected output selection. Guide the user to reselect the output in Windows or their app and, for an independently tested integration, to reattach their EQ/APO. Do not edit third-party configuration, reset global audio defaults, or imply every integration is supported. Routing, channel mapping and codec status remain distinct diagnostic categories.

The diagnostics view shows the current stage/reason, freshness and available counters with units. State resets and missing telemetry appear as discontinuities/unavailable values rather than zero errors. High-rate telemetry is coalesced for readability and cannot flood announcements or block playback. A copy/export workflow previews redacted content, chooses a local destination, records omitted data, and never uploads automatically. Raw audio, link keys, credentials and full crash dumps are excluded from ordinary export; opt-in raw investigations are a separate future workflow under SEC-007.

Packaged help includes getting started, capability explanations, restore steps, unsupported-configuration explanations and an offline report template. Before release, populate the public issue route, private security contact and support-version policy. This design does not invent a currently staffed support channel or response time. Public reporting prompts the user to inspect the redacted export; security disclosures must not default to a public issue. Recovery remains usable without a maintainer reply.

## Accessibility and interaction acceptance

- Use standard WPF controls where possible; define accessible names, roles and labels for custom elements. Every action, list and details section is keyboard reachable in a predictable order with visible focus, without requiring hover, drag gestures or color recognition.
- Associate each control with its label, validation message, units and disabled-state explanation. Make reasons reachable even when the affected control itself is disabled. Status includes text and icon meaning; color alone carries no state.
- Preserve focus during background refresh. Keep an edited value visible on validation failure; move focus to an error summary only after the user's failed action, with navigation to the affected control. Modal dialogs return focus to their initiating control.
- Announce meaningful transitions and terminal failures politely; use urgent announcements only for immediate action requirements. Do not announce every counter tick, percentage change or repeated identical event. Persistent errors cannot exist only in a transient toast.
- Support system text scaling, high contrast and resize/reflow without clipping Apply, Stop or Recovery. Long device names and translated strings wrap or expose an accessible full value. Do not assume a fixed pixel width or English-only label length.
- Describe interruption, pending cancellation, stale observations and recovery checks using plain text. Confirmation buttons name their action and target; generic "Yes" must not conceal whether a driver change or a saved preference is being approved.

M6 must verify these behaviors with keyboard-only navigation, a screen reader, enlarged text, high contrast, duplicated device names, long labels, service-down launch, denied elevation, and focus retained across asynchronous refresh. The current document is a specification, not a passed accessibility audit.

## Requirement and reference coverage

The following are planned UX acceptance additions. M0-WP04 expands procedures; the listed runtime scenarios remain unexecuted. Reference reports remain unverified and do not establish root causes.

| Requirement / planned scenario | UI behavior to verify | Reference reports |
| --- | --- | --- |
| RQ-001, RQ-002, RQ-026; TC-001, TC-002, TC-026 | Unknown/stale capability, unsupported USB path, build availability and PCM/profile/input/codec separation each have distinct explanations. | REF-007, REF-008, REF-015, REF-016, REF-017 |
| RQ-004, RQ-005; TC-004, TC-005 | Save never means active; interrupted/failed Apply retains the request and honest active state; alternative codec selection is explicit. | REF-008, REF-012 |
| RQ-006, RQ-009; TC-006, TC-009 | Distinguish connection, endpoint readiness and stream progress; resume refreshes state; service-down/worker faults retain recovery. | REF-001, REF-002, REF-003, REF-007 |
| RQ-007, RQ-011, RQ-016; TC-007, TC-011, TC-016 | Independently launchable offline recovery, lab-only experimental deployment, interrupted/reboot-pending verification and truthful recovery results. | REF-003, REF-005, REF-023 |
| RQ-013, RQ-019; TC-013, TC-019 | Measured versus unavailable counters, normal stream boundaries, format transitions, channel checks and explicit manual quality changes. | REF-010, REF-011, REF-012, REF-014, REF-015 |
| RQ-020; TC-020 | Recovery comparison includes unrelated outputs and separate microphone capture/mute state; no broad cleanup action. | REF-004, REF-006, REF-023 |
| RQ-021; TC-021 | Intended/observed owner mismatch is visible across restart/reconnect/update without an automatic privileged rebind. | REF-018 |
| RQ-022; TC-022 | Journal-backed stage/identity, indeterminate progress, stall handling, safe cancellation and reattachment after UI close. | REF-009 |
| RQ-023; TC-023 | Manual quality stays explicit; unavailable adaptive functionality cannot silently alter the request. | REF-013 |
| RQ-014, RQ-018, RQ-024; TC-014, TC-018, TC-024 | Offline use/help, accessible service-down recovery, no entitlement screen and accurate support/disclosure routes before release. | REF-019, REF-020, REF-022 |
| RQ-025; TC-025 | Endpoint mapping and device-specific routing/EQ reattachment guidance preserve third-party settings and disclose untested integrations. | REF-014, REF-021 |
| RQ-008, RQ-015, RQ-017; TC-008, TC-015, TC-017 | Cross-session privacy, revision conflicts, corrupt/newer preference explanation, redacted export preview and build provenance. | REF-018, REF-022 |

The API contract governs operation results and identity, storage governs persistence/migrations, and the architecture governs ownership. M6 implements the UI only after the relevant transport and recovery evidence exists; an earlier fixture-backed UI must be visibly labeled as such and cannot count as hardware proof.
