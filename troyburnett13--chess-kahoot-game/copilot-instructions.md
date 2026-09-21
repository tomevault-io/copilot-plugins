## chess-kahoot-game

> "Rook & Riddle" is a single-page, static chess trivia game built for a live, host-led game night with four teams. There is no backend and no build step — it is plain HTML/CSS/JS served as a static site on Netlify.

# AGENTS.md

## Project

"Rook & Riddle" is a single-page, static chess trivia game built for a live, host-led game night with four teams. There is no backend and no build step — it is plain HTML/CSS/JS served as a static site on Netlify.

## Architecture

- `index.html` — the only HTML page. It contains three `<section class="screen">` blocks (setup, game, end) that are toggled via the `.active` class in JS. There is no router.
- `css/style.css` — all styling, using CSS custom properties (defined on `:root`) for the palette. Chess-felt/ivory/brass theme; the four answer options and the four team colors are deliberately two separate color sets so a host never confuses "which answer is right" with "which team scored."
- `js/questions.js` — the question bank only (`QUESTIONS` array: `category`, `question`, `options[4]`, `correct` index). Pure data, no logic. Edit this file to change quiz content.
- `js/app.js` — all game state and rendering logic. State shape is defined in `defaultState()`. The whole app is one mutable `state` object re-rendered imperatively (no virtual DOM) — `renderScoreboard()`, `renderQuestion()`, `renderAwardRow()`, `renderEnd()`.

## State & persistence

Game state (team names, scores, current question position, whether shuffled) is persisted to `localStorage` under `rookAndRiddleState_v1` after every mutation via `saveState()`. This is intentional: the game is run by one host on one device/screen during a single sitting, so `localStorage` is sufficient and avoids needing any Netlify backend primitive. If a future version needs multi-device sync (e.g. teams answering on their own phones), that would be the point to introduce Netlify Blobs or the database — see the `netlify-database`/`netlify-blobs` skills — rather than extending localStorage.

## Conventions

- No framework, no bundler. Keep it that way unless the scope changes significantly (e.g. adding real-time multiplayer).
- Keep question data (`js/questions.js`) free of any rendering/DOM logic.
- Answer option colors are fixed by index (0=garnet, 1=blue, 2=gold, 3=forest) via `.answer-btn[data-opt="N"]` — don't reorder options when editing questions unless intentionally changing the visual mapping.

## Roadmap

There is no `PLAN.md` — this was built as a complete, single-purpose tool in one pass. Future extensions (remote/multi-device play, a question editor UI, saved game history) would be new, separately-scoped features rather than a continuation of an existing plan.

---
> Source: [troyburnett13/Chess-Kahoot-Game](https://github.com/troyburnett13/Chess-Kahoot-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
