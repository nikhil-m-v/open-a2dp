# Open A2DP

An independent project to build an open-source Windows 11 x64 Bluetooth Classic A2DP source, with selectable audio codecs and a reliable route back to Windows audio.

**Current status: M0 documentation batches 1 through 3 of 4 complete.** Architecture, UI/UX, storage and API contracts now have setup, implementation, integration and contributor handoffs. No application, driver, codec integration, installer, or hardware proof has been implemented. SBC transport and Windows system audio must be proven before adding LDAC. Other codecs, including aptX variants and LHDC, have separate feasibility and distribution gates.

## Documentation

Start with the [documentation index](docs/README.md).

- [Project requirements](docs/01-project-requirements.md)
- [UI and UX design](docs/02-ui-ux-design.md)
- [Architecture](docs/03-architecture.md)
- [Local storage](docs/04-database-and-storage.md)
- [API and contracts](docs/05-api-and-contracts.md)
- [Setup and initialization](docs/09-setup-and-initialization.md)
- [Feature implementation](docs/10-feature-implementation.md)
- [Integrations and dependency review](docs/12-integrations.md)
- [Contributor roles](docs/06-development-agents.md) and [session workflows](docs/07-agent-workflows.md)
- [Implementation milestones and session checkpoint](docs/08-project-implementation.md)
- [System design](docs/15-system-design.md)
- [Security](docs/11-security.md)
- [Sources and evidence](docs/source-evidence-register.md)
- [Defects and risks](docs/defect-risk-register.md)

The next bounded task is **M0-WP04**: QA/testing, debugging/logging, and the final cross-document consistency review. The first milestone, M0, remains in progress until that package is complete. M1 will then establish the safe development foundation. The roadmap splits the remaining work into achievable sessions.

The [defect/risk register](docs/defect-risk-register.md) includes the user-supplied Reddit discussion: 23 externally reported problems, expectations and integration caveats mapped to planned prevention and regression tests. These are unverified reference reports, not confirmed defects in this project.

## Development direction

The proposed stack is C++/WDK for native components and C#/.NET/WPF for the local desktop interface. User-mode encoding, narrow local IPC, bounded data paths, and independent recovery are design requirements. Playback requires no cloud service or runtime AI agents.

Experimental driver installation requires a separate recoverable test machine. Compatibility and codec support will be claimed only for configurations backed by recorded evidence.

## License and affiliation

The intent is to release the project as open source. A project license is pending dependency and codec distribution review; this repository does not grant codec rights or claim support that has not been implemented.

This project is independent of Alternative A2DP Driver, its publisher, REA, and codec vendors. References to those projects and products are research context, not endorsements.
