# Open A2DP

An independent project to build an open-source Windows 11 x64 Bluetooth Classic A2DP source, with selectable audio codecs and a reliable route back to Windows audio.

**Current status: documentation batch 1 of 4.** No application, driver, codec integration, installer, or hardware proof has been implemented. SBC transport and Windows system audio must be proven before adding LDAC. Other codecs, including aptX variants and LHDC, have separate feasibility and distribution gates.

## Documentation

Start with the [documentation index](docs/README.md).

- [Project requirements](docs/01-project-requirements.md)
- [Implementation milestones and session checkpoint](docs/08-project-implementation.md)
- [System design](docs/15-system-design.md)
- [Security](docs/11-security.md)
- [Sources and evidence](docs/source-evidence-register.md)
- [Defects and risks](docs/defect-risk-register.md)

The next bounded task is **M0-WP02**: architecture, UI/UX, local storage, and API contracts. The roadmap splits the remaining work into achievable sessions.

## Development direction

The proposed stack is C++/WDK for native components and C#/.NET/WPF for the local desktop interface. User-mode encoding, narrow local IPC, bounded data paths, and independent recovery are design requirements. Playback requires no cloud service or runtime AI agents.

Experimental driver installation requires a separate recoverable test machine. Compatibility and codec support will be claimed only for configurations backed by recorded evidence.

## License and affiliation

The intent is to release the project as open source. A project license is pending dependency and codec distribution review; this repository does not grant codec rights or claim support that has not been implemented.

This project is independent of Alternative A2DP Driver, its publisher, REA, and codec vendors. References to those projects and products are research context, not endorsements.
