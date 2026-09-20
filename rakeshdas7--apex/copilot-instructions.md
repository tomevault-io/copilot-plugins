## apex

> - Active Stage: Read `plan/CURRENT_STAGE.md` for default focus.

# APEX AGENT PROTOCOL

## 1. STAGE FOCUS & RETROACTIVE FLEXIBILITY
- Active Stage: Read `plan/CURRENT_STAGE.md` for default focus.
- Open active stage spec (`plan/stages/stage-*.md`).
- Retroactive Edits: CAN modify/refactor code, schemas, configs, or tests from ANY previous stage at any point whenever needed or requested.
- No Unprompted Leapfrogging: Do not implement future unstarted stages unless explicitly asked.

## 2. STAGE PROGRESSION (EXPLICIT USER COMMAND ONLY)
- Never auto-advance stage on your own.
- When active stage tasks complete: notify user, report status, and AWAIT explicit user command to advance.
- Advance `plan/CURRENT_STAGE.md` ONLY when user explicitly commands (e.g. "proceed to stage 2", "advance stage").

## 3. CHECKLIST & VERIFICATION
- Work atomic: 1 item `- [ ]` at a time.
- Commands: ALWAYS prefer `make` targets (`make test`, `make verify-stage-0`, `make verify-stage-1`, `make clean`).
- Run verify command. Exit code 0 REQUIRED before marking `- [x]`.
- Cross-stage regression: If editing code from previous stages, run `make test` to ensure zero regressions.

## 4. ARCHITECTURE CONTRACTS
- Pipeline: Source -> Collector -> FareObservation -> Normalizer -> PostgreSQL -> IndexEngine -> API.
- Scraper: Emit `FareObservation` only. Zero index math knowledge.
- Engine: Read clean DB only. Zero scraper/HTTP knowledge.
- Provenance: Store raw payload + SHA-256 hash. Never mutate raw.
- Offline-first: 100% testable via `tests/fixtures/`. Zero live network in unit tests.
- Safety: Rate limits, backoff, circuit breaker.

## 5. ENVIRONMENT (ZERO UNAPPROVED INSTALLS)
- NO `pip install`, NO `brew install` without user command.
- Scope: Any pip install strictly in `.venv` (local project), never global.
- Stack: Python 3.11+, local PostgreSQL, Playwright async, FastAPI, Pydantic v2.
- Tooling: `make` drives all testing and execution via `.venv/bin/python`.

## 6. CODEBASE INDEX PROTOCOL (`CODEBASE_INDEX.md`)
- Scope: Codebase structure ONLY (files, symbols, tables, fixtures). ZERO plan/roadmap data.
- READ: Before writing code -> check existing classes/helpers. Prevent duplication.
- WRITE: Auto-update on new module, public class/function, DB model, or fixture.
- DO NOT WRITE: Internal logic edits, bug fixes, comments. Max 1 line per entry.

## 7. TURN EXECUTION LOOP
1. Check user prompt:
   - If user requests ad-hoc change / refactor (even for earlier stages) -> execute, test (`make test`), update index if symbols changed.
   - If user requests working on active stage -> check `plan/CURRENT_STAGE.md`, read active spec, do 1 item `- [ ]`.
   - If user explicitly commands stage progression -> update `plan/CURRENT_STAGE.md` and `plan/ROADMAP.md`.
2. Run `make` verify command. Exit 0 required.
3. Update `CODEBASE_INDEX.md` if new symbols/modules added.

---
> Source: [RakeshDas7/APEX](https://github.com/RakeshDas7/APEX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
