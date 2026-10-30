# Session and contributor workflows

Status: M0-WP03 workflow design, 2026-10-30 (Asia/Calcutta). This describes how to execute future packages and preserve evidence between sessions. It creates no background agent, scheduled task, tool installation or runtime dependency. See [roles](06-development-agents.md), [roadmap](08-project-implementation.md), [setup](09-setup-and-initialization.md) and [security](11-security.md).

## Start or resume one package

1. Read applicable instructions, the latest roadmap checkpoint and the relevant requirement/design documents. Earlier checkpoints are historical evidence; the newest applicable checkpoint controls the next step.
2. Inspect the checkout and current branch before editing. Preserve existing uncommitted changes and distinguish the user's work from this session's changes. Identify the configured remote without exposing credentials.
3. If a previous session stopped at commit/push/review, inspect its real state first. Finish the outstanding authorized step without recreating an already completed commit or assuming a missing tool response means failure.
4. Select the next bounded work package whose prerequisites are met. Name its objective, exclusions, acceptance checks and next handoff. If too large, record named subpackages with their own exit evidence before implementation; do not silently weaken the original exit condition.
5. Separate current-machine work from lab deployment. Documentation is available now; M1 starts after M0 closes. Experimental drivers require the dedicated M2 target, recovery evidence and scoped authorization. No driver or security change is implied by a build request.

Read-only repository checks can use PowerShell from the confirmed repository directory:

```powershell
git status --short --branch
git log -3 --oneline
git diff --stat
git diff --cached --stat
git diff --check
```

An exit code or message must be inspected; several commands in one shell invocation do not make earlier failures successful. Remote checks require connectivity and the normal credential/approval mechanism. Never work around an approval failure with another execution path. Record whether a failed operation actually executed.

## Plan, implement, review, validate

| Stage | Required action | Completion evidence |
| --- | --- | --- |
| Scope | Map the package to its RQ/SEC/REF/TC entries and unresolved questions. Identify files and contract owner. | Bounded task record and prerequisites, with gaps explicit. |
| Inspect | Read the existing implementation/design and exact tool/dependency inputs relevant to the change. | Base revision and provenance; no assumptions that a package or tool is installed. |
| Implement | Make only the selected package's changes; coordinate disjoint contributor assignments. | Reviewable diff and owned artifacts; no edits to synced references. |
| Review | Inspect the combined change for correctness, security, privacy and conflicting assumptions. | Actionable findings resolved or explicitly carried as blockers/limitations. |
| Validate | Execute checks appropriate to the actual change on the permitted environment. | Exact command/tool, version, result and limits; failed/unexecuted checks retained. |
| Checkpoint | Update current package status, evidence/risk records and next package. | A resuming contributor can continue without reconstructing the chat. |
| Commit/publish | Review the exact staged files, commit and push within the user's publishing scope. | Local commit identity and separately confirmed remote result. |

Documentation validation checks links, stable identifiers, fenced examples, traceability and statements of status. It does not run or pass the planned TC scenarios. Code validation must cover actual behavioral boundaries, failures and regressions; do not add tests that only restate the implementation. Hardware acceptance requires the exact authorized lab and independent evidence specified by the roadmap.

## Package record

Use this record in the roadmap checkpoint or a linked package-specific evidence file. A template with empty fields is not completed evidence.

```text
Package / date / responsible contributor:
Objective and explicit exclusions:
Base revision, existing changes, and prerequisites:
Applicable requirements / controls / references / tests:
Permitted environment and target authorization (when relevant):
Files/contracts to change and owners:
Planned checks and expected evidence:
Actual changes and review findings:
Executed checks: command/tool/version, result, evidence location:
Failed or unexecuted checks and reason:
Risks/questions and recovery implications:
Status: complete / partial / waiting, with exit-condition disposition:
Local commit and remote-publication result, if performed:
Next bounded package:
```

Use the user's intended local date/time and include a timezone for dated evidence. Runtime deadlines and ordering use the contract's monotonic timing/generations, not human-readable timestamps. Keep full personal machine identifiers and sensitive evidence in private storage; commit only sanitized records needed for reproducibility.

## Design or experiment decision

When a feasibility question needs an experiment, write an architecture decision record in the owning package. Its proposed location may be `docs/decisions/`; that directory and records are created only when needed. Include question, alternatives, exact experiment, permitted environment, evidence, result, rejected assumptions, consequences and revisit condition.

Q-001 binding ownership is resolved in M3, Q-002 endpoint/PCM exchange in M4, and Q-003 coexistence in M4. Q-004 dependency feasibility and Q-006 toolchain/OS selection have M1 inputs; Q-005 measurement availability needs actual interface evidence. Q-007 reference intake remains distinct from local reproduction. A speculative diagram or copied sample cannot close these questions.

For dependency work, follow [integration review](12-integrations.md): exact immutable source, transitive inputs, provenance/notices, build settings, known limits and permitted fixtures. For generated contributions, inspect the resulting code and its provenance. A tool's successful output is not sufficient evidence of a safe or distributable implementation.

## Regression and defect intake

Retain all REF-001 through REF-023 as reference reports with confidence and attribution limits. Add a confirmed project defect only when there is project-specific evidence; record build, environment, reproduction, expected/actual behavior, affected RQ/TC, severity and safe recovery. Do not turn a forum report into a confirmed root cause.

Route new findings to an owner before the next dependent change:

- Connection/readiness/resume, microphone/profile and stream transitions go to transport/audio with capability freshness and connection/stream generations; use the relevant TC-002/003/009/019/026 procedures.
- Binding drift, install interruption, endpoint loss or unrelated-device disruption go to recovery/compatibility; preserve protected operation identity and immediate/post-reboot evidence under TC-007/011/020/021/022/025.
- Quality changes, incomplete telemetry or unsupported capability claims go to codec/UX; keep manual quality and unknown observations explicit under TC-004/005/006/013/023/026.
- Privacy, unsafe inputs and unreviewed source/dependency material go to security/provenance review under TC-008/017. Offline support and release claims use TC-012/014/024.

Batch 4 expands these into executable procedures and bounded logging policy. All runtime scenarios here are planned and unexecuted. If a reproduction risks primary-machine availability, preserve the report and defer that experiment to the lab rather than trying a destructive workaround.

## Commit and remote verification

Stage an explicit reviewed file list, inspect the staged diff and run whitespace/required checks before committing. Confirm that private investigation artifacts, credentials, raw device identifiers, dumps and synced reference files are absent. Preserve another contributor's unstaged work; do not use destructive cleanup to manufacture a clean tree.

Use the existing authorized branch strategy. New task branches use the repository's configured convention (currently `codex/`) unless the user specifies otherwise. Continuing an explicitly authorized existing `main` workflow does not require creating a branch solely for ceremony. Avoid amending shared commits or force-pushing as a routine response to conflicts.

After an authorized push, compare the remote branch commit with the intended local commit and report whether they match. A local tracking label alone is insufficient if the remote operation failed or was not executed. If the remote moved, inspect the new commits and reconcile normally before retrying; do not overwrite them. A created pull request must be attached to the chat, while a direct push is reported as a direct push. CI results, if applicable, must be reported separately from push success.

## Interrupted or blocked work

| Interruption | Resume behavior |
| --- | --- |
| Editing stopped | Inspect changed files and partial artifacts; validate before treating them as complete. |
| Test stopped or output missing | Record unknown outcome; inspect artifacts/process state and rerun the bounded check when safe. Do not mark pass from silence. |
| Commit completed but push did not | Confirm the saved commit and remote; retry only the outstanding authorized publication step. |
| Push response lost | Query the remote before retrying; it may already contain the commit. |
| Credentials, connection or approval review failed | Preserve the local result, explain the actual failure and request only the needed guidance/access. Do not bypass the mechanism. |
| Lab or dependency gate missing | Mark only the affected package waiting/partial and retain useful safe work. Do not claim the milestone complete. |
| New user steering | Incorporate it into the active objective and record changed scope; preserve completed work unless explicitly canceled. |

The final handoff states what changed, which checks ran, the local/remote commit status, material limitations and the next package. It does not claim runtime support from documentation, count fixture results as hardware evidence, or hide an unexecuted push behind a generic completion message.
