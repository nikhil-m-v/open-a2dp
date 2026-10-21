# Open-source Windows A2DP codec project

Documentation baseline: 2026-10-21 (Asia/Calcutta). Working project name; no product affiliation is implied.

## Current state

Batch 1 of 4 is authored: requirements, implementation milestones, system design, security, and the supporting registers. These documents specify future behavior. No application, driver, codec integration, installer, hardware experiment, or binary reverse engineering has been implemented or tested.

The user-supplied Reddit thread has been reviewed: 23 reference reports/caveats are classified in the defect/risk register and mapped to prevention, requirements and planned regressions. They remain externally reported, not confirmed project defects.

Confirmed direction: Windows 11 x64 desktop application; SBC transport proof followed by LDAC; behavior-first investigation; one active A2DP output; offline operation. Only the user's primary PC is currently available. Experimental driver deployment waits for a separate recoverable lab.

## Read first

1. [Project requirements](01-project-requirements.md): what success means and which tests will demonstrate it.
2. [Implementation roadmap](08-project-implementation.md): the next bounded task and the gates between milestones.
3. [System design](15-system-design.md): proposed audio path, ownership, lifecycle, and unresolved feasibility questions.
4. [Security](11-security.md): boundaries, installation, recovery, and release requirements.
5. [Evidence register](source-evidence-register.md) and [defect/risk register](defect-risk-register.md): what is known, proposed, and still awaiting evidence.

## Document inventory and batches

Pending filenames below are plain text deliberately: the files do not exist yet. A pending document is not an approved specification.

| Batch | Document | State |
| --- | --- | --- |
| 1 | [01-project-requirements.md](01-project-requirements.md) | Authored |
| 1 | [08-project-implementation.md](08-project-implementation.md) | Authored |
| 1 | [11-security.md](11-security.md) | Authored |
| 1 | [15-system-design.md](15-system-design.md) | Authored |
| 2 | `02-ui-ux-design.md` | Pending |
| 2 | `03-architecture.md` | Pending |
| 2 | `04-database-and-storage.md` | Pending |
| 2 | `05-api-and-contracts.md` | Pending |
| 3 | `06-development-agents.md` | Pending |
| 3 | `07-agent-workflows.md` | Pending |
| 3 | `09-setup-and-initialization.md` | Pending |
| 3 | `10-feature-implementation.md` | Pending |
| 3 | `12-integrations.md` | Pending |
| 4 | `13-qa-and-testing.md` | Pending |
| 4 | `14-debugging-and-logging.md` | Pending |
| 4 | Cross-document consistency and requirement/test review | Pending |

The two agent documents cover development roles and session workflows. They do not introduce AI processes into audio playback. These documents are not a request to launch agents.

## How to continue

The next work package is **M0-WP02: documentation batch 2**. Carry the recorded REF-001 through REF-023 mitigations into architecture, UI/UX, storage and API contracts. Incorporate any additional evidence without treating anecdotes as reproduced failures. Do not start all remaining milestones in the same prompt.

Each session selects one package from the roadmap, records its evidence and validation, and updates the checkpoint there. Later findings update these files in place with the reason and affected requirements. Unresolved driver feasibility must not silently become an implementation claim.

## Authority and terminology

- The user-approved direction and root project instructions govern this work. `sources/` and existing synced files remain read-only.
- `RQ-*` identifies requirements; `TC-*` identifies planned acceptance scenarios; `SEC-*` identifies security controls; `RISK-*` identifies prospective risks; `SRC-*` identifies sources; `DEC-*` identifies decisions; `Q-*` identifies unresolved research questions.
- **Documented externally** means a source states a behavior. **Observed locally** requires a recorded run. **Proposed** means a design choice. **Verified** requires a linked result for a specific build and environment.
- Requirements and tests below are planned. Authoring or reviewing a document does not pass a runtime test.
- The PRD owns product scope, system design owns lifecycle invariants, security owns trust-boundary requirements, and the roadmap owns task status. API layouts, numeric limits, dependency versions, and executable contracts are finalized in their scheduled packages.

## Distribution intent

The project is intended to be open source and usable without subscriptions, activation, or a hosted service. A project license has not been selected or granted by these documents. Dependency compatibility and codec distribution terms must be reviewed before selecting it and publishing code or binaries. Driver signing and test hardware can require funding even when end-user software is free.
