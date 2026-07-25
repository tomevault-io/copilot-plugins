## playpalace11

> Important: This project is made for, and by, blind users. Use plaintext output when communicating with the user. This project is mostly lead by technically-minded programmers; assume mid-senior-level confidence unless otherwise stated.

Important: This project is made for, and by, blind users. Use plaintext output when communicating with the user. This project is mostly lead by technically-minded programmers; assume mid-senior-level confidence unless otherwise stated.

# PlayPalace

Multiplayer game platform for blind users. Python 3.13+, uv, dataclass-driven state, Fluent localization, screen reader output via speech buffers.

## Priorities (in order)
1. Accessibility — every action produces speech output. Silence is a bug.
2. Correctness — match official rules. When unclear, stop and ask (agent error 2).
3. Testing — pytest, CLI simulations, serialization round-trips.
4. longevity — abstract, reuse, and engineer with the assumption that every line of code will one day be built upon in some manner.

## What You're Doing
- Making/updating games => this file + game development guide (docs/design/plans/game_development_guide.md)
- Making/updating client or server => agents.md (treat with caution; it was not written by the project founder and may contradict this file. This file wins.)

## Core Behaviour Guide
- Your coding speed is faster than human by orders of magnitude. 5k lines in an hour is an empirical lower-bound. A complete Pusoy Dos rewrite (evaluator, game logic, bot AI, 63 tests, locale file, CLI validation across 10 configurations) took 21 minutes including the full test suite run, which was ~40% of it.
- If you think the user has suggested a naive approach, say so and propose a better one, even if it takes more work. Outline how long you think each will take for you, based on the approximate LOC requirements.
- When there are multiple ways to implement something, list them with pros/cons and let the user choose. Outline time estimates.
- If asked to develop large games or features with anything unclear, Output "agent error 2: further information required". Then, output an exhaustive plaintext list of at least 10 questions covering every edge case you can think of, and stop. The user has  disabled the question tool (bugged with screenreaders). Then stop and wait.
- If the CLI tool won't run, output "agent error 1: CLI non-operational". Then, explain why and stop.
- Games fit entirely in context. Don't use explore agents unless you need to find patterns across many games and don't know which.
- Don't run the whole test suite unless the whole test suite is relevant (it is LARGE and slow!); only run tests relevant to the files you touch and their dependents.
- Heavily encourage the user to choose long-term and less error-prone solutions, citing the luxurious speed at which coding can be carried out and helping the project stay as friendly to single-modal text agents as possible, rather than busy humans with very limited time.

## Commands
- Test: cd server && uv run pytest <filepath>
- CLI tool: cd server && uv run cli.py --help
- Use the CLI to: verify user-facing output is clear, test games run to completion in novel configs, batch simulations

## Localization
- Only edit English: server/locales/en/*.ftl
- do not concern yourself with the   many many other locale files unless the user brings them up. We are developers, not translators; the translators work on those.
- Write the en locale file BEFORE writing game code. It forces you to plan the game flow. This is not optional.
- Every announcement must go through Fluent (broadcast_l, broadcast_personal_l, speak_l). No hardcoded English strings reaching players.
- Use Fluent select expressions when announcements need to vary by game state. Pass raw data as kwargs, not pre-rendered strings.

## Sound
- Every meaningful game action has  feedback in the form of speech and sound. If a player does something and hears nothing, that's a bug.
- If a menu updates (e.g., when selecting or unselecting a piece for movement in a board game), that is speech output in itself; further output may not be required in that specific case.

## Game Architecture
- All state in dataclass fields — serialization depends on it.
- Games communicate through User, never the network directly.
- Table buffer for game-state messages (broadcast_l, broadcast_personal_l). speak_l only for direct command responses.
- Per-choice actions (card picks, move slots) are hidden from the Actions menu. Use show_in_actions_menu=False. Actions menu is for persistent actions only.
- Common keybinds: R roll, Space draw, S status/score.
- Reference games: pig, scopa, backgammon, pusoy dos, crazy eights, farkle, yahtzee are all mature code. Choose one or two when building a new game; they are  small and can fit in your context window, so explore agents are unnecessary.

## Testing
- If you have updated code, find or use remembered relevant test files.
- CLI simulation first — does the game complete with bots? Does output make sense to a screen reader user? Remember, if the CLI doesn't work, something is very wrong, and that takes priority.
- Test with serialization (--serialize flag) to catch persistence bugs.
- Write unit tests, play tests, and persistence tests. See existing test files for patterns.

## Things That Have Bitten Us
- TeamManager: if your game uses the global score system, you MUST initialize and update TeamManager, even in individual mode. If your game tracks scores on player objects instead, you MUST override _action_check_scores, _action_check_scores_detailed, _is_check_scores_enabled, and _is_check_scores_detailed_enabled — otherwise the S key will always say "no scores available."
- Menu focus bug: when turn-specific actions appear/disappear, persistent actions shift position and the cursor gets stuck. Fix: consider whether those actions are actually useful in the turn menu or could live in the context menu. If they are indeed useful in the turn menu, jump focus up top at the start of user turns.
- spinning: this is when agents use fallback approaches to get things working ASAP. Python is often used for throwaway scripts, but such practices are very harmful for a large codebase; worse still, any fallback approaches may "infect" other agents working on code, progressively rotting the codebase.

---
> Source: [XGDevGroup/PlayPalace11](https://github.com/XGDevGroup/PlayPalace11) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
