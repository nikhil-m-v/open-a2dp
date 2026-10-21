# Security

Status: batch 1 security design, 2026-10-21. No controls have been implemented or verified. These requirements govern future driver, service, installer, logging, and release packages. See [requirements](01-project-requirements.md) and the [risk register](defect-risk-register.md).

## Current-machine boundary

Only the user's primary PC is available. This session and the pre-lab packages permit documentation, offline/user-mode builds and tests, and read-only inspection. They do not authorize installing an experimental driver, changing device binding, enabling Driver Verifier, altering boot/signing/Secure Boot/Memory Integrity settings, replacing the inbox stack, or rebooting for tests.

M2 must establish a separate recoverable target, baseline, out-of-band recovery path, and recovery rehearsal before deployment. A deployment task must identify the target and intended changes and have explicit authorization for that scope. Existing authorization covers its necessary bounded steps; do not repeatedly request permission for steps already included. If the test target is not available, continue independent safe tasks rather than moving the experiment onto the primary PC.

## Assets, actors, and trust boundaries

Protect kernel integrity, system bootability, working inbox Bluetooth/audio, paired-device state, audio confidentiality, per-user settings, diagnostic artifacts, update trust, and signing keys.

| Entry / actor | Treat as | Main concern |
| --- | --- | --- |
| Ordinary local process, another user/session, remote pipe caller | Unauthorized until OS identity/ownership checks succeed | Privilege escalation, cross-user audio/control, denial of service. |
| Desktop UI and configuration files | Fallible user input, never authoritative for active driver state | Invalid settings, path injection, replay/stale commands. |
| Encoder worker and codec library | Least-privilege component with native-code attack surface | Worker compromise, malformed frames, excessive CPU/memory. |
| Headphones, Bluetooth traffic, vendor driver responses | Untrusted external data even from paired devices | Malformed capabilities, invalid state transitions, resource exhaustion. |
| User-to-kernel and driver-to-driver requests | Independently validated at each boundary | Length/overflow bugs, race conditions, use-after-free, data leakage. |
| Installer packages, recovery journals, update manifests | Integrity-sensitive inputs | Arbitrary privileged file/registry/device operations, malicious rollback. |
| REA, analysis providers, investigated binaries | Development-only tools/artifacts requiring provenance and containment | Target execution, tool-generated mistakes, proprietary data disclosure. |

A fully compromised administrator or kernel is not a boundary this application can enforce. Even administrator-initiated operations remain narrowly scoped to reduce accidental damage. Pairing alone does not make remote inputs memory-safe.

## Security controls and required validation

### SEC-001 — Privilege separation and session ownership

Run the UI and codec worker without elevation. Use a dedicated restricted service identity where access permits, granting only needed device/service rights. M3 records the tested identity and ACLs before expanding privilege. If additional privilege is essential, record the exact reason and independent review; do not default the entire application to LocalSystem or administrator.

Use a short-lived elevated helper for installation, binding changes, and restoration. The helper exposes fixed operations on validated targets/packages, not arbitrary commands, scripts, paths, or registry writes. It is usable independently of the normal service. Do not store administrator credentials or keep an unrestricted elevated channel open after completion.

The service derives SID/session identity from Windows authentication, not a client-provided user field. One interactive controller owns one selected-device stream. Reject cross-user mutations, remote sessions outside the documented local policy, and attempts to adopt stale handles. Ownership release/logoff stops the old stream before a new owner can acquire it. A driver media handle is scoped to its stream and does not confer installation/control rights.

Validation: standard-user playback; denied unrelated-user/remote access; session switch/logoff; restricted-worker token/handle audit; helper operation allowlist; no broad privileged process launch.

### SEC-002 — Local IPC authorization

Control uses versioned local named pipes with explicit DACLs for the service and authorized OS identities, remote clients rejected, and peer identity checked. Protect against pipe impersonation/name squatting by validating the server identity and using service-owned endpoints; clients cannot define the service endpoint or authentication policy.

Validate message framing, maximum sizes, enum/range/format fields, request IDs, connection generation, and operation ownership before dispatch. Apply bounded per-client concurrency, rate limits, timeouts, and cancellation. Use OS authentication rather than a reusable secret in JSON. Disconnect stalled clients without blocking the audio path. No default TCP/HTTP listener or remotely hosted control code.

Batch 2 must define major/minor compatibility and per-operation authorization before implementation. A newer incompatible control client must not send unrecognized mutation messages. Recovery uses its independent compatibility path, not a broken service handshake.

Validation: malformed/truncated/oversized/replayed messages; unauthorized callers; fake server; concurrent clients; stalled client; version mismatch; late responses after generation changes.

### SEC-003 — Driver interface and kernel memory safety

Create explicit device-interface ACLs; deny general public mutation access. Expose narrow IOCTLs with required access bits and state validation. Prefer buffered I/O for small control structures. Do not introduce METHOD_NEITHER or caller-supplied kernel/physical pointers as a shortcut. Any future direct-I/O or mapped-buffer path needs an ownership/lifetime design and independent review first.

Check input/output lengths, structure version, alignment, signedness, integer additions/multiplications, frame counts, offsets, identifiers, and legal state. Reject unknown operations and invalid reserved fields. Zero/initialize all returned bytes; never leak padding, pointers, kernel addresses, or stale buffer contents. Avoid unsafe cross-bitness assumptions even though the first product is x64.

Validate actual runtime limits against negotiated MTU/configuration as well as static maximums. Do not trust an authenticated service, an encoder-produced frame, or a lower-level successful status as proof that supplied lengths and state are valid.

Use bounded nonblocking callbacks, explicit IRQL-safe behavior, cancel-safe ownership, and reference-counted lifetimes. PnP removal, close, timeout, and completion races must complete each request at most once. Do not free a buffer or device context until all potential users are quiescent. Failing safely means stopped playback with a reason, not continuing with corrupt state.

Validation: static analysis and compiler mitigations; user-mode parser fuzzing; boundary/overflow fixtures; I/O cancellation/removal races; focused driver verification on the lab; uninitialized output inspection; independent kernel-interface review.

### SEC-004 — Codec and protocol isolation

Use reviewed build-time selected codec adapters; do not load arbitrary DLLs from user preferences, current directories, downloads, or untrusted plugin paths. Keep encoding/parsing outside kernel where feasible. The worker cannot update drivers or load kernel modules. Restrict inherited handles and filesystem/network access to what the tested design requires.

Validate remote AVDTP messages, capabilities, transactions, frame/packet sizes, and state transitions against pinned specifications. Limit all queues, outstanding operations, parser nesting/iterations, retries, and transaction duration. Treat unknown vendor-specific data conservatively. Fuzz shared parsers in user mode before exposing them through the driver.

Validation: malformed remote fixture corpus; failed negotiation; output exceeding MTU; codec worker crash/hang; repeated connection attempts; unsupported/vendor codec data. A disabled codec remains unselectable and cannot be loaded by a client override.

### SEC-005 — Resource ownership and failure containment

Assign every buffer, channel, process, handle, timer, and outstanding request a single documented owner and release path. Associate work with a connection generation. Bound memory, handle counts, retry loops, and CPU work; numeric budgets are approved in the owning implementation package before code lands.

Separate control and data paths. Never wait for UI, disk logging, or a codec computation in a real-time kernel path. Worker/service death must stop orphaned streams and clean up pending work. Cleanup timeout must not lead to freeing still-referenced memory or reusing a live channel.

Validation: saturation/backpressure, sleep/resume, remove/close races, worker/service kill, leaked-resource observation across reconnect cycles, and delayed callback injection.

### SEC-006 — Install, update, and independent recovery

Before privileged changes, verify package identity/integrity/signature appropriate to the environment and resolve the exact target device from OS state. Do not trust a display name or arbitrary client path. Verify files in a protected staging location to avoid check/use races or user-replaceable binaries.

Record pre-change binding/package identifiers and an appendable operation journal in machine-protected storage. Use allowlisted operations, narrow INF matching, idempotent checks, and explicit handling of interrupted steps. Do not delete unrelated driver-store packages, pairing data, or vendor components. Prefer restoring the observed previous binding rather than assuming every machine has the same previous state.

The independent helper reads validated protected state and checks live OS state before acting. Keep recovery instructions and artifacts available offline. Recovery tests include service/UI unavailable, reboot if actually required on the lab, and confirmation of inbox playback afterward. A system restore point or backup claim is not a substitute for a tested binding/recovery path.

Initial releases use a manually initiated verified package/update flow. Automatic updates are deferred. A signed but incompatible old build must not silently downgrade a newer driver/service; intentional rollback uses an explicitly compatible pair and revalidation. The package must never silently change boot/security settings.

Validation: wrong target/package, denied elevation, altered staged files, truncated journal, process/power interruption, repeated operation, version mismatch, upgrade/rollback, and clean restoration.

### SEC-007 — Logs, audio privacy, and storage

Normal logs contain structured state/errors and bounded counters, not PCM, encoded audio, Bluetooth link keys, credentials, or unrestricted packet captures. Treat device names/addresses, user paths, and crash dumps as sensitive. Restrict local file access by SID/service role and separate per-user preferences from privileged machine installation state.

Before logging implementation, define hard byte caps, retention periods, opt-in diagnostic duration, and cleanup behavior. High-rate callbacks use bounded event buffers and may drop diagnostics rather than block playback or fill disk. Document dropped-event counts without recording private content.

Support export is user initiated, locally generated, redacted, and inspectable. Address masking uses an export-scoped alias, not a persistent globally correlatable identifier. Do not automatically bundle full dumps/captures or upload anything. Optional raw diagnostics need a specific purpose, explicit consent, duration limit, private destination, and separate deletion policy.

Settings migrations and recovery-state parsing use the same bounds/path discipline as IPC. Prevent link/reparse/path traversal and permission inheritance from creating a privileged arbitrary-write primitive.

Validation: malformed/newer settings, interrupted writes, ACLs, log rotation/full disk, export search for seeded identifiers, opt-in expiry, and no network dependency.

### SEC-008 — Build, signing, and release trust

Pin reviewed dependencies/tools by immutable revision and checksum where applicable; record source, license, notices, build settings, and dependency inventory. Build with appropriate warnings, static analysis, and supported exploit mitigations. Compare artifacts from a documented clean environment; explain unavoidable signing/timestamp differences rather than claiming bit-identical signed output without evidence.

Signing keys and submission credentials stay outside the repository and untrusted builds. Separate unsigned CI validation from a controlled signing/release step. Validate current Microsoft submission policy and applicable certification tests before public distribution. Do not promise that a self-signed or merely Authenticode-signed package will load on all target systems. [Driver signing policy](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/kernel-mode-code-signing-policy--windows-vista-and-later-), [signing requirements](https://learn.microsoft.com/en-us/windows-hardware/drivers/dashboard/code-signing-reqs)

Release qualification requires the selected supported Windows builds with Secure Boot and Memory Integrity enabled, standard-user playback, valid package integrity, tested recovery, complete notices, and no unresolved critical/high security findings. A development test-signing path is lab-only and never an end-user prerequisite.

Validation: dependency/provenance audit, clean build, signing-key exposure checks, signature verification, applicable driver verification/HLK results, clean-target signed install, offline operation, and rollback evidence.

### SEC-009 — Research provenance and tool execution

Start with specifications/public documentation and recorded behavior. For targeted binary analysis, record the unresolved interoperability question, lawful source/access context, exact target version/hash, tools, observations, and limitations. Do not assume a tool's license gives rights to its analysis target. Do not copy proprietary implementation material or bypass activation, DRM, signing, or kernel protections.

REA is optional development tooling, with provider-specific capabilities to validate. Pin and test it on a known permitted fixture first; prefer upstream, fork only for a reproduced missing capability and bounded testable patch. Never give an analysis tool or investigated executable elevated access on the primary PC. Static analysis and execution have different risks; inspect intended actions before running target code.

Validation: source/observation lineage, separation of restricted artifacts, generated-code review, tool revision and fixture result, and independent corroboration for architectural conclusions.

## Gates and incident response

Before the first driver deployment: M2 recovery evidence, reviewed SEC-001/002/003/006 contracts, validated exact target and scoped authorization. Before alpha: device removal and independent recovery tested. Before beta: adversarial and lifecycle coverage plus independent review. Before public release: SEC-008 qualification and all applicable critical/high issues closed.

For suspected corruption or compromise, stop reproduction attempts, preserve the minimum private evidence and exact build identity, use the independent recovery path, and investigate on the lab. A suspected security defect blocks affected releases until triage. Before M8, establish a private maintainer contact and disclosure procedure; the contact is not yet created. Never publish an unredacted dump in a public issue as the default debugging step.
