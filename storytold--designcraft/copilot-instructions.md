## designcraft

> DesignCraft is a clean-room, open-source, Rust-native page-layout application targeting Adobe InDesign parity — and superiority (speed, openness, agent control). It runs natively on macOS, Windows and Linux, and on the web via WASM. Siblings with the same conventions: `../drawcraft` (Illustrator-class), `../photocraft` (Photoshop), `../printcraft` (Acrobat), `../filmcraft` (Premiere), `../lightcraft` (Lightroom).

# DesignCraft — instructions for agents

DesignCraft is a clean-room, open-source, Rust-native page-layout application targeting Adobe InDesign parity — and superiority (speed, openness, agent control). It runs natively on macOS, Windows and Linux, and on the web via WASM. Siblings with the same conventions: `../drawcraft` (Illustrator-class), `../photocraft` (Photoshop), `../printcraft` (Acrobat), `../filmcraft` (Premiere), `../lightcraft` (Lightroom).

## Start every session here
1. Read `plan/STATUS.md` (current milestone, next task), then the task in `plan/execution-plan.md` and the relevant `plan/architecture.md` section. Behaviour reference: `plan/indesign/*.md` (`11-observed-ui.md` holds measured observations of the running app).
2. Follow the autonomous operation protocol (`plan/execution-plan.md` §7). Don't stop to ask unless §7 lists the decision as the user's.

`plan/` is gitignored (local only).

## Non-negotiables
- **Clean-room.** InDesign is installed on the dev machine and may be *observed* black-box: run it, use its UI with synthetic documents, take screenshots (by window id) stored only under `plan/indesign/screenshots/` (never committed). Never read, disassemble or copy anything inside the InDesign bundle (names/listings only), never copy Adobe icons, artwork, presets or wording beyond feature names, never commit files produced by InDesign. Never copy GPL/AGPL/LGPL code (Scribus, LibreOffice, Ghostscript…).
- **Assets:** no Adobe iconography or images — ever. Every asset is original / OSI / CC0 / redistributable CC and has a row in `ASSETS.md` (`cargo xtask assets` enforces it). Prefer art generated in code.
- **Everything is a command.** User-visible behaviour = a command in `crates/engine/src/cmd/*` (id, label, menu path, shortcut, params doc, `enabled`, `run`) + tests. Tools emit commands (Begin/Preview/Commit). UI-only commands live in `crates/ui-egui/src/menus.rs` (`UI_COMMANDS`). The control channel and MCP reach all of them.
- **Layering** is enforced by `cargo xtask layers`. Nothing below L6 depends on egui/eframe/winit/rfd. The UI crate is swappable.
- **The UI is thin**: panels read engine state and act through `app.run(id, params)`. Colours come from `theme::Tokens`.
- **Rust only** (no handwritten JS/TS). **Never break wasm** (`cargo xtask wasm`).
- **Quality gates** before every commit: `cargo xtask ci` (fmt, clippy -D warnings, tests, assets, layers, wasm). One task id per commit (`M2.1: type tool caret navigation`).

## Running and looking at the app
- `cargo run --release -p designcraft -- --sample --control 7979` (sample magazine + control channel).
- Drive it: JSON lines on `127.0.0.1:7979`, e.g. `{"id":1,"method":"engine.execute","params":{"command":"frame.create","params":{"rect":[36,36,300,200],"content":"text"}}}` then `{"id":2,"method":"ui.screenshot","params":{"path":"/tmp/shot.png"}}`. Methods: `crates/ui-egui/src/control.rs`, docs: `docs/control-protocol.md`.
- **For UI work, look at the result** (take `ui.screenshot`, read the PNG) and compare with `plan/indesign/02-ui-ux.md` / `11-observed-ui.md`. If no frame is presented (screen locked) use `ui.render` or `designcraft-cli run --sample --all-pages DIR`.
- Headless: `designcraft-cli run --sample --cmd 'frame.create={"rect":[0,0,100,100]}' --export out.png`.
- Shell gotcha: `mv`/`cp` are aliased interactive here — use `/bin/mv -f` / `/bin/cp -f`.
- Parallel agents: separate `CARGO_TARGET_DIR` per agent; edit only the crates you own; delete your target dir when done (disk).

## Roadmap
`ROADMAP.md` (committed) tracks status, milestones and estimates. Update it whenever a milestone task lands.

---
> Source: [storytold/designcraft](https://github.com/storytold/designcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
