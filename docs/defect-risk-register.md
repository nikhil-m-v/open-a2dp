# Defect and risk register

Baseline: 2026-10-21 (Asia/Calcutta). Ownership names below are responsibilities, not assigned people. One contributor may cover several roles; an independent review is required for changes crossing the kernel or installation boundary.

## Classification and intake

- A **reference report** is a supplied or externally documented issue. It is not proof our implementation has the issue.
- A **project defect** needs actual behavior versus an expected requirement on a specified project build.
- A **risk** is a possible future failure or unresolved feasibility concern. It does not imply a verified defect.
- Severity: critical = kernel compromise, unrecoverable system failure, or release-wide trust failure; high = loss of playback/recovery or milestone blocker; medium = degraded behavior with a usable workaround; low = limited impact.
- Likelihood is initially unknown until experiments support a rating. Priority reflects impact and project sequencing, not a fabricated failure probability.

For a report record: stable ID, source/date, product and exact version, OS/radio/headphone/firmware, steps, expected/actual behavior, frequency, logs with consent/redaction, severity, confidence, linked requirements/tests, owner, workaround, and resolution evidence. Keep identifiers and dumps private unless deliberately sanitized for publication.

## Supplied defects

No user defect list has been supplied. No local reference-app experiments or project runtime tests have run. There are therefore no confirmed project defects in this register. Append future reports as `REF-001`, `REF-002`, etc.; project reproductions use `BUG-001`, `BUG-002`, etc. Preserve the original report alongside later interpretation.

Vendor FAQ topics are research leads under SRC-008, not reproduced defects. When relevant, record their stated environment before converting them into regression cases.

## Prospective risks

| ID | Concern / impact | Priority; owner | Mitigation and closure evidence | Links |
| --- | --- | --- | --- | --- |
| RISK-001 | A safe inbox A2DP binding replacement or required channel ownership may not be achievable through the proposed path. | High; driver lead | M3 proves target-specific binding, L2CAP/AVDTP ownership, playback, and reversal. Failure stops dependent milestones and triggers design revision. | RQ-003, RQ-007; TC-003, TC-007 |
| RISK-002 | Endpoint timing and user/kernel PCM exchange may cause starvation, excessive delay, or audio discontinuities. | High; audio lead | M4 defines and measures the buffer contract; bounded queues, deadlines, and lifecycle stress. Tune against baseline before beta. | RQ-003, RQ-013; TC-003, TC-013 |
| RISK-003 | Codec availability or distribution terms may block the intended open-source bundle. | High; dependency reviewer | Separate technical/source/license/distribution states; review each exact dependency and codec before release. LHDC remains research until evidence resolves availability. | RQ-005, RQ-017; TC-005, TC-017 |
| RISK-004 | Driver/service input flaws could expose kernel memory, execute code, or escalate privileges. | Critical; security reviewer | SEC-001 through SEC-005, hostile-input tests, static analysis, focused fuzzing, and independent review on the lab. | RQ-008, RQ-012; TC-008, TC-012 |
| RISK-005 | Crash, failed update, driver binding conflict, or uninstall could strand Bluetooth audio. | High; installer lead | Record prior state, journal privileged changes, prove recovery with service unavailable, never remove unrelated packages. | RQ-007, RQ-011; TC-007, TC-011 |
| RISK-006 | Only a primary PC is available; experimental deployment could disrupt daily use. | High; project maintainer | Restrict current work to documentation/offline/user-mode/read-only tasks; satisfy M2 before deployment. No recovery instructions are treated as tested yet. | RQ-016; TC-016 |
| RISK-007 | HFP/microphone switching, AVRCP, multipoint, or a vendor radio driver may conflict with the custom A2DP path. | High; compatibility lead | Preserve Windows ownership of out-of-scope profiles, test transitions, requery capabilities, publish exclusions; block general alpha if no safe behavior exists. | RQ-009, RQ-010; TC-009, TC-010 |
| RISK-008 | RF interference and high bitrate may cause dropouts; reliable radio-quality measurements may not be exposed. | Medium; audio lead | Count observable queue/transport behavior honestly, lower settings explicitly, validate any adaptive algorithm; never invent RF/retransmission metrics. | RQ-004, RQ-006, RQ-013; TC-004, TC-006, TC-013 |
| RISK-009 | Signing eligibility, funding, HLK requirements, or security-policy incompatibility could block public installation. | High; release lead | Review current Microsoft policy early, plan certificate custody and tests, prove signed installation with required security features before release. | RQ-012, RQ-014; TC-012, TC-014 |
| RISK-010 | REA/provider output may be incomplete, inaccurate, or incompatible with the target. | Medium; research lead | Pin tools, test against a permitted fixture, retain evidence/confidence, verify conclusions independently; native tools remain an alternative. | RQ-017; TC-017 |
| RISK-011 | Addresses, names, audio, user paths, crash memory, or captures could leak through logs or support bundles. | High; security reviewer | No routine raw audio; bounded local logs; explicit export; redact and review bundles; keep dumps separate and private. | RQ-006, RQ-008; TC-006, TC-008 |
| RISK-012 | Observations or generated code could incorporate proprietary implementation material. | High; research lead | Track provenance, prefer specifications and behavior, separate restricted artifacts, review generated contributions and dependencies before publication. | RQ-017; TC-017 |
| RISK-013 | OS or vendor updates could invalidate a previously working driver combination. | High; compatibility lead | Record exact matrix, test update/recovery behavior, revalidate supported builds, avoid blanket Windows 11 compatibility claims. | RQ-001, RQ-010, RQ-011; TC-001, TC-010, TC-011 |
| RISK-014 | Protected audio may require additional endpoint/signing support and may not play through the proposed path. | Medium; audio lead | Test an authorized protected-playback scenario separately; state support limits; do not bypass protected-media mechanisms. | RQ-003, RQ-010; TC-003, TC-010 |
| RISK-015 | Multiple users or stale sessions could control another user's stream or overwrite device preferences. | High; service lead | Authenticate OS session identity, use exclusive controller ownership, release ownership on logoff, separate settings by SID, reject cross-session writes. | RQ-002, RQ-008, RQ-015; TC-002, TC-008, TC-015 |

All listed risks remain open. No mitigation is represented as implemented. Release-blocking risks require measured closure or an explicit approved scope reduction; a comment acknowledging a critical risk is not closure.

## Open research questions

| ID | Question | Resolving package and required artifact |
| --- | --- | --- |
| Q-001 | How can the selected A2DP device be bound to the project without broad radio replacement or conflicting profile ownership? | M3-WP01: binding/channel-ownership ADR with successful reversal on the lab. |
| Q-002 | Which endpoint/PCM mechanism meets scheduling and isolation needs with user-mode encoding? | M4-WP01: endpoint and buffer-contract ADR based on experiment. |
| Q-003 | What HFP/AVRCP interactions remain safe under that binding model? | M4-WP03: coexistence matrix and fallback/recovery behavior. |
| Q-004 | Which exact SBC/LDAC libraries and licenses are compatible with Windows and the eventual project license? | M1-WP01 and M5-WP01: pinned dependency inventory and distribution review. |
| Q-005 | Which measurements can the selected Windows/radio interfaces actually expose? | M3-WP02 and M7-WP02: observability inventory distinguishing unavailable, estimated, configured, and measured values. |
| Q-006 | What is the supported minimum Windows build and matching toolchain/signing route? | M1-WP01 records build targets; M2 records hardware baseline; M8-WP01 revalidates release eligibility. |
| Q-007 | What known defects should become explicit project requirements/regressions? | Next session when supplied: retain reference provenance and update PRD and test mapping. |

## Change log

| Date | Change | Evidence |
| --- | --- | --- |
| 2026-10-21 | Created prospective risk register and empty defect intake. | User-approved direction, SRC-001 through SRC-012; no runtime experiments. |
