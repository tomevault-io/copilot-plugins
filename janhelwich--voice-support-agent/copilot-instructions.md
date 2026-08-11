## voice-support-agent

> Web-based voice bot (Hausverwaltung) — FastAPI backend (`backend/`, uv, pytest) + Vue 3 frontend (`frontend/`, vitest). Spec: `docs/superpowers/specs/2026-07-26-voice-bot-design.md`.

# Voice-Support-Bot — Working Agreements

Web-based voice bot (Hausverwaltung) — FastAPI backend (`backend/`, uv, pytest) + Vue 3 frontend (`frontend/`, vitest). Spec: `docs/superpowers/specs/2026-07-26-voice-bot-design.md`.

## Coding style: program like Kent Beck

These govern all code in this repo — write and review against them:

- **Simple design, in this order:** (1) passes the tests, (2) reveals intention, (3) no duplication, (4) fewest elements. When rules conflict, the earlier one wins.
- **YAGNI.** Build only what the current task needs. No speculative parameters, hooks, abstractions, or "might need later" code.
- **KISS.** The dumbest thing that works beats the clever thing. Straight-line code beats indirection until duplication forces the abstraction.
- **Same level of abstraction** within a function: a function either orchestrates named steps or does low-level work — never both. Extract until each reads as a single level.
- **Small, intention-revealing units.** Names say what, not how. A reader should understand a function without its callees.
- **TDD rhythm:** red → green → refactor. Make it work, then make it right (tidy while green), then (only if measured) make it fast.
- **Tidy first:** do structural changes (renames, extractions, moves) before the behavioral change that needs them — but keep them in the same commit. Do not split a piece of work into structural and behavioral commits.
- **Tests test behavior**, not implementation details. Pristine output — no warnings.

## Conventions

- Backend: run everything from `backend/` via `uv run …`; uv only, never pip. Async throughout; audio is PCM16 mono 24 kHz everywhere.
- Frontend: `frontend/` via npm; framework-free TS modules for audio/WS, Vue renders state.
- Tests are offline — never require `OPENAI_API_KEY` or the network.
- German user-facing copy by default.
- WebSocket event names exactly as in the spec §5.

---
> Source: [janhelwich/voice-support-agent](https://github.com/janhelwich/voice-support-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
