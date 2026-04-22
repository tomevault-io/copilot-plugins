## go-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Run the local server (then open http://localhost:8080 in a browser)
go run main.go

# Build a binary
go build -o server main.go

# Change the port
PORT=9000 go run main.go
```

No build step for the frontend — all JS/CSS/HTML is served as static files from `static/`.

## Architecture

All game logic lives in the browser. The Go server (`main.go`) is a thin static-file server with no game-specific routes — it exists purely so we can `go run` and get a URL, and so it's easy to extend into a real server later.

### Frontend layers (strict separation)

| File | Responsibility |
|---|---|
| `static/js/board.js` | `Board` class — all game state and rules. **No DOM access.** |
| `static/js/render.js` | `Renderer` class — Canvas drawing only. Reads `Board` state, never writes it. |
| `static/js/ui.js` | Wires DOM events → `Board` methods → `Renderer.draw()`. Owns all HTML element references. |

### Game flow

1. `index.html` — user picks board size (9/13/19) and toggles komi → navigates to `game.html?size=N&komi=K`
2. `ui.js:init()` parses URL params, creates `Board` and `Renderer`, sizes the canvas.
3. Canvas click → `board.placeStone()` or (in scoring phase) `board.toggleDeadGroup()` → `renderer.draw()`.
4. Pass/End Game → `board.pass()` / `board.endGame()` → enter scoring phase (controls swap to scoring panel).
5. Scoring phase: players click groups to mark dead, score preview updates live via `board.calculateScore()`.
6. Confirm Score → `board.finalizeScoring()` → display winner.

### Key Board rules implemented

- **Stone capture** — opponent groups with zero liberties are removed after each placement.
- **Suicide prevention** — a move that leaves the placed group with zero liberties (after captures) is rejected.
- **Ko rule** — simple ko: if exactly 1 stone was captured and the played group is size 1, that capture point is forbidden for the opponent on the very next move.
- **Territory scoring** — Japanese-style: territory (flood-fill) + captures + dead opponent stones + komi for White.

### Adding features later

- **Cosmetics/theming** — all visual decisions are in `render.js` and `style.css`. Board colours, stone gradients, coordinate labels go there.
- **Server features** (auth, persistence, multiplayer) — add routes to `main.go`; the static frontend is already served from a handler that can coexist with new API routes.
- **Undo / move history** — `Board` would need a `history` stack; `placeStone` would push a snapshot before each move.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ardan-S) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
