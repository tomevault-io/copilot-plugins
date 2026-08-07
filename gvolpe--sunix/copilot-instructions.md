## sunix

> Guidance for AI agents working on SUNix.

# AGENTS.md

Guidance for AI agents working on SUNix.

## Project Overview

SUNix is a small Rust GTK4 layer-shell popup for Wayland compositors. It shows expected package changes before a NixOS or Home Manager switch by building the configured flake output and diffing it with `dix`.

The project intentionally has a small dependency surface:

- Rust 2024, `rust-version = "1.92"`.
- GTK via `gtk4`.
- Layer-shell integration via `gtk4-layer-shell`.
- JSON parsing via `serde` and `serde_json`.
- No CLI parser crate; CLI parsing is currently hand-written in `src/cli.rs`.
- No TOML parser crate; config parsing is currently a simple `key=value` parser in `src/config.rs`.

## Repository Layout

- `src/main.rs`: process startup, CLI parsing, renderer default, config load, UI entrypoint.
- `src/cli.rs`: supported CLI flags. Keep this dependency-free unless there is a strong reason to add a parser crate.
- `src/config.rs`: `sunix.toml` loading and parsing.
- `src/command.rs`: generic command execution, failure formatting, and streaming stderr logs for the loading UI.
- `src/dix.rs`: Nix/Home Manager build orchestration, `dix` execution, and dix JSON parsing. Keep generic process-management code out of this file.
- `src/format.rs`: display formatting for versions and sizes.
- `src/model.rs`: UI-facing report data model.
- `src/ui.rs`: GTK application setup, layer-shell window setup, stylesheet loading.
- `src/ui/*.rs`: split UI modules for chooser, messages/loading, navigation/keybindings, reports, state, and widgets.
- `assets/style.css`: default full stylesheet, embedded with `include_str!`.
- `assets/sample.json`: bundled demo report.
- `nix/drv.nix`: Nix package.
- `nix/shell.nix`: development shell.
- `nix/hm.nix`: Home Manager module exposed as `homeModules.default` from `flake.nix`.
- `.github/workflows/ci.yml`: CI builds Linux systems and runs flake checks.

## Common Commands

Use these before handing back code changes:

```console
cargo fmt
cargo test
cargo clippy --all-targets --all-features
```

Keep clippy clean; do not leave new warnings behind.

`cargo test` may report only unit tests because this project currently has no integration test suite.

Nix checks/builds:

```console
nix build
nix flake check --all-systems
nix eval .#homeModules.default --apply 'x: "ok"'
```

Use `nix develop` for the full local development environment.

## Runtime Behavior

Required config lives at:

```console
$XDG_CONFIG_HOME/sunix/sunix.toml
```

Required fields:

```toml
flake_dir=$HOME/workspace/nix-config
home_flake=niri-hdmi
nixos_flake=aorus
```

Optional fields:

```toml
home_flake_dir=$HOME/workspace/home-config
nixos_flake_dir=$HOME/workspace/nixos-config
dix_binary=/path/to/dix
style_css=/path/to/custom-style.css
show_demo=true
```

`home_flake_dir` and `nixos_flake_dir` take precedence over `flake_dir` for their respective report types. If unset or empty, they fall back to `flake_dir`.

`style_css` is a full replacement for `assets/style.css`. If unset or empty, SUNix uses the bundled stylesheet. If the configured file cannot be read, SUNix shows a config error using the bundled stylesheet.

`dix_binary` is intentionally still named `dix_binary`; do not replace it with `diff_command` unless the user asks again.

## Report Flow

Home Manager:

1. Resolve the active Home Manager profile symlink. Try `${XDG_STATE_HOME:-$HOME/.local/state}/nix/profiles/home-manager` first, then `/nix/var/nix/profiles/per-user/$USER/home-manager`.
2. Run:

   ```console
   nix build --print-out-paths --no-link .#homeConfigurations.<home_flake>.activationPackage
   ```

3. Run `dix --output=json <old-generation> <new-output-paths...>`.
4. Parse dix JSON into `UpdateReport`.

NixOS:

1. Run:

   ```console
   nix build --print-out-paths --no-link .#nixosConfigurations.<nixos_flake>.config.system.build.toplevel
   ```

2. Run `dix --output=json /run/current-system/ <new-output-paths...>`.
3. Parse dix JSON into `UpdateReport`.

The `nix build` command is shared through `nix_build_output_paths`. Keep the shared command construction factored so Home Manager and NixOS do not drift.

The loading popup streams stderr from `nix build` through `src/command.rs`. It keeps stdout captured for output path parsing and does not stream `dix` JSON output to the UI.

## UI Notes

The first screen is the actual chooser, not a landing page.

Keyboard shortcuts:

- `M`: Home Manager report.
- `N`: NixOS report.
- `D`: Demo report, only when `show_demo=true` or `sunix --demo`.
- `Up` / `K`: scroll up.
- `Down` / `J`: scroll down.
- `Left` / `H`: back.
- `Esc`: close.

Report results are cached for the current SUNix process.

The loading log is intentionally six fixed single-line GTK labels, not one multiline label. This prevents long Nix log lines from wrapping into more visible rows. If changing it, preserve:

- Latest 6 log entries only.
- Fill from the top when fewer than 6 lines are available.
- Single-line ellipsized rows.

## Styling

Default styles are embedded from `assets/style.css`. When adding CSS classes, document them by keeping names descriptive and scoped to the component (`loading-log`, `back-button`, `changes-grid`, etc.).

Custom styling is done via the `style_css` config option. Do not add GTK theme-name support unless requested; the current design is file-based stylesheet replacement.

Keep layout dimensions stable. Buttons, keycaps, report grids, and loading logs should not resize or shift when content changes.

## Nix And Home Manager

`flake.nix` exposes:

- `packages.<system>.default`
- `devShells.<system>.default`
- `overlays.default`
- `homeModules.default`

When adding config options, update all relevant places:

- `src/config.rs`
- `README.md`
- `nix/hm.nix` under `programs.sunix.settings`
- Parser tests in `src/config.rs`

Home Manager option names use camelCase and generate snake_case TOML keys. Example: `settings.styleCss` writes `style_css`.

The module should keep all user configuration under `programs.sunix.settings`.

## Parser Notes

The config parser supports simple `key=value` lines with optional single or double quotes. It strips `#` comments outside quotes and expands these home forms for path fields:

- `$HOME`
- `${HOME}`
- `~`
- `$HOME/...`
- `${HOME}/...`
- `~/...`

Do not assume full TOML semantics unless you also replace the parser and update tests/docs accordingly.

For dix JSON parsing, support the known version entry shapes:

- `changed` with `old` and `new`.
- `added` or `removed` with `version`.
- `amount_changed` with `version`, `old_amount`, and `new_amount`.

Downgraded reports should display a higher old version and lower new version when the data allows it.

## Development Gotchas

Because `assets/style.css` is embedded with `include_str!`, stale Cargo build artifacts can make `cargo run` appear to use an older stylesheet after packaging/dry runs. If behavior looks stale, run:

```console
cargo clean -p sunix
cargo run
```

For crate dry runs, prefer a separate target directory to avoid confusing local debug builds:

```console
CARGO_TARGET_DIR=/tmp/sunix-package-target cargo publish --dry-run
```

When cross-building Nix outputs, `nix build --system aarch64-linux` on a non-aarch64 machine requires a working remote builder, emulator, or native runner. CI uses an arm runner for `aarch64-linux`.

## Change Discipline

- Prefer small, scoped changes that follow the existing module split.
- Preserve user changes in the working tree; do not reset or checkout files unless explicitly asked.
- Use `rg` for search.
- Use `apply_patch` for manual edits.
- Keep generated docs and examples consistent with the actual code and Home Manager module.

---
> Source: [gvolpe/sunix](https://github.com/gvolpe/sunix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
