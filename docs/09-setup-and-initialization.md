# Setup and initialization

Status: proposed M0-WP03 procedure, 2026-10-30 (Asia/Calcutta). The repository currently contains documentation only: there is no solution, application/native project, build script, dependency lock, toolchain pin, or runnable test suite. This package has not inventoried installed build tools, installed software, restored packages, compiled code, or changed a driver. The commands below prepare later work; they are not evidence that setup or a build succeeded.

Read the [roadmap](08-project-implementation.md), [architecture](03-architecture.md), [contracts](05-api-and-contracts.md), and [security boundary](11-security.md) before selecting work. M0 must close before M1 establishes the development foundation. A missing tool blocks its dependent build, not documentation or other independent safe work.

## Development host and hardware target

| Environment | Permitted work and evidence | Excluded assumption |
| --- | --- | --- |
| Current primary PC, as development host | Documentation; M1 read-only tool/OS inventory; reviewed user-mode builds and offline fixture tests once projects exist. | A successful build does not authorize driver deployment, binding changes, Driver Verifier, reboot, or boot/security changes. |
| Separate recoverable Windows 11 x64 lab target | M2 baseline, independent recovery rehearsal and target-scoped authorization precede M3 driver experiments. | The development host is not an automatic fallback target; a VM is insufficient without demonstrated radio access and recovery. |
| Later clean build/CI environment | Recreate pinned tools/dependencies and run the approved user-mode build/test entry points without signing secrets or hardware access. | CI success does not prove receiver support, Windows audio integration, signing eligibility, or recoverability. |

Experimental drivers remain absent from ordinary host setup. Installation/update/recovery helpers are later product components with explicit target validation and protected journals, not developer bootstrap scripts. Offline recovery artifacts and any disk-encryption recovery material stay private and outside the repository.

## Repository orientation

Keep `sources/` and synced project instructions read-only. Review the current checkpoint and Git working state before edits; preserve unrelated local changes. Current documentation may be read without any SDK. Future source/build layout is selected in M1-WP01, with distinct user-mode-only entry points and separate driver/deployment projects.

The existing ignore rules cover `local/`, `private/`, `artifacts/`, `.vs/`, and component `bin/`/`obj/` outputs. Use an ignored local directory for raw inventory and build diagnostics; confirm ignore behavior before writing sensitive output. Do not commit tokens, user paths, machine/device identifiers, dumps, private captures or signing material. Publish a sanitized environment summary, not an unrestricted environment-variable or registry dump.

## M1-WP01: read-only tool inventory

The following PowerShell queries are proposed for the later inventory package and have not been run as part of M0-WP03. Run unelevated in a known shell. First inspect resolved executable locations; use the expected installed binary rather than a same-named file from an untrusted working directory. Missing output means unavailable in that search context, not permission to install it.

```powershell
Get-Command git.exe, dotnet.exe, msbuild.exe, cl.exe -ErrorAction SilentlyContinue |
    Select-Object Name, Source
```

If the resolved .NET host is trusted and present, record its architecture, installed SDKs/runtimes and effective SDK from the intended checkout. Installed runtime and SDK inventories are different; a runtime alone cannot build the project. The invoked host's architecture affects what it lists. [Microsoft .NET CLI reference](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet)

```powershell
dotnet --info
dotnet --list-sdks
dotnet --list-runtimes
dotnet --version
```

Discover installed Visual Studio/Build Tools without launching their installer. The following uses the conventional installed `vswhere` location; if absent, record that fact and inspect an already known installation through a separately reviewed read-only route. Do not download a discovery utility automatically. [Microsoft vswhere examples](https://github.com/microsoft/vswhere/wiki/Examples)

```powershell
$a2dpVsWhere = Join-Path ${env:ProgramFiles(x86)} 'Microsoft Visual Studio\Installer\vswhere.exe'
if (Test-Path -LiteralPath $a2dpVsWhere) {
    & $a2dpVsWhere -all -products '*' -format json
    & $a2dpVsWhere -all -products '*' -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -format json
}
```

Record the actual instance/build, MSVC toolset and compiler version, MSBuild location/version, installed Windows SDK/WDK components, and relevant test tools. A compiler missing from ordinary `PATH` may exist in a developer environment; a folder named Windows Kits alone does not prove WDK integration. Inspect registered component/file versions and kit metadata without repairing them or executing arbitrary project targets. Match the exact SDK/WDK and Visual Studio combination against the then-current official matrix. Microsoft requires matching SDK/WDK build numbers and documents supported Visual Studio combinations. [WDK setup and compatibility](https://learn.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk)

Record each item as **observed**, **missing**, **not inspected**, or **incompatible**, with date, method and sanitized version evidence. A stale prior inventory cannot substitute for the selected validation environment. Any installation need becomes an explicit subsequent setup action with package source/version and scope; do not silently upgrade the host or treat this document as installation authorization.

## Select and pin the toolchain

M1-WP01 owns the version decision. Select supported compatible versions from actual inventory, target OS requirements and dependency constraints; do not copy an arbitrary latest version or the examples in vendor documentation. The selected baseline must identify Windows target build(s), x64 architecture, build configuration, tool versions, acquisition source and any missing components.

| Area | Required selection/pin before implementation |
| --- | --- |
| .NET core, service and future WPF UI | Exact SDK, target framework and desktop targeting requirements; a reviewed `global.json` and explicit SDK roll-forward policy. Keep the safe core/fixture entry point independent of UI and driver deployment. |
| Native user-mode code | Exact compiler/toolset, runtime linkage, Windows SDK, build system and configuration; record compiler/linker options and dependency revisions. Choosing C++ does not automatically choose CMake or a package manager. |
| Later driver build | Compatible Visual Studio/Build Tools, SDK, WDK and extensions/components; record their versions and source integrity. Build support and signing/deployment eligibility are separate decisions. |
| Tests and analyzers | Exact framework, adapter/runner and analyzer versions, fixture provenance, architecture and command entry points. Pin any native test harness too; do not add a tool simply because it is already installed. |
| Dependencies and specifications | Immutable commit/package version, retrieval URL/date, archive/package hash, transitive inventory, license/notices review, selected A2DP/AVDTP revisions and permitted fixture sources. SBC/LDAC candidate review is not codec integration or distribution approval. |

For a reproducible SDK selection, `global.json` can require an exact SDK with `rollForward: "disable"`; set prerelease policy explicitly. Record the actual chosen SDK rather than leaving a placeholder in a committed lock. This is a proposed pinning policy, not a file that exists today. [SDK version selection](https://learn.microsoft.com/en-us/dotnet/core/tools/global-json)

A WPF project requires a Windows target framework and `UseWPF`; confirm the required desktop targeting assets for the chosen SDK before claiming it builds. WPF initialization belongs to its implementation package, not to this documentation session. [Desktop SDK project settings](https://learn.microsoft.com/en-us/dotnet/core/project-sdk/msbuild-props-desktop)

For NuGet application/test entry points, enable and commit the reviewed dependency lock closure, pin direct dependencies without floating versions, and use locked restore during validation. A library's lock file alone does not constrain the application's final dependency graph. Lock changes require review, a renewed provenance/security check and matching offline assets. Native dependencies need their own immutable inventory/lock mechanism. [NuGet lock-file behavior](https://learn.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files#locking-dependencies)

## Future restore/build/test procedure

**Conditional templates only.** No placeholder below names a current repository file. M1 must create and review the chosen projects, pins, source configuration and fixture suite, then replace these templates with exact tested commands. Review imported build tasks, package targets, code generators and post-build hooks: a command named Build can execute code. The approved host entry point must not install a service/driver, invoke a helper, change binding, download unreviewed code or run hardware tests.

Separate dependency acquisition from offline validation. An online acquisition step may download packages, tooling metadata and audit information; record its allowed sources and results explicitly. Prepare a complete verified local package feed/cache for the pinned configuration, including required targeting/runtime/test assets. Use an explicit source configuration so user/machine feeds and credential plugins do not silently fill gaps. `--locked-mode` fixes the dependency graph; it does not make restore offline. [Restore command](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-restore)

```text
dotnet restore <reviewed-user-mode-test-project.csproj> --locked-mode --configfile <reviewed-local-only-NuGet.Config> --packages <verified-local-package-cache>
dotnet build <same-user-mode-test-project.csproj> --configuration Release --no-restore
```

Run each step only if its predecessor succeeds. Use the same SDK, framework, architecture, configuration and package/cache settings across restore/build/test; if a setting affects dependency resolution, include it in the restore too. Missing offline assets fail the run; do not enable remote fallback or discard lock errors to obtain a passing result.

The .NET CLI normally restores implicitly; `--no-restore` suppresses that restore, but is not a network-isolation guarantee. The SDK can also attempt workload advertising downloads, and imported tasks may access the network. Validate the offline claim in an environment with network unavailable and documented CLI/task behavior; do not silently change primary-PC networking to perform that experiment. Keep separate evidence for dependency acquisition, vulnerability review and offline reproducibility. [Build and implicit restore behavior](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-build)

If M1 selects a compatible VSTest-based test project, a possible subsequent command shape is:

```text
dotnet test <same-user-mode-test-project.csproj> --configuration Release --no-build --no-restore
```

The test runner determines supported options. Pin the runner and verify its exact invocation rather than applying VSTest flags to a different runner. `--no-build` requires the matching test binary already built; it does not excuse using an older binary. Record the test binary/build identity, executed test count and result artifact. [Test command and runner selection](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-test)

If M1 instead selects a native MSBuild user-mode project, the build shape is:

```text
& '<pinned-MSBuild.exe>' '<reviewed-user-mode.vcxproj>' /t:Build /p:Configuration=Release /p:Platform=x64
```

Use the pinned compiler environment and explicit project path. This does not select a driver project, restore native dependencies or define the native test runner; those commands must come from the reviewed M1 layout. Inspect the selected project's targets before execution. [MSBuild command reference](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference)

## Initialization and completion evidence

User-mode fixtures start from synthetic data and isolated temporary state, not live machine recovery stores. Do not create the production service, protected installation journal, endpoint or driver binding to satisfy a parser test. Fixture identities cannot authorize a real `targetId` or `controllerLease`; saved settings cannot invent active playback. The [storage](04-database-and-storage.md) and [contract](05-api-and-contracts.md) version/identity rules apply to their eventual implementation.

M1 handoff must record the exact commit, clean or explained working state, sanitized environment/tool versions, lock/inventory identities, commands, exit results, test counts, artifacts and unexecuted checks. A successful protocol fixture run demonstrates only that user-mode behavior on that build. No build or test result is asserted by M0-WP03.

| Acceptance area | Required later evidence |
| --- | --- |
| RQ-014/017; SEC-008; TC-014/017 | Pinned dependency/tool inventory; reviewed provenance; explicit acquisition versus offline build/test results; no hidden network or entitlement dependency. |
| RQ-008/015; SEC-001/002/007; TC-008/015 | Isolated user-mode fixtures, scoped state, sanitized artifacts and no production journal/privileged writes during ordinary setup. |
| RQ-016; TC-016 | Host/lab distinction, M2 recovery rehearsal and explicit target scope before deployment; missing lab never redirects experiments onto the primary PC. |
| RQ-001/010; TC-001/010 | M1-WP03 actual read-only environment inventory and later exact hardware matrix; tool discovery does not claim headset capability or compatibility. |

Next setup owner: M1-WP01 after M0-WP04 completes the documentation baseline. Record remaining tool installation needs and unresolved Q-004/Q-006 rather than claiming that documentation has initialized a working application.
