# Source and evidence register

Last reviewed: 2026-10-21 (Asia/Calcutta). This is a documentation research log, not a record of local reverse engineering.

## Evidence rules

For each future investigation, record source/version or commit, acquisition date, lawful access context, tool versions, file hashes where applicable, exact question, observation, conclusion, confidence, limitations, and related requirement/risk/test. Separate quoted facts from inference. Store no proprietary binaries, copied code, license material, device identifiers, or unredacted captures in the public repository.

Documentation links can change. Pin source commits and tool versions before running experiments; no package or revision is pinned or installed by this batch. Recheck distribution and signing requirements at the relevant milestone.

## Sources consulted

| ID | Source | Finding used | Limits and follow-up |
| --- | --- | --- | --- |
| SRC-001 | [Microsoft: Bluetooth Classic audio](https://learn.microsoft.com/en-us/windows-hardware/drivers/bluetooth/bluetooth-classic-audio) | Windows supports SBC and aptX Classic; AAC is available on Windows 11; aptX Adaptive depends on selected compatible systems. Classic A2DP is an output profile; microphone use involves HFP. | OS documentation, not this machine's inventory or a compatibility guarantee. Record exact OS/radio capabilities in M1/M2. |
| SRC-002 | [Microsoft: Using the Bluetooth driver stack](https://learn.microsoft.com/en-us/windows-hardware/drivers/bluetooth/using-the-bluetooth-driver-stack) | Profile-driver selection uses PnP/INF matching. Kernel profile drivers use BRBs/IOCTLs for Bluetooth operations, including L2CAP. | Does not establish that our driver can safely coexist with or replace the relevant inbox binding. M3 must prove that. |
| SRC-003 | [Microsoft: SysVAD sample](https://learn.microsoft.com/en-us/samples/microsoft/windows-driver-samples/sysvad-virtual-audio-device-driver-sample/) | Provides a WDM/WaveRT audio-endpoint starting reference and separates building from target-machine deployment. | A sample endpoint is not a working Bluetooth sender. Review the source license, current WDK compatibility, and PCM boundary before reuse. |
| SRC-004 | [Microsoft: Driver signing policy](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/kernel-mode-code-signing-policy--windows-vista-and-later-) | Modern kernel-driver distribution requires an appropriate Microsoft signing route. | Older compatibility exceptions are not the release strategy. Confirm current policy and applicable tests before M8. |
| SRC-005 | [Microsoft: Driver code signing requirements](https://learn.microsoft.com/en-us/windows-hardware/drivers/dashboard/code-signing-reqs) | Hardware dashboard registration/submission has certificate requirements. | Revalidate organizational eligibility, costs, certificate custody, and current submission rules. No account or certificate acquired. |
| SRC-006 | [Chrome: Web Bluetooth capabilities](https://developer.chrome.com/docs/capabilities/bluetooth) | Browser API access centers on BLE/GATT, not replacing the Windows Classic A2DP audio path. | The native-app recommendation is our architectural conclusion. A later web UI would still require native components. |
| SRC-007 | [Alternative A2DP Driver product page](https://www.bluetoothgoodies.com/a2dp/) | Advertises per-device driver selection, codec configuration, and SBC/AAC/aptX/HD/LL/LDAC support. | Retrieved page identifies itself as an offline copy. Vendor claims are not independently tested; exact version is not recorded. No local installation inspected. |
| SRC-008 | [Alternative A2DP Driver FAQ](https://www.bluetoothgoodies.com/a2dp/faq/) | Describes source-only A2DP, capability-dependent codec selection, hardware constraints, and no LHDC support in its stated plans. | Retrieved page identifies itself as an offline copy; some codec discussion is dated. Do not use vendor licensing comments as authoritative permission for our distribution. |
| SRC-009 | [REA repository](https://github.com/morluto/rea) and [README](https://raw.githubusercontent.com/morluto/rea/main/README.md) | Optional analysis orchestration across several tools and target types. | Marketing breadth is not proof of Windows driver-analysis coverage. No clone, setup, execution, fork, or target analysis performed. |
| SRC-010 | [REA license](https://raw.githubusercontent.com/morluto/rea/main/LICENSE) | MIT license for REA itself. | Does not license investigated products, providers, codecs, or copied output. Preserve notices if reused. |
| SRC-011 | [REA Windows Ghidra support](https://raw.githubusercontent.com/morluto/rea/main/docs/windows-ghidra-p0.md) | Windows native analysis support has provider, version, and release boundaries. | Evaluate a pinned version against a known permitted fixture before using it for project conclusions. |
| SRC-012 | [AOSP libldac license](https://android.googlesource.com/platform/external/libldac/+/refs/heads/main/LICENSE) | The referenced implementation carries Apache License 2.0. | Assess complete source provenance, build dependencies, notices, patents, trademarks, and redistribution for our planned use; no distribution clearance recorded. |
| SRC-013 | [User-supplied Reddit discussion: Alternative A2DP Driver for PC](https://www.reddit.com/r/SonyHeadphones/comments/112zjf9/alternative_a2dp_driver_for_pc/) | Accessible comments reviewed 2026-10-21; classified as REF-001 through REF-023 with proposed prevention and regression coverage in the defect/risk register. Includes failures, success reports, questions and conflicting interpretations. | Anecdotal, mixed products/versions, mostly missing logs/builds; some second-hand/deleted accounts. Reader returned a cached page marked crawled six days earlier. Hidden/deleted comments and recommended external threads were not covered; individual comment permalinks and JSON retrieval failed, so use thread URL plus commenter/subject locators. Relative age labels are not exact timestamps. No complete inventory, current-version defect, causal attribution, malware finding or failure rate is established. |

## Decisions and rationale

These are project decisions, not conclusions guaranteed by the sources.

| ID | Decision | Basis and revisit condition |
| --- | --- | --- |
| DEC-001 | Native Windows 11 x64 first; WPF UI and C++/WDK native components are the proposed stack. | User confirmed Windows 11 x64; SRC-002/003/006 inform the integration path. Specific tool versions and endpoint implementation remain to be validated. |
| DEC-002 | SBC proof before LDAC; every other codec has its own later gate. | User confirmed LDAC priority; prove transport independently from additional encoder complexity. |
| DEC-003 | Behavior-first independent implementation. | User confirmed the approach; binary analysis is optional for a documented interoperability question. |
| DEC-004 | No experimental drivers on the user's primary PC. | User reported that only this PC is available. Lab deployment requires M2 completion and an explicitly authorized target. |
| DEC-005 | No database server, cloud dependency, accounts, or runtime AI agents. | One local device session and local settings do not require them. Revisit only for an approved product requirement. |
| DEC-006 | Evaluate upstream REA before considering a fork. | SRC-009/010/011; fork only for an evidenced missing capability with a bounded patch and test. |
| DEC-007 | Four documentation batches and one bounded package per implementation session. | User requested manageable milestones and accepted this sequencing. |
| DEC-008 | Project license selection is a pre-publication deliverable. | Open-source intent is confirmed, but no particular license or dependency compatibility has been approved. |

## Pending evidence

| Item | State | Next action |
| --- | --- | --- |
| User-supplied Reddit risks/defects | Intake recorded under SRC-013 / REF-001 through REF-023 | Exact builds, logs, independent corroboration and local reproduction remain unavailable. Extend intake for additional supplied evidence. |
| Exact reference-application build and runtime behavior | Unknown | Record only during an authorized behavior study; keep vendor claims separate. |
| Primary PC OS/radio/headphone inventory | Not inspected | M1 read-only inventory; no capability claims in advance. |
| Recoverable test machine and LDAC headset | Not arranged | M2 gate; do not substitute a VM without validated physical-radio access and recovery. |
| Codec/driver dependencies and REA revision | Not selected or pinned | M1 inventories required runtime dependencies; pin optional REA only if an investigation needs it. |
| Protocol specification revisions | Not yet reviewed | M1 records exact A2DP/AVDTP revisions, accessible normative material, and rights for test fixtures before protocol implementation. |
| Driver binding, PCM exchange, HFP/AVRCP coexistence | Unproven | M3/M4 experiments and architecture decision records. |
| Playback, performance, compatibility, and security test results | None | Execute only in the appropriate later package and record exact tested versions. |
