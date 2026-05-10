## test-claude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

Open `tictactoe.html` directly in a browser — no build step or server required:

```bash
open tictactoe.html
```

## Repository workflow

All changes must be committed with a descriptive message and pushed to GitHub:

```bash
git add <file>
git commit -m "<descriptive message>"
git push
```

Remote: `https://github.com/TheBiotechGuy/test_claude`

## Architecture

`tictactoe.html` is a single-file app — HTML structure, CSS, and JavaScript are all inline.

**Game state** (module-level variables):
- `board` — `Array(9)` of `null | 'X' | 'O'`, indexed 0–8 left-to-right, top-to-bottom
- `current` — whose turn it is (`'X'` or `'O'`)
- `gameOver` — boolean gate that blocks further moves
- `vsCPU` — toggles human vs CPU mode
- `scores` — `{ X, O, D }` persists across restarts

**Key functions:**
- `init()` — resets board, DOM classes, and turn state (scores are NOT reset)
- `applyMove(i)` — writes a move to `board` and DOM, calls `checkWinner()`, advances turn
- `checkWinner()` — checks `WINS` array against live `board`; returns `{ winner, line }` or `null`
- `checkWinnerFor(b)` — pure version used by minimax (takes a board snapshot, returns winner string or `null`)
- `minimax(b, isMax)` — recursive minimax; O is the maximizing player (+10), X is minimizing (-10)
- `cpuMove()` — picks the best move via minimax and calls `applyMove`

**CSS class conventions on `.cell`:**
- `.x` / `.o` — applied on move, sets text color
- `.taken` — blocks hover style and click handling
- `.win` — applied to the winning three cells, triggers pulse animation

---
> Source: [TheBiotechGuy/test_claude](https://github.com/TheBiotechGuy/test_claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
