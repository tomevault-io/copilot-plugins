## fw-fanctrl-gui

> Guidance for AI agents and contributors working in this repository.

# AGENTS.md

Guidance for AI agents and contributors working in this repository.

## What this is

A Tauri 2 desktop app (Rust backend + React/TypeScript frontend) that provides a GUI for
[fw-fanctrl](https://github.com/TamtamHero/fw-fanctrl): fan curve editing, live telemetry,
and a tray profile switcher for Framework laptops on Linux.

## Key architectural facts

- **The app never elevates.** The fw-fanctrl service socket
  (`/run/fw-fanctrl/.fw-fanctrl.commands.sock`) is world-writable; the root service executes
  commands (including `set_config`, which writes `/etc/fw-fanctrl/config.json`) on behalf of
  any user. systemd unit operations use plain `systemctl`, which triggers a polkit dialog.
  Never add sudo/pkexec calls.
- The Rust backend shells out to `fw-fanctrl --output-format JSON <cmd>` and parses JSON.
- Fan RPM is read from sysfs (`/sys/class/hwmon/*/`, the `cros_ec` hwmon).
- Battery state is read from `/sys/class/power_supply/BAT*/status`. fw-fanctrl only
  distinguishes discharging; the app itself switches strategies on charge start/end
  (`AppCore::power_tick`, driven by the telemetry poller).

## Layout

- `src/` — React frontend. Pure logic in `src/lib/` (no React imports), hooks in
  `src/hooks/`, page components in `src/pages/`, shared UI in `src/components/`.
- `src-tauri/src/` — Rust backend. Effect boundaries are traits (`FanctrlClient`,
  `SystemdClient`, `HwmonReader`); production impls + mockall mocks swap behind them.
  `commands/` is a thin IPC layer that only delegates.
- `.github/workflows/quality.yml` — reusable lint/test/coverage job shared by CI and release.

## Commands

```bash
npm run lint        # eslint + prettier check + clippy (-D warnings) + rustfmt check
npm run format      # auto-fix all of the above
npm run test        # vitest + cargo test
npm run coverage    # coverage with 80% gates (vitest thresholds + cargo llvm-cov)
npm run tauri dev   # run the app locally
```

Git commits run `npm run lint` automatically via Husky (`.husky/pre-commit`). Hooks are
installed on `npm install` through the `prepare` script. Skip in a pinch with
`git commit --no-verify`.

Runtimes come from asdf (`.tool-versions`). CI runs on Ubuntu and installs
`libwebkit2gtk-4.1-dev libayatana-appindicator3-dev librsvg2-dev libgtk-3-dev libxdo-dev patchelf`;
locally on Arch the equivalents are `webkit2gtk-4.1 libappindicator-gtk3`.

## Conventions

- Files target <=300-400 lines; split by responsibility when exceeded.
- Minimal inline comments (only genuinely non-obvious logic). Public APIs get rustdoc
  `///` / TSDoc `/** */` instead.
- Curve math and config mutations are pure functions (`src/lib/`, `src-tauri`'s
  config module) so they are trivially testable.
- Frontend tests mock Tauri IPC with `@tauri-apps/api/mocks`; never spawn real processes
  in tests. Rust tests use mockall against the trait boundaries.
- TS types in `src/lib/types.ts` mirror the Rust serde structs manually; update both
  together.
- 80% coverage is enforced; write tests alongside features, not after.

## Release

Pushing a semantic tag (`vX.Y.Z`) triggers `.github/workflows/release.yml`: it re-runs
the quality workflow, verifies the tag matches all three version files, builds
deb/rpm/AppImage bundles with `tauri-apps/tauri-action`, publishes a GitHub Release,
and pushes an updated PKGBUILD to the AUR (`fw-fanctrl-gui-bin`).

Use `npm run bump -- <version>` to sync `package.json`, `src-tauri/Cargo.toml`,
`src-tauri/tauri.conf.json`, and both lockfiles; then commit, tag with `git tag -s`,
and push the tag. AUR publishing requires the `AUR_USERNAME`, `AUR_EMAIL`, and
`AUR_SSH_PRIVATE_KEY` repository secrets.

## Gotchas

- Tag version must match the three version files or the release workflow fails fast.
- `fw-fanctrl set_config` expects the _entire_ config JSON, not a patch.
- The service rejects deleting/renaming a strategy referenced by `defaultStrategy` or
  `strategyOnDischarging`; the UI guards this.
- GNOME needs the AppIndicator extension for the tray icon; this is a GNOME limitation,
  not a bug.
- AppImages are post-processed with `scripts/patch-appimage.sh` (strip bundled GPU libs,
  embed `packaging/appimage/bundle.yml`, repack as type2). They require host `fw-fanctrl`
  and GTK/WebKit libraries; see `packaging/aur/PKGBUILD-appimage` for the Arch wrapper.

---
> Source: [jslay88/fw-fanctrl-gui](https://github.com/jslay88/fw-fanctrl-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
