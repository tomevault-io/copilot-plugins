## meteobar

> Weather widget for Waybar (Rust). See the workspace `AGENTS.md` for the shared

# AGENTS.md — meteobar

Weather widget for Waybar (Rust). See the workspace `AGENTS.md` for the shared
widget contract (exit 0 with valid Waybar JSON on every path) and the AUR rules.

- Build: `make build`; install: `make install PREFIX=~/.local`. Lint: `cargo clippy`; format: `cargo fmt`.
- Tests: `cargo test` (unit tests in `src/structured.rs` and `src/cache.rs`).

## Non-Obvious Rules

- **Quickshell emits NEITHER `started` NOR `exited` when the command does not exist** — `running` just drops back to false. `sawExit` is the discriminator: no `exited` = the run could not start; an `exited` run with empty output is an operational failure, never "not installed".
- **`installCmd` is the one constant** — the message shows it and the button copies it (`Util.execArgv(["wl-copy", ...])`, no shell line, no trailing newline). The button gates on `notInstalled`, never on error text. Pinned in `tests/plugin_qml.rs`.

- Output must be valid Waybar JSON (`{"text": ..., "tooltip": ..., "class": ..., "alt": ...}`)
- `--output json` is a second, structured output mode (raw data, no Pango; consumed by the Omarchy shell plugin in `omarchy/`) — it must always exit 0 with valid JSON, errors go in the `error: {message}` field
- Forecast selection lives ONLY in `forecast.rs` (`upcoming_hours`, `forecast_days`); both the Waybar tooltip and the structured JSON render those slots, so "next N hours" and day/night are identical on both surfaces. Never index the API's parallel arrays — they can disagree in length in a cached payload and that used to panic the Waybar path; zip them instead
- The core publishes `palette` in the structured JSON (text/dim/accent, temp_cold/temp_warm, and `precip_ramp` as `{pct, color}` stops). QML consumes it via `rampColor()`, which mirrors `theme.rs::ramp_color`. Thresholds and colors live in the core; the panel must not re-derive them
- The response cache is request-keyed (`weather-<hash>.json`, hash over location input + units + days + hours), so different flag sets never cross-serve payloads
- Tooltip uses Pango markup for colors and formatting — escape user-facing strings
- `--no-color[=all|bar|tooltip]` (plus `NO_COLOR`, which the explicit flag overrides) resolves to a `ColorChoice` in `waybar.rs`; all color markup goes through `Paint`, which emits nothing when disabled. Monochrome drops color ONLY — glyphs, box drawing, bold, alignment and the `class`/`alt` fields all stay, and `--output json` is byte-identical either way
- Argument errors go through `report_cli_error` (not clap's default exit 2): they emit a waybar error object, or a structured error when the raw argv asked for `--output json`, always exit 0. `--help`/`--version` print normally
- Tooltip always uses Nerd Font icons regardless of `--icons` setting (for monospace alignment)
- Response cache uses flock-based file locking (`cache.rs`) with 60s TTL
- Theme resolution chain (`theme.rs::load_from`): Omarchy theme at `$XDG_STATE_HOME/omarchy/current/theme/colors.toml` (default `~/.local/state/...`, legacy `~/.config/omarchy/...` as fallback) → pywal cache at `$XDG_CACHE_HOME/wal/colors.json` (default `~/.cache/...`) → built-in One Dark defaults. pywal is consulted only when no Omarchy theme file was found
- Omarchy theme keys: prefer the semantic names current themes ship (`accent`, `foreground`, `background`, `red`, `green`, `yellow`, `orange`); `color1/2/3` are the legacy fallback
- pywal mapping: `special.foreground`/`special.background` → text/dim blend, `color4` (fallback `special.cursor`) → accent, `color2` → green, `color3` → yellow, `color1` → error. pywal has no orange slot — synthesize it as the yellow⊕red midpoint, never alias it to red (that flattens gauges across the widget family)
- A missing or non-hex value must only affect its own field — never sink the whole theme load. Values are validated with the strict `is_hex_color` (`#rgb`/`#rgba`/`#rrggbb`/`#rrggbbaa`) so malformed colors can't reach Pango markup
- Theme loading must never panic or error: absent/unreadable/invalid files degrade silently to the next tier, preserving exit 0
- Font Awesome icons are wrapped in Pango markup (`<span>`) for correct rendering in Waybar

## Release

A release is automated by pushing a tag — do NOT build or upload the binary by hand:

1. Merge the work into `master`. In the release commit (`chore: release X.Y.Z`): bump `version` in `Cargo.toml` + `Cargo.lock` AND in `manifest.json` (the marketplace shows the manifest's version; it must equal the tag). Push.
2. `git tag vX.Y.Z && git push origin --tags`.
3. The tag push triggers `.github/workflows/release.yml`, which builds and publishes the GitHub release with the asset `meteobar-X.Y.Z-x86_64-linux` (consumed by the `meteobar-bin` AUR package).
4. Only after the release exists, bump both AUR repos (`aur/meteobar` source + `aur/meteobar-bin`) per the workspace `AGENTS.md`. Order matters: `updpkgsums` fetches the tag tarball AND the release asset, so both must already be live.

---
> Source: [mryll/meteobar](https://github.com/mryll/meteobar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
