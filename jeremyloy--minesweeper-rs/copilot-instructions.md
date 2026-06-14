## minesweeper-rs

> Guidance for working in this repository.

# CLAUDE.md

Guidance for working in this repository.

## What this is

A cross-platform desktop **Minesweeper** game written in Rust using the
**egui/eframe** immediate-mode GUI framework. Single binary, no assets, no
network, no persistence. Classic 9×9 / 10-mine "Beginner" board.

## Build & run

```sh
cargo run              # debug build + launch
cargo run --release    # optimized build + launch
cargo build            # compile only
cargo fmt              # format (rustfmt defaults)
cargo clippy           # lint
```

The code should compile with **zero warnings** — keep it that way.

### Web build (WASM)

The same `src/main.rs` also targets the browser. A `#[cfg(target_arch = "wasm32")]`
`main` mounts the app onto the `<canvas>` in `index.html` via `eframe::WebRunner`;
the native `main` is `#[cfg(not(target_arch = "wasm32"))]`. Both share `make_app()`.

```sh
trunk serve --release          # local dev server at http://localhost:8080
trunk build --release          # output to dist/
```

Key web-specific points:

- **Renderer split** (`Cargo.toml`): native uses eframe's default **wgpu**; the
  `wasm32` target uses **glow** (WebGL2) via per-target dependency tables. Because
  Cargo unifies features per target graph, native is completely unaffected.
- **Time:** use `web-time` (`web_time::Instant`/`SystemTime`), *not* `std::time` —
  `std`'s clock **panics** on `wasm32-unknown-unknown`. `web-time` re-exports `std`
  on native, so it's a no-cost alias there.
- **`on_exit` signature differs by renderer:** glow passes `Option<&glow::Context>`,
  wgpu takes none — hence the two `#[cfg]`'d `on_exit` methods.
- **Persistence** silently no-ops on web (no config dir), so the game still runs.
- **Deploy:** `.github/workflows/deploy-web.yml` builds with Trunk and publishes to
  GitHub Pages on push to `main`, using `--public-url "/<repo>/"` for project-site
  routing.

### Toolchain (important on Windows)

Build with the **MSVC** toolchain: `stable-x86_64-pc-windows-msvc`. It requires
the Visual Studio Build Tools ("Desktop development with C++" workload) for
`link.exe`.

Do **not** use the `x86_64-pc-windows-gnu` toolchain here: it links plain Rust
fine but fails on eframe's transitive `windows-*` crates with
`error calling dlltool 'dlltool.exe': program not found` (the GNU `raw-dylib`
path needs MinGW binutils; MSVC handles it natively). If you see that error,
run `rustup default stable-x86_64-pc-windows-msvc`.

On macOS/Linux the system C toolchain (Xcode CLT / `build-essential`) suffices.

## Architecture

Everything lives in **`src/main.rs`**, deliberately. The program is small enough
that a single file is easier to read end-to-end than a module tree. It is
organized in three clear sections:

1. **State** — `Minesweeper` (the whole game), `Cell`, and the `CellState` /
   `Status` enums. `grid` is a `Vec<Vec<Cell>>` indexed `[row][col]`.
2. **Rules / logic** — methods on `Minesweeper`: `place_mines`, `reveal`,
   `flood_reveal`, `chord`, `toggle_flag`, `check_win`, `lose`. These contain no
   UI code and could be unit-tested in isolation.
3. **Rendering** — the `eframe::App` impl (`ui`) plus the `draw_cell` /
   `draw_flag` / `draw_mine` painter helpers.

### Key design decisions

- **Logic is separated from rendering.** Game-state methods never touch egui;
  the render layer never mutates state directly. During a frame, clicks are
  *collected* into local `Option<(r,c)>` values and applied **after** the cell
  loop — this both keeps the borrow checker happy (no mutating `self` while
  iterating its grid for display) and centralizes state transitions.
- **First-click safety** is implemented by deferring mine placement until the
  first reveal (`mines_placed` flag). `place_mines(safe_r, safe_c)` excludes the
  clicked cell *and its 8 neighbors* from candidates, then runs a partial
  Fisher–Yates shuffle. There's a fallback (exclude only the clicked cell) for
  hypothetical boards too dense for a full 3×3 safe zone.
- **Flood fill is iterative** (an explicit stack), not recursive, so a large
  empty region can never blow the call stack.
- **Cells are drawn as shapes, not glyphs.** Flags, mines, and the beveled
  raised look are painted with `egui::Painter` primitives (polygons, circles,
  lines). This avoids depending on emoji-font coverage, which varies by
  platform and by egui's bundled font subset — the board looks identical on
  Windows, macOS, and Linux. Only the digits 1–8 use text (ASCII, always
  available), with the classic per-number colors in `number_color`.

## Conventions

- **Formatting:** rustfmt defaults. Run `cargo fmt` before finishing.
- **Naming:** `snake_case` items, `CamelCase` types, `SCREAMING_CASE` consts.
  Board parameters are the consts `ROWS`, `COLS`, `MINES`, `CELL` at the top of
  the file — change difficulty by changing those, not by sprinkling literals.
- **Coordinates** are always `(row, col)` order, both in storage and arguments.
- **Comments** explain *why*, not *what*. Non-obvious rules (first-click safety,
  chording, the click-collection pattern) carry a short rationale comment;
  mechanical code does not.
- **Dependencies:** keep them minimal. We deliberately avoid pulling in `rand`
  for 10 mines — a tiny seeded `xorshift64` (`Rng`) does the job. Add a crate
  only when it earns its place.
- **No `unwrap()` on fallible runtime paths.** Time/seed code uses
  `unwrap_or(...)` fallbacks.

## Testing philosophy

The game logic is intentionally UI-free and therefore unit-testable. There are
no tests yet; when adding them, prefer a `#[cfg(test)] mod tests` in
`src/main.rs` that exercises the pure logic (e.g. mine-count after placement,
first-click safe-zone invariant, flood-fill reveal counts, win detection)
without instantiating egui. Don't try to test rendering — assert on game state.

## Why these tools

- **Rust** — the requested language; gives a single self-contained native binary
  with no runtime to install.
- **egui + eframe** — pure-Rust immediate-mode GUI. Chosen over alternatives
  because it is genuinely cross-platform (Windows/macOS/Linux, and even
  WASM/web) from one codebase, has no C/system-GUI dependencies to wrangle, and
  immediate-mode suits a board that fully re-renders each frame from simple
  state. `eframe` is egui's official windowing/run-loop wrapper.
- **No game engine** — Minesweeper is a grid of buttons; a full engine
  (Bevy, etc.) would be overkill.

## Gotchas

- eframe 0.34 uses the newer API: implement `App::ui(&mut self, ui, frame)` (not
  the deprecated `update(ctx, frame)`), and show panels with
  `Panel::top(...).show_inside(ui, ...)` / `CentralPanel::default().show_inside(...)`.
- The process spawns more than one OS window (a winit detail); `MainWindowHandle`
  is not reliably the game window if you script screenshots — enumerate and pick
  the largest visible window.
- `windows_subsystem = "windows"` is set for release builds only, so the release
  binary doesn't pop a console window.

See also `TODO.md` for planned enhancements.

---
> Source: [JeremyLoy/minesweeper-rs](https://github.com/JeremyLoy/minesweeper-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
