## wordguess

> A Wordle-style static word-guessing game built in plain HTML, CSS, and JavaScript — no framework, no build step, no dependencies.

# WordGuess

A Wordle-style static word-guessing game built in plain HTML, CSS, and JavaScript — no framework, no build step, no dependencies.

## Project overview

Players get 6 attempts to guess a hidden 5-letter word. Each guess reveals per-letter feedback: correct position (green), wrong position (yellow), or absent (gray). The game resets on page reload with a new word drawn from a bundled word list.

## Architecture

Single-page application delivered as static files:

```
wordguess/
├── index.html        # Shell: grid, keyboard, modal scaffolding
├── style.css         # All visual styles (see design system skill)
├── app.js            # Game logic: state machine, input handling, win/lose
├── words.js          # Exported word list and daily-word picker
└── .claude/
    ├── skills/
    │   └── design-system/SKILL.md
    └── agents/
        └── design-reviewer.md
```

## Build slices

The game is delivered in vertical slices — each slice is a playable or reviewable increment:

| Slice | Deliverable |
|-------|-------------|
| 1 | Static HTML shell + CSS grid/keyboard (no logic) |
| 2 | `words.js` word list + random word picker |
| 3 | Input handling: physical keyboard + on-screen keyboard |
| 4 | Guess evaluation + tile colour flip animation |
| 5 | Win/lose detection + end-game modal |
| 6 | Accessibility pass (ARIA live regions, focus management, reduced-motion) |
| 7 | Polish: hard-mode toggle, high-contrast mode, share button |

## Conventions

- No `id` selectors in JS — use `data-*` attributes for DOM queries.
- Game state lives in a single plain object; no globals beyond `window.WG`.
- CSS custom properties only — no magic numbers in stylesheet rules.
- All colour decisions live in the design system skill; never hardcode hex values elsewhere.
- Commits follow Conventional Commits (`feat:`, `fix:`, `style:`, `a11y:`).

## Running locally

```bash
# Any static file server works; Python ships with one:
python3 -m http.server 8080
# then open http://localhost:8080
```

No build step required.

---
> Source: [nyeinchan-lwin/wordguess](https://github.com/nyeinchan-lwin/wordguess) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
