## cloud-tunnels

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**CloudTunnels** — bundle ID `com.fourninecloud.cloud-tunnels`, SwiftPM target `CloudTunnels`. Swift 5.9 / macOS 13+. A native menu-bar app plus a `ctun` CLI for managing port-forwarding tunnels across four providers: **GCP IAP**, **AWS SSM**, **Cloud SQL Auth Proxy**, and **SSH** (with optional kubeconfig auto-patching for private GKE / Loft vCluster workflows). Originally named `GCPIAPTunnel`; `ConfigStore` chains migration from `~/Library/Application Support/GCPIAPTunnel/config.json` → `~/.gcp-iap-tunnels/config.json` so existing users upgrade cleanly.

## Commands

All Makefile targets run from the package root.

```bash
make test            # swift test (XCTest, 365 tests)
make build           # swift build -c release --arch arm64 --arch x86_64
make app             # build + assemble build/CloudTunnels.app bundle
make run             # build + open the .app
make install         # copy the .app to /Applications and strip quarantine
make cli             # build release ctun only
make install-cli     # install ctun to /usr/local/bin
make zip             # build/CloudTunnels.zip (universal)
make zip-arm64       # build/CloudTunnels-arm64.zip  (Apple Silicon only)
make zip-x86_64      # build/CloudTunnels-x86_64.zip (Intel only)
make zip-all         # both single-arch zips
make clean           # swift package clean + rm .build{,-arm64,-x86_64} build
```

Single test or test class via swift directly:
```bash
swift test --filter SSHLauncherTests
swift test --filter SSHLauncherTests/testGcloudIAPUpstreamWrapsSshArgs
```

Debug build (faster iteration, no universal binary):
```bash
swift build                         # debug, native arch only
swift run CloudTunnels               # run the menu-bar app from .build
swift run ctun list                  # run CLI from .build
```

**Known gotcha:** after a failed `make install`, the `.build/apple/CompilationCache.noindex/` directory may end up owned by `root:staff` (XCBuild artifact). `rm -rf .build` will fail with permission errors. Either `sudo rm -rf .build/apple` or build into `.build-universal` via `swift build -c release --arch arm64 --arch x86_64 --build-path .build-universal`.

Logs: `log stream --predicate 'subsystem == "com.fourninecloud.cloud-tunnels"' --info`

Config: `~/Library/Application Support/CloudTunnels/config.json` (legacy read-once paths, checked in order: `~/Library/Application Support/GCPIAPTunnel/config.json`, then `~/.gcp-iap-tunnels/config.json`).

## Architecture

Three SwiftPM targets in `Package.swift`:

- **`TunnelCore`** (library) — provider-agnostic models, launchers, and helpers. Pure, Sendable-friendly. All shared logic lives here so both the GUI app and the CLI can reuse it.
- **`CloudTunnels`** (executable) — SwiftUI menu-bar app. Depends on `TunnelCore`.
- **`ctun`** (executable) — ArgumentParser CLI. Reads the same `config.json` the GUI writes and dispatches tunnels through the same `LauncherFactory`.

### Provider extensibility (critical)

The system is built around a **tagged union `ProviderConfig` enum** paired with a **`TunnelLauncher` protocol**. Adding a new provider means adding a case to both plus one switch branch in each of ~6 places. The pattern has been applied four times: GCP IAP → AWS SSM → Cloud SQL Proxy → SSH. When modifying, follow the existing shape exactly.

**`Sources/TunnelCore/Models/ProviderConfig.swift`** holds:
- `TunnelProvider` enum (`.gcpIAP`, `.awsSSM`, `.cloudSQLProxy`, `.ssh`) — the discriminator used by the UI segmented control, menu-bar tabs, and `LauncherFactory`.
- Per-provider config structs (`GCPIAPConfig`, `AWSSSMConfig`, `CloudSQLProxyConfig`, `SSHConfig`).
- `ProviderConfig` tagged union with a discriminated-union `Codable` impl (`type` + `data` fields) — stable on-disk format.
- Per-case helpers: `kind`, `targetDescription`, `accountTag`.

**`Sources/TunnelCore/TunnelLauncher.swift`** defines the launcher contract:
- `executableURL()` / `executableURL(for:)` — which binary to run. SSH needs the tunnel-aware variant to pick between `/usr/bin/ssh` and `gcloud compute ssh`.
- `arguments(for:)` — argv.
- `listeningMarkers` / `authFailurePatterns` — stderr strings `TunnelProcess` scans for to transition state.
- `fallbackConnectedSeconds` — timer-based "connected" promotion when the launcher stays silent on stderr.
- `environment(for:)` — extra env vars merged into the child process (used by Cloud SQL Proxy for `CLOUDSDK_CORE_ACCOUNT`).
- `LauncherFactory.launcher(for:)` dispatches by `tunnel.provider.kind` — the only place all four cases are enumerated for instantiation.

### Runtime lifecycle

`Tunnel` (struct, `Codable`) → `TunnelManager` (@MainActor `ObservableObject`) → `TunnelProcess` (owns one `Foundation.Process`) → launcher-provided argv.

- `TunnelManager.connect(id:)` creates a `TunnelProcess`, drains its `AsyncStream<TunnelEvent>` on a background `Task`, and translates events into `statuses: [UUID: TunnelStatus]` updates that the SwiftUI views observe.
- Events: `.connecting → .connected → .stopped` on the happy path; `.authExpired` and `.failed` trigger optional auto-reconnect (max 3 attempts × 10s delay, skipped on auth failure).
- `TunnelProcess` kills pre-existing holders of the target local port via `PortUtil.killHolders`, spawns the child with merged launcher env, watches stderr line-by-line for listening/auth markers, and falls back to a timer for quiet launchers.

### Kubeconfig lifecycle (SSH provider only)

When an SSH tunnel with `SSHConfig.kubeconfigPatch` transitions to `.connected`, `TunnelManager.applyKubeconfigPatchIfNeeded` runs `kubectl config set-cluster <cluster> --proxy-url=socks5://127.0.0.1:<socksPort> [--insecure-skip-tls-verify=true]`. On any terminating event (`.stopped`/`.failed`/`.authExpired`), `restoreKubeconfigPatchIfNeeded` unsets `proxy-url` and `insecure-skip-tls-verify` on that cluster. Tracked in `patchedClusters: [UUID: KubeconfigPatch]`. The pure argv builders on `KubeconfigPatcher` (`buildSetClusterArgs` / `buildUnsetArgs`) are unit-tested without shelling out.

**Known limitation:** if the user had a pre-existing `proxy-url` on the same cluster for unrelated reasons, restore wipes it. Nobody hits this in practice.

### Ports

`Tunnel.localPort` is a single Int for compat with the legacy schema, but an SSH tunnel may bind multiple ports (SOCKS + any number of `-L` forwards). `Tunnel.allLocalPorts()` returns the full list per provider; `Tunnel.validate()` uses it for intra-tunnel and inter-tunnel port conflict detection.

`PortUtil.firstFreeLocalPort(startingAt:excluding:)` probes candidates via a direct `bind()` on 127.0.0.1. The Add/Edit form uses it in `.onAppear` (and on kind change) to replace static defaults with a free port — so adding a second Postgres / SSH SOCKS tunnel doesn't silently collide with the first.

### UI layer (CloudTunnels target)

- `CloudTunnelsApp.swift` — app entry, `MenuBarExtra(.window)` host, status icon, IPC observer for `ctun stop`.
- `MenuBarView.swift` — the popover. Five tabs (GCP / AWS / SQL / SSH / Tools). Several `switch provider` blocks (tab bar, `onLoginGroup`, footer, `validity(for:)`) must each be extended when a new provider is added.
- `AddEditTunnelView.swift` — the Add/Edit form. Large per-provider state blocks + ViewBuilder sections (`gcpFields` / `awsFields` / `cloudSQLProxyFields` / `sshFields`). On appear it loads `SSHConfigParser.hosts()` and `KubectlClustersList.fetch()` for SSH dropdowns. Layout is standard SwiftUI with a local `FormField` wrapper.
- `PreferencesView.swift` — app preferences pane.
- `Tools/` — utility suite shown in the "Tools" tab, independent of the tunnel subsystem. ~23 tools across 7 categories (`network`, `encoding`, `identifiers`, `generation`, `tls`, `cloud`, `productivity`) registered through `ToolRegistry.all` in `Tools/Tool.swift` and dispatched via the switch in `Tools/ToolsRootView.swift`. Pattern: pure helper (testable, no SwiftUI) + thin SwiftUI view + tests. Notable entries: SSL Checker (`SSLChecker.swift` — opens TLS handshake via URLSession delegate, parses chain via `CertInspector`), Certificate / CSR Inspector (`CertInspector.swift`, `CSRInspector.swift` — use swift-certificates `X509.Certificate` / `CertificateSigningRequest`), Key Matcher / SSL Converter, K8s Secret Coder, Kubeconfig Inspector, Cluster Health Checker, Cron Parser, Calendar. Tool views must wrap their body in `ScrollView` to prevent the popover footer overlap bug (fix lives in commit `a504838`).
- `Services/CalendarManager` + `UI/UpcomingMeetingBanner` + `Tools/Views/CalendarView` — EventKit-backed next-meeting radar. `CalendarManager` (`@MainActor ObservableObject`) owns `EKEventStore` auth (bridges the macOS 13 `requestAccess(to:)` vs macOS 14+ `requestFullAccessToEvents()` API split), a 5-minute polling loop, and a 60-second reminder tick. Publishes a flat `Upcoming` DTO so views stay EventKit-free. `UpcomingMeetingBanner` renders at the top of `MenuBarView` above the tab bar, visible from every tab. `MeetingAlerter` routes reminders to `ToastManager` when the popover is open vs `Notifications.postMeeting(...)` (with a Join action via `UNNotificationCategory`) when it's closed. `JoinLinkExtractor` pulls Zoom / Meet / Teams / Webex URLs out of event fields. Reminder dedupe is a pure static helper `CalendarManager.dueAlerts(events:now:leadMinutes:alreadyFired:)` — unit-tested without EventKit.
- `Core/AuthManager` and `Core/AWSAuthManager` — track GCP gcloud account list and AWS CLI profile health. Drive the per-group "valid/invalid" badges in the menu bar. Polled periodically.

### CLI (ctun target)

`Sources/ctun/main.swift` defines the ArgumentParser root with subcommands `list`, `start`, `stop`, `status`, `config`, `run-detached`. Commands live in `Sources/ctun/Commands/`. `ctun start <name>` resolves the tunnel from `ConfigStore.shared.load()`, builds a `TunnelProcess` via `LauncherFactory`, and either runs in the foreground (streaming events) or detaches via `RunDetachedCommand`. `ctun stop <name>` posts a `DistributedNotificationCenter` message that the menu-bar app observes (`IPCNotifications`) and uses to cleanly disconnect without triggering auto-reconnect.

### Testing notes

- XCTest (not Swift Testing). Tests live in `Tests/CloudTunnelsTests/`.
- Pure argv builders and config parsers are the primary unit targets — e.g., `SSHLauncherArgumentsTests`, `KubeconfigPatcher.buildSetClusterArgs`, `SSHConfigParser.hosts(at:)`.
- `TunnelProcess` is tested with a fake `TunnelLauncher` that runs `/bin/echo` — see `TunnelProcessTests.swift`.
- No network-dependent tests. No real `kubectl` / `gcloud` / `aws` invocations in unit tests.

## Config file schema quirks

- `ProviderConfig` uses a discriminated `{"type": "...", "data": {...}}` envelope. `type` matches the `TunnelProvider` raw value. Legacy flat GCP schema is still read by `Tunnel.init(from:)` and wrapped as `.gcpIAP(...)` on decode.
- Per-provider configs accept empty strings as `nil` for optional fields (e.g., `account`, `profile`, `region`, `impersonate_service_account`, `kubeconfig_path`). This is load-bearing — the Add form emits empty strings rather than omitting keys.
- `Tunnel.localPort` is **required** in the schema and for SSH tunnels is derived from `socksPort` (preferred) or the first `localForwards` entry's local port. Never the "primary visible port" for the UI — use `Tunnel.allLocalPorts()` or `ProviderConfig.targetDescription`.

## Extension rules — adding a new provider

The tagged-union + `TunnelLauncher` protocol pattern has been applied four times. When adding a 5th provider, touch these sites in order:

1. **`Sources/TunnelCore/Models/ProviderConfig.swift`** — add `TunnelProvider.<name>` enum case, a new `<Name>Config` struct, extend the `ProviderConfig` tagged union case and its `Codable` / `kind` / `targetDescription` / `accountTag` switches.
2. **`Sources/TunnelCore/Models/Tunnel.swift`** — extend `accountKey` + `validate()` switches. If the provider binds multiple local ports, also extend `allLocalPorts()`.
3. **`Sources/TunnelCore/<Name>Launcher.swift`** (new) + optionally `<Name>Locator.swift` — implement `TunnelLauncher`. Override `executableURL(for:)` instead of `executableURL()` if binary selection is tunnel-dependent. Use `environment(for:)` to inject env vars into the child (Cloud SQL Proxy pattern).
4. **`Sources/TunnelCore/TunnelLauncher.swift`** — add `LauncherFactory.launcher(for:)` case.
5. **`Sources/CloudTunnels/Core/TunnelManager.swift`** — only touch this if the provider needs lifecycle side effects (see SSH's kubeconfig patch/restore for the pattern).
6. **`Sources/CloudTunnels/UI/AddEditTunnelView.swift`** — add per-provider `@State`, init-time hydration from `editing?.provider`, form branch in the `switch provider` (line ~166), a `<name>Fields` ViewBuilder, and a `save()` switch case. Load any dropdown data sources in `.onAppear` (see `SSHConfigParser.hosts()` and `KubectlClustersList.fetch()`).
7. **`Sources/CloudTunnels/UI/MenuBarView.swift`** — add the tab button (pick a distinct accent color), and extend three exhaustive switches: `onLoginGroup` closure, footer Add/Login row, and `validity(for:)`.
8. **`Tests/CloudTunnelsTests/<Name>LauncherTests.swift`** — mirror `SSHLauncherTests` as the richest template: argv assertions × config permutations, provider-mismatch throw, auth-pattern detection, `Codable` round-trip, `ProviderConfig` tagged union round-trip, validation edge cases.

If the provider shell-outs to a side-effecting tool (like `kubectl` for the SSH provider), factor the argv builder into a pure `static` function and unit-test it directly without spawning processes.

## Known gotchas

- **Kubeconfig cluster names must be exact.** `kubectl config set-cluster <name>` is an upsert — a name that doesn't match an existing entry creates a stray empty-server entry and leaves the real one unpatched. The SSH form provides a dropdown from `kubectl config get-clusters` specifically to prevent this. When diagnosing "tunnel's up but kubectl doesn't work", check with `kubectl config view --raw -o jsonpath='{.clusters[?(@.cluster.server=="")].name}'`.
- **`.build/apple/CompilationCache.noindex/` may end up root-owned** after some XCBuild multi-arch release runs. `rm -rf .build` will fail partway through. Fix: `sudo rm -rf .build/apple` or build into `.build-universal` via `--build-path`. Self-healing across XCBuildData hash rotations, so don't panic — just work around.
- **`swift-argument-parser` 1.7.1 emits internal deprecation warnings** during compile (`_errorLabel`, `customDeprecated`). Upstream library referencing its own deprecated symbols. Harmless, not worth hiding with `-suppress-warnings` (which would also hide ours).
- **Adding a provider tab to the menu bar pushes the layout.** Current 5-tab layout fits at 420px window width with `.fixedSize()` labels, `Spacer()` between providers and Tools, no label truncation. A 6th tab means revisiting this.
- **Port collisions silently break tunnels.** `PortUtil.firstFreeLocalPort` + `AddEditTunnelView.freeLocalPort(startingAt:)` are wired into `.onAppear` and kind-change for all providers. Don't regress this — the alternative is users hitting "port X already in use" errors at connect time.
- **GKE private-cluster prerequisites go beyond the tunnel itself.** For `kubectl` to work via SSH SOCKS: (a) `gcloud container clusters get-credentials --internal-ip`, (b) the bastion/VM must be on the cluster's **Control plane authorized networks** allow list, (c) kubeconfig `proxy-url` on the exact cluster entry, (d) in-VPC upstream SSH host. The app only handles (c); the other three are operational prerequisites documented in `README.md`.
- **macOS 13 deployment target constrains SwiftUI APIs.** Use the single-arg `.onChange(of:) { _ in ... }` closure form — the two-arg `.onChange(of:initial:)` closure is macOS 14+. Tool views that wrap their body in `ScrollView` must do so to avoid the popover footer overlap bug observed in generation-tools (fix commit `a504838`). When tool content grows without ScrollView, the footer gets painted over.
- **Tool spawning kubectl/gcloud/aws goes through `Tools/Kubectl.swift`.** Do not duplicate the binary-search logic — use `Kubectl.findBinary()`, `Kubectl.run(_:)`, `Kubectl.runSync(_:timeout:)`, `Kubectl.runJSON(_:as:)`. The helper was extracted so every new k8s tool can reuse the same search paths (`/usr/local/bin`, `/opt/homebrew/bin`, `/usr/bin`) and fallback-to-`which` logic.
- **Sandbox is disabled** in `Resources/CloudTunnels.entitlements` so the GUI process can spawn `kubectl`, `gcloud`, `aws`, `openssl`, `caddy`, etc. directly. New tools that shell out don't need entitlement changes.
- **Hardened runtime still requires a privacy entitlement per framework** even with sandbox off. We sign with `--options runtime`, and on macOS 14+ (confirmed on Tahoe 26.x) TCC silently denies access to any privacy-sensitive framework — Calendar, Contacts, Reminders, Photos, Mic, Camera, Location — unless the matching `com.apple.security.personal-information.<thing>` entitlement is sealed into the code signature. The symptom is the "access denied" state with **no permission prompt ever shown**, and no entry under System Settings → Privacy & Security → <thing>. The tccd error log spells it out: `Prompting policy for hardened runtime; service: kTCCServiceCalendar requires entitlement com.apple.security.personal-information.calendars but it is missing`. Fix: add the key to `Resources/CloudTunnels.entitlements` and rebuild. `NSCalendarsUsageDescription` in Info.plist is necessary but not sufficient. Adding a feature that reads Photos / Contacts / etc. will hit the same wall.
- **Ad-hoc signing (`SIGN_IDENTITY=-`) blocks all privacy prompts on macOS Tahoe.** TCC requires a real Team Identifier to key its decisions against. For calendar / contacts / photos testing, rebuild with `SIGN_IDENTITY="Apple Development: <email> (<TEAMID>)"` — list available identities via `security find-identity -v -p codesigning`. The Makefile's sign step quotes `$(SIGN_IDENTITY)` so identities with parens work; `export SIGN_IDENTITY=...` in your shell to avoid passing it every invocation.
- **TCC denials stick to the cdhash, not the bundle id.** Re-signing the app with a new identity doesn't clear a prior denial — run `tccutil reset <Service> com.fourninecloud.cloud-tunnels` before relaunching. For a full wipe use `tccutil reset All com.fourninecloud.cloud-tunnels`.

---
> Source: [FournineCS/cloud-tunnels](https://github.com/FournineCS/cloud-tunnels) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
