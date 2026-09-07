## winmint-v1

> Windows 11 ISO builder. Windows-native. Requires `pwsh` 7.6.0+ for backend/runtime scripts (offline staging downloads the latest available 7.6.x+).

# WinMint — Agent context

Windows 11 ISO builder. Windows-native. Requires `pwsh` 7.6.0+ for backend/runtime scripts (offline staging downloads the latest available 7.6.x+).
Development usually happens from WSL or an editor, but all project scripts execute on Windows.

`AGENTS.md` is the compact implementation contract for coding agents. User-facing product behavior, usage examples, and rationale belong in `README.md`.

The core design rule: **UI creates intent. Engine performs work. Reports explain work. FirstLogon finishes live-user setup.**

PowerShell owns the backend and all real product work: profile normalization, ISO/WIM servicing, setup payloads, FirstLogon, reports, release tooling, and validation tooling. The actual build logic must stay headless. The shipped UI is a WebView2 host (`WinMintSetupShell.exe`) plus HTML/JS wizard assets under `assets/runtime/setup/setup-shell/`; it creates intent, previews choices, and invokes the headless PowerShell engine through `tools/ui-bridge/`. It must not own servicing, setup orchestration, offline registry edits, or live-user package installation.

Backend composition uses thin PowerShell modules under `src/runtime/modules/`. Public entrypoints import `WinMint.Bootstrap` (elevation/relaunch), `WinMint.Profile` (profile authoring for the UI bridge), and `WinMint.Engine` (dot-sources `src/runtime/image/WinMint.ps1` as the single canonical runtime load order). Do not add parallel per-area module wrappers unless they have real consumers.

## Product stance (opinionated)

- **User ISO is the truth.** There is no pinned “golden” Windows build inside the repo. Whatever **source ISO** the user picks (subject to documented minimums, e.g. Windows 11 **25H2+** in `README.md`) is the version DISM services. AppX prefixes and registry stamps are **best-effort** against common SKUs; odd OEM bundles may need follow-up outside the wizard.
- **Home-first target.** The primary product target is Windows 11 Home / Home Single Language / en-US. Generated profiles use schema v3, default visible region values of en-US / GeoID 244, and default fixed-edition builds to standard `Windows 11 Home`. Do not silently fall back to Pro when a fixed Home image is missing.
- **Source choice stays simple.** The user-provided official Microsoft ISO is the source of truth. WinMint does not expose UUP Dump selection or conversion as a public product choice. Do not bundle Microsoft payloads or silently download them; require only a high-level consent/automation acknowledgement when network download or conversion is needed.
- **No debloat/performance wizard flags.** Defaults live in engine/profile/setup scripts only—one coherent WinMint posture, not a choice matrix.
- **Subtractive default + opt-in "keep" flags, not granular toggles.** The default build removes full serviceable AI, Xbox/gaming, and the developer tweaks are folded in as baseline. Edge browser noise/AI/promo policy is applied by default. Edge stays installed (`keep.edge` is always true); WinMint does not automate Edge uninstall and does not present a keep/remove Edge choice (`-KeepEdge` is an accepted no-op). Opt-in keep flags suppress one domain each: `-KeepGaming` (keep Xbox/Game Bar AppX and suppress gamebar-policy), `-KeepCopilot` (keep all Copilot+ AI features *except Recall*, which is always removed). Game Mode / HAGS (`gaming-performance-policy`) is baseline for all builds, not gated on `-KeepGaming`. `-DesktopUI` is still an additive shell selection; `-Install windhawk,yasb,komorebi` picks window-manager tooling; editors stay opt-in while WSL2 is always enabled and distro selection remains explicit. WinMint is WSL-first, but Linux distro installs remain explicit. The baseline also uses XDG-style dotfolders for config/data/state/cache, with a temp-backed runtime directory, so XDG-aware tools avoid `AppData` by default. Dev Drive is Off by default; opt in with `target.devDrive` / `-DevDrive Partition|VhdDynamic` and `-DevDriveSizeGb 64|128|256` (default 128 when on). Partition is carved at Setup via diskpart (requires `AutoWipeDisk0` or `DualBootReserved`); VhdDynamic creates an expandable VHDX at FirstLogon. Not for WSL Linux trees. There is no `Developer`/`CopilotPlus` group and no `profileGroups`/`setupOption` vocabulary — the keep flags are the only profile dimension.
- **Profile is the source of truth; the CLI is verb-based.** `WinMint-CLI.ps1` dispatches verbs (`build`/`new`/`validate`/`list`/`clean`; no verb opens the interactive wizard). Configuration flags (`-Edition`, `-Keep*`, `-Install`, `-Dma On|Off`, `-Location On|Off`, `-GenericKey Auto|On|Off`, locales, drivers) live only on `new`, which authors a profile. `build <profile>` and `validate <profile>` consume a profile and accept only run-specific overrides (`-SourceIso`, `-DryRun`, `-WriteUsb -Disk N`, `-Yes`, `-Json`, `-Quiet`, `-AllowElevate`, and the image-quality overrides `-Compression Max|Fast|None` / `-FastImage`). Do not reintroduce flag-built builds or a parallel flat flag block. Image quality is a run override, not a profile field: `-Compression Max` (default) recompresses hard and runs WinSxS `StartComponentCleanup` for a lean release ISO; `Fast`/`None` skip the cleanup for fast test builds; `-FastImage` is the test-quality preset (`None` + no cleanup). The manifest records `servicing.exportCompression` and `servicing.componentCleanup`.
- **DMA interop is default-on, fixed-region, and restore-first.** Unless explicitly disabled with `-Dma Off` / `posture.setup.dmaInterop = false`, Windows Setup uses Ireland / `en-IE` / GeoID `68` internally. Do not expose an EEA country picker. FirstLogon must restore the user-configured visible region, locale, time zone, and location-services posture before OneDrive cleanup, optional agent modules, shell setup, WSL/editors, or live audit.
- **Local accounts require a password.** A pre-created passwordless local account still triggers the Windows 11 24H2/25H2 OOBE "Create a password" page, which blocks an otherwise-unattended install (omitting the `<Password>` element does not suppress it). The build preflight fails a real `-AccountMode Local` build with no password; supply one via `-Password`/`-PasswordPath`/`-PasswordEnvVar`, or use `-AccountMode MicrosoftOobe` for interactive account setup. For fully unattended local-account builds, the OOBE network page is hidden and the profile-computer-name is used directly. Dry runs are exempt (they generate artifacts without installing).
- **Laptop defaults are intentional.** Location services default on; `-Location Off` is the explicit opt-out and should also block Find My Device. Storage Sense safe cleanup and Modern Standby network-off policy are default-on, with Downloads cleanup disabled.
- **Power plans are explicit and non-destructive.** Generated profiles default to `target.powerPlan = Balanced`; `EnergySaver`, `HighPerformance`, and `UltimatePerformance` are explicit profile selections. SetupComplete activates the selected plan on laptops or desktops without deleting other Windows/OEM schemes. Desktop hibernation disable remains form-factor-gated and must not apply to laptops.
- **No maintenance payload.** WinMint must not leave a maintenance scheduled task, background service, or maintenance script behind on the installed system. Post-update drift is the user’s responsibility after installation; maintenance experiments do not belong under shipped runtime/setup folders.
- **Destructive disk behavior is explicit.** Disk modes are `Manual`, `AutoWipeDisk0`, and `DualBootReserved`. Dual-boot mode reserves Windows space using one of `WindowsHeavy`, `Balanced`, `EvenSplit`, or `LinuxHeavy`, and leaves the rest unallocated for another OS.
- **AI removal is serviceable by default.** Public behavior must use supported AppX, optional-feature, registry-policy, service-disable, task-disable, and audit paths. TrustedInstaller ownership tricks, CBS metadata deletion, fake update payloads, `IntegratedServicesRegionPolicySet.json` patches, and maintenance tasks are internal-only research territory and require the `AggressiveExperimental` gate.

## Commands

```powershell
# Validate syntax (PSScriptAnalyzer runs in CI and via -RunAnalyzer locally)
pwsh -NoProfile -File tools\validation\Validate.ps1
pwsh -NoProfile -File tools\validation\Validate.ps1 -RunAnalyzer

# Dual-channel build logs: One Half Dark Spectre via WinMint.ConsoleTheme.ps1 + Console\Logging.ps1;
# full detail always in output\WinMint-Build.verbose.log (mirror: WinMint-Build.log).
# Demo: tools\dev\Show-WinMintBuildLogging.ps1 · assert: tools\dev\Assert-WinMintBuildLogChannels.ps1

# Host contract suite (CI gate): profile invariants, install-plan, agent state,
# CLI matrix, autounattend dry-run (incl. Dev Drive diskpart), VM harness contracts.
pwsh -NoProfile -File tools\dev\Invoke-WinMintPesterContract.ps1
# Legacy entry (profile invariants only):
pwsh -NoProfile -File tests\contract\Test-ProfileInvariants.ps1
# Local install proof: managed Hyper-V smoke (see docs/VM-Acceptance.md).
# Physical: Collect-WinMintHardwareEvidence.ps1 (see docs/Hardware-Acceptance.md).

# Author a profile, then dry-run a build from it (no ISO required to author)
pwsh -NoProfile -File WinMint-CLI.ps1 new BuildProfile.json
pwsh -NoProfile -File WinMint-CLI.ps1 build BuildProfile.json -DryRun

# UI build
pwsh -NoProfile -File WinMint-GUI.ps1

# Release bundle (outputs to dist\)
pwsh -NoProfile -File tools\release\New-WinMintReleaseBundle.ps1 -Version v0.2.0

# VM acceptance — agents/Cursor (detached, pollable; elevated shell required)
pwsh -NoProfile -File tools\vm\Start-WinMintVmAcceptanceManaged.ps1 `
    -ProfilePath .\tests\profiles\hyper-v-smoke-arm64.json
pwsh -NoProfile -File tools\vm\Get-WinMintVmAcceptanceStatus.ps1

# VM acceptance — interactive maintainer (elevated Windows Terminal at repo root)
pwsh -NoProfile -File tools\vm\Invoke-WinMintVmAcceptance.ps1 `
    -ProfilePath .\tests\profiles\hyper-v-smoke-arm64.json

# Provisioning / setup-shell splash preview (-Wizard default; -Native for Direct2D host)
pwsh -NoProfile -File tools\dev\Show-WinMintSplash.ps1
pwsh -NoProfile -File tools\dev\Show-WinMintSplash.ps1 -Native
pwsh -NoProfile -File tools\dev\Show-WinMintSplash.ps1 -Wizard
```

For automation tests: leave the password fields blank (passwordless local
account). The wizard supports passwordless and the Identity preview shows a
"Passwordless" pill.

## Architecture

```
Bootstrap → UI/CLI → Engine → Windows Setup → FirstLogon Agent
```

## VM Test Invariant

Hyper-V VM acceptance profiles are always **Windows 11 Pro** with the Pro generic
key because Enhanced Session testing depends on Pro. Do not repoint
`tests/profiles/hyper-v-install-arm64.json` at a Home-only ISO or change it to
Home. If a pre-updated VM ISO is needed, build a separate pre-updated **Pro** ISO
from Pro-capable source media.

**Smoke vs full:** `tests/profiles/hyper-v-smoke-arm64.json` is the lean plumbing
gate (ISO → Setup → FirstLogon, minimal agent work). `tests/profiles/hyper-v-sl7-smoke-arm64.json`
is the SL7-shaped smoke gate (Israel regional, Cursor + Zen, mocked Fedora WSL,
Phone Link on, Edge kept + debloated). `hyper-v-install-arm64.json` remains the pre-release
release gate (browsers, editors, WSL distros, Nilesoft). Smoke/full use `profileName`
values for harness labeling (`Hyper-V Smoke` / `Hyper-V SL7 Smoke` / `Hyper-V Test`)
and explicit `diagnostics` presets authored by `tools/vm/WinMint-VmAcceptanceProfile.ps1`.
See `docs/VM-Acceptance.md` for the iteration decision tree.

**Smoke WSL is mocked:** Hyper-V smoke profiles (`Hyper-V Smoke` and `Hyper-V SL7 Smoke`)
set `diagnostics.wslRuntimeValidation = skip` so FirstLogon skips real WSL runtime
update/distro work (nested virt is usually unavailable in Hyper-V guests). Contract
tests assert the gate; full `hyper-v-install-arm64.json` still exercises WSL when
nested virt is available. For production-quality ISO size on SL7 smoke, pass
`-FullImage` to the managed/acceptance harness (Max compression + component cleanup).

**Agent/Cursor runs:** follow `.agents/skills/vm-acceptance-orchestration/SKILL.md`
(subagent roster: Run operator → poll → Evidence collector → Root-cause debugger →
Harness implementer → Spec/Quality reviewers). Use `Start-WinMintVmAcceptanceManaged.ps1`
(not raw `Invoke-WinMintVmAcceptance.ps1` from Cursor). Requires an **already-elevated**
shell (no UAC relaunch). Managed start opens **one** Windows Terminal with Spectre
build + harness progress in the same session (pass `-NoLogViewer` for headless).
Default managed runs use **SmartBuild** (ignore `-ForceBuild`
when the image fingerprint is unchanged) and **UseCheckpoint** when a PostSetup
snapshot exists — smoke target is **≤30 min** cached, **2–8 min** with `-PushOnly`
after the first install. Smoke Wait defaults to **90 minutes** (soft time budget
60). Pass `-ForceBuild` only when image/WIM staging changed and you need to bypass
the ISO cache; agents should pass `-TimeoutMinutes 90` explicitly on ForceBuild
examples. Pass `-PushOnly` for FirstLogon/agent/harness iterations when a
checkpoint already exists. Default `-NoObserve`; do not judge AutoLogon via
Enhanced Session password prompts. Poll with `Get-WinMintVmAcceptanceStatus.ps1`
until `complete` is `true`; treat `status=passed` + `verdict=pass` as green. Smoke
verdict also requires `acceptance-result.json` → `setupShell` evidence and
`removalDrift`. Do not trust `stopped` or `running` as final results.
One managed run at a time; `-Force` replaces a stale run.

| Layer | Entry point | Purpose |
|-------|-------------|---------|
| CLI | `WinMint-CLI.ps1` | Headless verb dispatcher (`build`/`new`/`validate`/`list`/`clean`; no verb = help). Verb functions live in `src/runtime/image/Cli.ps1` and delegate to the engine |
| Engine | `src/runtime/image/WinMint.ps1` | Dot-sources all private modules; owns DISM/WIM servicing |
| UI | `WinMint-GUI.ps1`; `apps/setup-shell-web/` | WebView2 wizard host (`WinMintSetupShell.exe --wizard`) plus HTML/JS under `assets/runtime/setup/setup-shell/`. Frontend-only: guided input, previews, validation, and bridge calls into the headless engine via `tools/ui-bridge/`. Rebuild with `tools/release/Build-WinMintSetupShell.ps1` after `apps/setup-shell-web/` or wizard asset edits. |
| UI bridge | `tools/ui-bridge/` | PowerShell scripts invoked by the wizard host for ISO probe, profile generation, and dry-run builds. Must not own DISM, offline registry servicing, Windows Setup orchestration, or first-logon package installs. |
| Agent | `src/runtime/firstlogon/Start-WinMintAgent.ps1` | Runs at first logon; installs editors, WSL distros, and shell layers |
| Setup scripts | `src/runtime/setup/FirstLogon.ps1`, `src/runtime/setup/SetupComplete.ps1` | Machine-phase setup during Windows install |
| Bootstrap | `winmint.ps1` | Downloads release, verifies hash, launches UI |

**SetupComplete Autologon invariant (Local+autoLogon):** Stamp Winlogon to the profile account **before** any unbounded/long SetupComplete network install (`autologon-stamp` before `toolchain-install`); keep a final restamp (`autologon-stamp-final`) before secret cleanup. Never leave `DefaultUserName=defaultuser0` with `AutoAdminLogon` for the first interactive logon — that hangs FirstLogonAnim ("Just a moment") and FirstLogon never starts. Stamp is fail-closed (verify + throw) when Local+autoLogon is selected; SetupComplete still runs secret wipe after action errors.

**FirstLogon runtime load order (setup phase):** `FirstLogon.ps1` → `FirstLogon.Context.ps1` (`Set-WinMintFirstLogonContext`) → `FirstLogon.Support.ps1` (loads `WinMint.Runtime.Common.ps1`, `ProvisioningGuard.ps1`, and split setup modules) → `FirstLogon.Transaction.ps1` → `FirstLogon.Runtime.ps1` (`Invoke-WinMintFirstLogonSetupPhase`). Default `-AgentMode Auto` engages the **provisioning lock** (native `WinMintSetupShell.exe` fullscreen host + `ProvisioningGuard` desktop guard) before pre-agent work and runs the agent headless under it; `-AgentMode Console` / `Headless` skip lock engage/release.

**Provisioning lock invariants (default `-AgentMode Auto`):** Three layers — **transaction** (step order, control phases, agent launch), **ProvisioningGuard** (`NoWinKeys`, `DisableTaskSwitching`, taskbar policy, host lifecycle, idempotent `Stop-WinMintProvisioningHostResidual`), **presenter host** (native AOT `WinMintSetupShell.exe` Direct2D/GDI fullscreen splash reading `setup-shell-status.json`; GDI fallback when Direct2D is unavailable). **First-logon Winlogon Shell** is stamped on DefaultUser to `WinMintLogonShell.cmd` (not `explorer.exe`) so the splash is the first session UI after auth; `WinMintLogonShell.ps1` starts the host immediately, waits for terminal phase (`complete`/`failed`) or a 90‑minute fail-open, and restores `explorer.exe` (reboot phase keeps the custom Shell for autologon resume). `FirstLogon.PreLock.ps1` (autounattend) adopts that host, writes dark theme keys, applies bloom/ImmersiveColorSet under cover, then dismisses Start — wallpaper/SPI must not gate host start. `engage-provisioning-lock` uses `-AdoptIfRunning`; `release-provisioning-lock` / finalize unlock the logon Shell except on `reboot`. Transaction steps: `bootstrap-session` → `engage-provisioning-lock` → `prepare-host` → `persist-retry-autologon` → `restore-visible-user-posture` → `apply-live-user-defaults` → `run-agent` → `finalize-desktop-under-lock` → `release-provisioning-lock` → finalize paths. Pre-agent work runs **under lock**. Control phases are `running` → `finishing` → `complete` (or `failed` / `reboot`); host honors terminal phases and dwell timers (`host=native`, `presenter=gdi-fallback` or Direct2D in `SetupShell.log`). Status projection via `Get-WinMintProvisioningProjection` writes `setup-shell-status.json` with OOBE stages (`stageId` / `taskLabel`: ready → apps → wsl → finish), `detailLabel` + `itemIndex`/`itemTotal` for the current unit of work (brand-stripped install events; never package-manager names or generic `Running *.exe`), stage/item-weighted `progressPct` with `progressMode` `indeterminate` for ready/finish and during long current items, and `elapsedMs` for diagnostics (not painted). The presenter paints main/detail/`i of n`/thin bar (no `%`, no step checklist, no footer meta); hero pulse only while indeterminate; 90s stall and failed/reboot frames show recovery copy + `logDir`. Accessibility: honors `SPI_GETCLIENTAREAANIMATION` (reduced motion freezes pulse/indeterminate travel), high-contrast system colors (flat canvas, no hero bloom), and Narrator via window-title + `EVENT_OBJECT_NAMECHANGE` announcements on status changes. Control carries persisted `preAgentStage`. Rebuild via `tools/release/Build-WinMintSetupShell.ps1` after `apps/setup-shell/` edits. Wizard HTML/JS under `assets/runtime/setup/setup-shell/` is staged on the ISO; native splash assets ship from the same tree. Preview mockup: `tools/dev/setup-shell-splash-mockup/index.html`.

**Agent runtime load order:** `Start-WinMintAgent.ps1` → `WinMint.Runtime.Common.ps1` → `Agent.Context.ps1` (`Set-WinMintAgentContext`) → `Agent.Console.ps1` / `Agent.State.ps1` / `Agent.Host.ps1` / `Agent.Install.ps1` / `Agent.Plan.ps1` / `Agent.Runtime.ps1`.

Setup and agent phases each carry an explicit context object — no ambient `$logDir` / `$payloadDir` mirrors. Setup modules read `(Get-WinMintFirstLogonContext).LogDir` / `.PayloadDir`; agent modules read `(Get-WinMintAgentContext)` for paths, profile, manifest, and console flags. `WinMint.Runtime.Common.ps1` is byte-identical under `src/runtime/setup/` and `src/runtime/firstlogon/` (encoding init, elevation probe, atomic JSON helpers).

`src/runtime/image/WinMint.ps1` dot-sources every private module in order — that is the intentional load pattern. Do not call sub-files directly.

## Separation of Concerns

Strict boundaries. Violations here are architectural bugs.

| Layer | Owns | Must not own |
|-------|------|--------------|
| UI | Guided input, previews, validation messages, profile creation, headless engine invocation | DISM calls, WIM servicing, registry hive edits, setup orchestration, live-user package installs |
| Profile | Defaults, derived settings, schema validation, compatibility checks | Mounting images, installing packages |
| Engine | ISO extraction, WIM servicing, drivers, staged setup files, output ISO | GUI controls, user interaction, live-user app installs |
| Setup scripts | Machine-level setup phases during Windows install | User preference prompts, package source policy |
| FirstLogon Agent | Live-user setup, WSL runtime, editors, shell layers, retry state | Offline image servicing, destructive disk choices |
| Reporting | Manifest, logs, user-readable summaries | Business logic decisions |

## JSON Contracts

Three first-class build contracts plus one diagnostic stream. All business logic passes through the build contracts — never bypass them.

| Contract | Generated by | Consumed by | Lives at |
|----------|-------------|-------------|----------|
| `BuildProfile.json` | UI or CLI | Engine, FirstLogon Agent | `output/<build>/` |
| `BuildManifest.json` | Engine | Reports, human audit | `output/<build>/` |
| `BuildDelta.json` | Audit pipeline | GUI review, CLI summaries, reports | `output/<build>/` |
| `state.json` | FirstLogon Agent | Agent retry logic | `%LOCALAPPDATA%\WinMint\state.json` |
| `WinMintAgent-events.jsonl` (diagnostic) | `Write-AgentEvent` / `-EmitProgressJson` | Future GUI progress, log review | Agent `Logs\` (or demo temp dir) |

**BuildProfile owns:** source ISO path, architecture, device mode (`ThisPC`/`DifferentPC`), edition mode, disk mode/layout, driver source, identity, desktop layers, WSL distros, editors, feature toggles.  
**BuildProfile does not own:** build timestamps, WIM mount paths, download hashes, step success/failure state, UI display strings.

**BuildProfile schema v4 (breaking):** `schemaVersion` must be `4`; v3 profiles are rejected. Subtractive intent is `keep` only (`edge`, `gaming`, `copilot`); AppX removal prefixes are derived from `keep`, not authored. `removals` carries `aiPolicy` only. User-facing posture lives under `posture` (appearance, explorer, accessibility, setup); privacy under `privacy` with state enums (`enabled`/`disabled`). `development.wsl` is `{ distros: [] }` — WSL enablement is implicit when distros are listed. Map helpers: `Resolve-WinMintBuildTweaksFromProfile`, `Resolve-WinMintBuildPrivacyFromProfile`, `Get-WinMintProfileAppxRemovalPrefixFromKeep`. Migrate v3 with `tools/dev/Convert-WinMintBuildProfileV3ToV4.ps1`.

**BuildDelta owns:** generated backend truth for "what WinMint changes" after profile normalization and install-plan derivation. Every record must include phase, contributor source, applicability/suppression metadata, and a concise change list. Do not maintain a separate handwritten authoritative backend behavior summary once a behavior is represented in `BuildDelta`.

Driver sources are conservative by design. `None`, `Host`, and `Custom` remain compatibility values, but Surface driver packs should use either `SurfaceCatalog` or `SurfaceMsiSafe`. `SurfaceCatalog` means `profile.drivers.path` is an exact WinMint Surface catalog device id from `config/surface-drivers.json`; it resolves the official Microsoft Download Center package at build time, downloads it into the temp work directory, verifies Microsoft ownership/signature evidence, then runs the same safe Surface classification path. `SurfaceMsiSafe` is the manual official Surface MSI path. Both paths extract the MSI, require the `SurfaceUpdate` payload, exclude firmware-class drivers from offline injection, and write `WinMint-DriverInventory.json`. The catalog must use Microsoft-owned URLs only; third-party catalogs such as SurfaceTip are research references only and must not be runtime download sources. Do not route Surface recovery-critical installs through raw recursive `Custom` injection unless deliberately investigating an experimental failure.

**Agent step status values:** `pending`, `running`, `ok`, `failed`, `skipped`, `retryable`, `needsReboot`.

Schemas:

- `schemas/winmint.buildprofile.schema.json`
- `schemas/winmint.buildmanifest.schema.json`
- `schemas/winmint.agentstate.schema.json`
- `schemas/winmint.agentevents.schema.json` (JSONL line shape for agent progress events; diagnostic-only)

Validate with `tests/contract/Test-ProfileInvariants.ps1`.

## Key Files

| File | Purpose |
|------|---------|
| `config/packages.json` | winget / Store / Scoop package catalog |
| `src/runtime/image/Private/Image/Tweaks/` | Registry tweak modules — one `NN-<id>.ps1` per tweak, each carrying its definition (`set`/`remove`/metadata) **and** its `appliesTo` curation predicate. `TweakRegistry.ps1` validates each definition, derives the typed `operations.registry` DOM, and exposes `Get-WinMintSelectedRegistryTweaks`. `Tweaks.ps1` executes the normalized operations through guarded helpers, not loose ad hoc delete/set loops. Add/change a tweak by editing one file there. |
| `config/tweaks.json` | Public metadata mirror of the tweak modules; kept in parity by a contract test (`StaticAssertions.ps1`). Not the executable source. |
| `schemas/winmint.*.schema.json` | JSON Schema for the profile, manifest, and agent-state contracts |
| `config/autounattend.xml` | Windows unattended install template; generated output must ship alongside ISO |
| `assets/runtime/desktop/windhawk/preset.manifest.json` | Windhawk mod preset manifest (installed as a unit, not individual mods) |
| `assets/runtime/desktop/yasb/preset.manifest.json` | YASB preset manifest |
| `assets/runtime/desktop/komorebi/`, `assets/runtime/desktop/yasb/` | Shell layer configs |
| `PSScriptAnalyzerSettings.psd1` | Linter settings |

`output/` and `dist/` are build artifacts — gitignored, do not edit.

## Desktop Shell Model

Layers are **additive and composable**, not mutually exclusive.

| Layer | What it adds |
|-------|-------------|
| Standard Windows | Clean WinMint baseline; no extra shell |
| Windhawk | Shell polish, dock/taskbar styling |
| YASB | Top bar / status surface |
| thide | Taskbar hiding for launcher-centered workflows |
| Komorebi | Tiling window manager |
| Nilesoft Shell | Context-menu polish |

Standard Windows means zero added layers. Windhawk, YASB, thide, Komorebi, and Nilesoft can be freely combined unless a future contract says otherwise. The build/agent must install, configure, and start each selected layer so the desktop matches any UI preview.

## Agent Module Map

| Feature | Module path |
|---------|-------------|
| Package managers | `src/runtime/firstlogon/Modules/PackageManagers.ps1` |
| Editor bootstrap | `src/runtime/firstlogon/Modules/Editors.ps1` |
| Git config | `src/runtime/firstlogon/Modules/Git.ps1` | **Deferred:** no `development.git` profile block yet; MinGit via PackageManagers is the baseline Git provider. Module stays disabled (`modules.git.enabled = false`). |
| Dotfiles | `src/runtime/firstlogon/Modules/Dotfiles.ps1` | Opt-in via `development.dotfiles` (`repository`, optional `ref`, `installScript`). HTTPS public repos only in v1; clones with MinGit and runs install script when set. |
| WSL2 bootstrap | `src/runtime/firstlogon/Modules/Wsl.ps1` |
| Launcher key binding | `src/runtime/firstlogon/Modules/LauncherKey.ps1` |
| Shell layers (yasb/thide/komorebi/nilesoft) | `src/runtime/firstlogon/Modules/TilingDesktop.ps1` |
| Windhawk | `src/runtime/firstlogon/Modules/Windhawk.ps1` |
| Profile composition | `src/runtime/firstlogon/Modules/Profiles.ps1` |

Launcher selection is currently `None` only (`features.launcher`). The launcher key module always records the common Copilot hardware-key chord, `Win+Shift+F23`, and clears Copilot app key policy so Windows can use the native Search target/fallback. Windows Search and indexing stay on for Start/Settings integrations. Keep optional tray icons hidden unless the icon exposes a real, user-facing status or control surface.

Phone Link policy and live install audit are also opt-in live-user modules. They must stay disabled unless `features.phoneLink` / `features.liveInstallAudit` or the matching CLI flags are explicitly selected. When Phone Link is off, the release image also removes `Microsoft.YourPhone` and `MicrosoftWindows.CrossDevice` provisioned AppX; opt-in keeps them on the WIM. Live install audit is diagnostic and non-blocking: it must write a report and return warning/error counts without failing FirstLogon.

Each module must be idempotent. A failed optional layer must not break the entire first logon. Every step writes `state.json` before and after it runs.

## Package Source Policy

The user does not choose package sources. WinMint decides.

| Source | Used for |
|--------|----------|
| winget | GUI apps, Microsoft apps, signed installers, shell integrations, desktop services, and packages where the upstream installer is canonical. Baseline also installs Coreutils for Windows (`Microsoft.Coreutils`) as native UNIX-style host CLI. |
| Scoop | User-local developer CLI tools and toolchain plumbing. Scoop is installed during FirstLogon with the official installer; MinGit is the baseline Windows-host Git provider; Starship is the baseline Windows-host prompt with the `nerd-font-symbols` preset; selected Neovim is Scoop-owned. |
| GitHub release | Reserved for future upstream-asset-backed tools when winget metadata lags or a specific release asset/architecture is needed |
| Store source | Store-backed packages where the upstream app is distributed through Microsoft Store and winget surfaces them via `msstore` |
| Direct download | Reserved for narrow pinned exceptions when winget/Scoop cannot supply a required native asset |

For an ARM64/aarch64 source ISO, FirstLogon must aggressively prefer native ARM64 package assets in both winget and Scoop where the package-manager metadata supports them. For an amd64/x64 ISO, do not force architecture flags; use the package manager's default selection.

## Debloat Tiers

**Tier 0 — Never touch:** Windows Update, Defender, SmartScreen, Firewall, Store infrastructure, Desktop App Installer, winget, WebView2/Edge runtime, WSL, Virtual Machine Platform, Hyper-V networking, IPv6, WinRE, WinSxS component store, UAC.

**Tier 1 — Apply by default (WinMint Core):** DMA-aware Microsoft AppX removal (Clipchamp, Xbox unless `-KeepGaming`, Solitaire, Teams consumer, Recall, Copilot unless `-KeepCopilot`, Dev Home, WebExperience, Calculator, Quick Assist, Sound Recorder, Sticky Notes, Maps, To Do, OneNote, Remote Desktop Store client, legacy media apps, etc.), advertising/content surfaces, Edge noise debloat (first-run/startup boost/promos/workspaces/Spotlight/import nags/address-bar trending/inline compose/web AI APIs while preserving explicit Edge Copilot page-context chat; Edge browser stays installed), OneDrive removal, GameDVR plus no-op Game Bar protocol handlers when Xbox/Game Bar is removed, Home-safe privacy policy, Storage Sense safe mode, Modern Standby network-off, Delivery Optimization peer-to-peer off with Windows Update preserved, vendor driver co-installers blocked and network device-metadata / companion-app prompts blocked (`PreventDeviceMetadataFromNetwork`) with Windows Update driver delivery preserved, WPBT disable, and Explorer dev-QoL defaults (show extensions, hidden files, keep Home, full path in title bar, quiet Quick Access, Git FE version-control toggle, hide Gallery, long paths, End Task on the taskbar, quiet taskbar/tray affordances, local clipboard history on, cloud clipboard upload off). Small reinstallable inbox apps are not protected platform components; users can reinstall them from Store/App Installer after setup. Default also applies a narrower serviceable AI policy: Recall is always removed; imposed Copilot app/shell surfaces, Notepad AI, web AI APIs, and app access to system/generative AI models are disabled unless `-KeepCopilot` is selected. Click to Do, Paint AI, Edge Copilot page-context chat, the local Settings agent, Office AI, agent connectors, and workspaces are not touched by the default AI policy. WebView2 / Edge runtime infrastructure is never removed. The developer QoL tweaks (Developer Mode, PS RemoteSigned, .NET/PS telemetry opt-out, elevated terminal, OpenSSH, Scoop, MinGit, Coreutils for Windows, Starship with `nerd-font-symbols`) are baseline. Windows Terminal defaults to PowerShell 7, Cascadia Code NF, One Half Dark, no audible bell, and centered launch. Archive/extraction stays native; do not install a third-party archive manager by default. After successful FirstLogon cleanup, create a final `WinMint post-install complete` System Restore point.

Broad third-party and OEM AppX prefixes should stay candidate-only by default. DMA setup reduces the expected default-app/promotional payload, so do not expand normal removal into a broad OEM cleanup list unless a specific source ISO proves it is needed and the catalog/test contract is updated deliberately.

**Tier 3 — Reject:** Disabling Defender/Firewall/SmartScreen, disabling Windows Update or WaaSMedic, removing WinSxS, removing WebView2, disabling IPv6, disabling Hyper-V/HNS networking services, hosts-file endpoint blocks, blanket scheduled task disabling, CPU security mitigation disables, "Ultimate Performance" mode by default.

See `docs/Windows-Debloat-Strategy.md` for the full audit and Tier 2 candidates.

## Architecture Invariants

The backend architecture refactor is complete and merged. These four properties
are now established invariants — preserve them, do not regress them:

1. UI saves a complete `BuildProfile.json` before starting
2. Engine builds from a profile without GUI code loaded
3. Manifest explains the build without scraping logs
4. FirstLogon resumes after interruption via `%LOCALAPPDATA%\WinMint\state.json`

`docs/Project-Structure.md` is the repository layout contract. Keep the app
runnable after every change; do not rewrite working subsystems wholesale.

## PSScriptAnalyzer

Run: `Invoke-ScriptAnalyzer -Path . -Settings PSScriptAnalyzerSettings.psd1`

Intentional exclusions — do not "fix" these:

| Rule | Why it is excluded |
|------|--------------------|
| `PSUseShouldProcessForStateChangingFunctions` | UI/build helpers don't need -WhatIf |
| `PSReviewUnusedParameter` | DryRun param used indirectly via CmdletBinding |
| `PSAvoidUsingInvokeExpression` | Dot-sourcing internal blocks is the load pattern |
| `PSAvoidUsingWriteHost` | Script entry points and validation helpers intentionally report directly to the console |

## Architecture Detection

Parsed from ISO filename via case-insensitive regex. Cross-checked against WIM metadata and `setup.exe` PE header — all three must agree or the build aborts with no side effects. If the filename has no arch marker, the script prompts once at runtime.

## Distribution

Short launch path: `irm https://winmint.yanai.sh | iex`
Default bootstrap behavior is ephemeral: download the release zip and `.sha256`
to a unique `%TEMP%` session, verify SHA256, extract and run from that session,
wait for the launched process, then best-effort remove the session. Do not
reintroduce a default `%LOCALAPPDATA%\WinMint\versions` install/cache path.
Durable release cache behavior is explicit opt-in only through launcher switches
such as `-InstallRoot` or `-CacheRelease`.
Bootstrap failures must go through the friendly failure envelope in `winmint.ps1`
with operation, failure kind, reason, recovery guidance, and retry-safety text.
Preserve the explicit failure kinds: network, integrity, package, runtime,
elevation, relaunch, usage, and unexpected.
Cloudflare Worker source: `cloudflare/winmint/` — deploy with `bunx wrangler@latest deploy --config wrangler.jsonc`

Release bundle: `tools/release/New-WinMintReleaseBundle.ps1` → produces `dist/WinMint-<version>.zip` + `.sha256`. Upload both to the matching GitHub release.
CI/CD policy: normal pushes and PRs validate only. Releases are tag/manual driven through `.github/workflows/release.yml`; push a `v*` tag or run the workflow manually to build and upload the zip plus hash assets. Do not publish on every push.

## Decision records

Architecture and product decisions live in [`docs/decisions/`](docs/decisions/). Read [DECISIONS.md](docs/decisions/DECISIONS.md) for the audit matrix; add an ADR before reversing an Accepted decision.

## Documentation Ownership

- Update `README.md` when user-facing behavior, setup commands, requirements, or rationale changes.
- Update `AGENTS.md` when architecture boundaries, coding constraints, repo contracts, or agent workflow rules change.
- Update `docs/decisions/` when stack, contract, or acceptance strategy changes.
- Treat `docs/codebase/` as current-development snapshots for onboarding and audits, not as a continuous authoritative source of truth.
- Update schemas and contract tests together whenever `BuildProfile.json`, `BuildManifest.json`, or `state.json` shape changes.

## Commit Style

Conventional commits: `feat(scope):`, `fix(scope):`, `refactor:`, `docs:`, etc.  
Scope = component name: `windhawk`, `agent`, `shell`, `firstlogon`, `engine`, `profile`, `ui`, `komorebi`, `yasb`.

---
> Source: [yanai-sh/winmint_v1](https://github.com/yanai-sh/winmint_v1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
