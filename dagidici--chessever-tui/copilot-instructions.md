## chessever-tui

> Guide for AI coding agents (and humans) working on **chessever-tui**.

# AGENTS.md

Guide for AI coding agents (and humans) working on **chessever-tui**.

## What this is

Pure-Dart terminal UI chess client. Plays the [Maia](https://maiachess.com)
network locally via `lc0` over UCI. Renders a pixel-art board with
[nocterm](https://nocterm.dev). MIT-licensed, open source.

Repo: <https://github.com/Chessever/chessever-tui>
Install: `curl -fsSL https://tui.chessever.com/install.sh | sh`

## Layout

```
bin/chessever_tui.dart      # CLI entry. Parses argv, calls runApp().
lib/
  app.dart                  # Top-level NoctermApp / tab shell
  shell/                    # Layout shell + sidebar (tabs)
  play/
    play_pane.dart          # Pre-game config screen
    active_game.dart        # Running game: clock, history, drag, sfx
    board.dart              # BoardView: cells, highlights, mouse routing
    pieces.dart             # Head-only PieceSprite glyphs (full/compact/mini)
    play_config.dart        # ELO / time control / side dataclass
  engine/                   # ChessEngine interface + Maia (lc0) + Vibes fallback
  audio/sfx.dart            # WAV playback for move/capture/check/etc.
  settings/                 # Settings tab + ChesseverSettings model
  commands/                 # Commands tab (curated commands feed)
  update/                   # In-app updater (downloads latest release)
  theme/colors.dart         # Brand palette (ported from frontend_desktop)
assets/
  pixelart/                 # Reference sprite sheet
  sfx/                      # Move/capture/check WAVs (bundled in binary? no — runtime)
.github/workflows/release.yml  # CI build + publish
pubspec.yaml                # Dart package manifest
```

## Build

Local dev:

```bash
dart pub get
dart analyze --fatal-infos      # must pass; CI gate
dart run bin/chessever_tui.dart # run from source
```

Produce a single-binary release locally:

```bash
dart compile exe bin/chessever_tui.dart -o build/out/chessever
```

The output is a self-contained native executable. No Flutter, no runtime
dependencies beyond `lc0` + Maia weights at runtime (engine is optional —
falls back to a built-in "vibes bot" if missing).

## Release flow

Driven entirely by `.github/workflows/release.yml`.

### Triggers

| Trigger                  | Result                                                |
|--------------------------|-------------------------------------------------------|
| push to `main` / `master`| Republishes the moving `tui-latest` GitHub Release    |
| push tag matching `v*`   | Publishes a tagged release (e.g. `v0.3.0`)            |
| `workflow_dispatch`      | Builds artifacts only, does not publish               |

### Matrix

| Label         | Runner          | Output                                  |
|---------------|-----------------|-----------------------------------------|
| macos-arm64   | macos-14        | `chessever-tui-macos-arm64.tar.gz`      |
| linux-x64     | ubuntu-latest   | `chessever-tui-linux-x64.tar.gz`        |
| windows-x64   | windows-latest  | `chessever-tui-windows-x64.zip`         |

Each archive contains `chessever` (or `chessever.exe`) plus the legacy
`chessever-tui` alias. Each is paired with a `.sha256` sidecar.

> macOS Intel (x86_64) is **not** built — `macos-13` runner is deprecated.
> Intel users currently run via Rosetta on the arm64 binary.

### What `main` pushes do

1. Build matrix runs `dart pub get` → `dart analyze --fatal-infos` → `dart compile exe`.
2. The `release` job downloads all artifacts.
3. The `tui-latest` git tag is force-moved to the new commit.
4. `softprops/action-gh-release@v2` republishes the `tui-latest` Release
   with `make_latest: true`, so `releases/latest` always points at it.

### Public download URLs

```
https://github.com/Chessever/chessever-tui/releases/latest/download/chessever-tui-macos-arm64.tar.gz
https://github.com/Chessever/chessever-tui/releases/latest/download/chessever-tui-linux-x64.tar.gz
https://github.com/Chessever/chessever-tui/releases/latest/download/chessever-tui-windows-x64.zip
```

`tui.chessever.com/install.sh` reads these. The in-app **Update** tab also
hits `releases/latest`.

### Cutting a versioned release

1. Bump `pubspec.yaml` `version:` (semver).
2. **Also bump** `const _version` in `bin/chessever_tui.dart` — it powers
   `chessever --version` and is NOT auto-synced from pubspec. Easy to miss.
3. Commit, tag, push:
   ```bash
   git tag v0.3.0
   git push origin main v0.3.0
   ```
4. Both jobs fire: tag → `v0.3.0` release with generated notes;
   main push → `tui-latest` updated. Tagged releases include
   `generate_release_notes: true`.

## Code conventions

- **Pure Dart, no Flutter.** Do not pull `flutter/*` packages — would
  inflate binary size and break the build matrix.
- **nocterm idioms**: components extend `StatelessComponent` /
  `StatefulComponent`. `MouseRegion` for drag tracking, `GestureDetector`
  for clean taps. `LayoutBuilder` to read terminal size.
- **Density tiers** in `BoardView`: `full` (7×3 cell), `compact` (5×2),
  `mini` (3×1). Picked dynamically in `active_game.dart` from terminal
  `constraints.maxHeight` / `maxWidth`. Side panel rows scale too.
- **Sprites** are head-only — each piece's top row is its identity (pawn `●`,
  knight asymmetric, bishop `▲`, rook `█ █ █`, queen `◆ ▲ ◆`, king `╋`).
  Do not re-introduce the foot row; pieces became indistinguishable.
- **Colors** live in `lib/theme/colors.dart`. Re-use named tokens
  (`ChesseverColors.lastMoveLight`, etc.); avoid raw `Color.fromRGB` calls
  outside the theme file.
- **Animations** (`AnimationController` via `TickerProviderStateMixin`) must
  be `dispose()`d. When using `forward(from: 0)` for a flash, reset the
  value in `whenComplete` or the controller will sit at `1.0` and the
  visual state persists (see prior bug: stuck `_moveFlash` made the TO
  square stay primary-tinted forever).

## Testing

No automated UI tests yet. Manual smoke test:

```bash
dart analyze --fatal-infos
dart compile exe bin/chessever_tui.dart -o /tmp/chesst && /tmp/chesst
```

Resize the terminal mid-game to verify density tiers switch. Try clicking,
dragging, keyboard input, and engine play.

## Things to know

- **`lc0` + Maia weights** are runtime dependencies, installed by the user
  under `~/.chessever-tui/weights/`. The binary does not bundle them.
- **SFX assets** in `assets/sfx/` are loaded at runtime — they must be
  present alongside the binary OR the SFX subsystem no-ops. Check
  `lib/audio/sfx.dart` for the resolution logic.
- **`tui-latest`** is a force-moved tag. Do not rely on its SHA being
  stable. For reproducible installs, use a `v*` tag.
- **`install.sh`** lives on the `tui.chessever.com` host, not in this repo.
  If you change asset names / archive layout, coordinate with that script.
- **CI must stay green** — `dart analyze --fatal-infos` is the gate; even
  info-level diagnostics fail the build.

## Contributing

PRs welcome. Keep the diff focused. Open an issue first for anything that
touches the release workflow, the install script's contract, or the
engine interface.

---
> Source: [dagidici/chessever-tui](https://github.com/dagidici/chessever-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
