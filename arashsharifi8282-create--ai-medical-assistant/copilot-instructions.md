## ai-medical-assistant

> High-signal guidance for working in this repo. New sessions: read before making changes.

# AI Chatbot — Agent Quickstart

High-signal guidance for working in this repo. New sessions: read before making changes.

## Environment

- Windows-local development; Python 3.12+; run everything from the project root.
- A project virtual environment exists at `.venv\`. Always use `.venv\Scripts\python.exe`; never silently fall back to system Python when `.venv` exists.
- Windows-first: all scripts are `.bat`; path separators are `\`.

## Commands

| Action | Command |
|---|---|
| Run all tests | `.venv\Scripts\python.exe -m pytest -q` |
| Run a single test file | `.venv\Scripts\python.exe -m pytest -q tests/<filename>.py` |
| Lint (ruff) | `.venv\Scripts\python.exe -m ruff check .` |
| Auto-fix (ruff) | `.venv\Scripts\python.exe -m ruff check . --select E,F,I,NB,B,UP,ASYNC --fix` |
| Start the bot | `start.bat` (recommended) or `.venv\Scripts\python.exe -m app.main` |
| Health check | `.venv\Scripts\python.exe scripts\health_check.py` |
| Run backup | `.venv\Scripts\python.exe scripts\backup.py` |

`scripts\health_check.py --live-references` and `scripts\test_references_live.py` make REAL network requests to pharmaceutical APIs — never run them unless the user explicitly asks for live testing.

## Security (CRITICAL)

- Never read `.env`; never print `.env`; never copy values from it. Use `.env.example` when configuration structure is needed.
- Never expose credentials in chat, reports, source, or logs. If old logs/transcripts (e.g. `session-*.md`) ever contain secrets, do not reproduce them.
- Never commit secret files; stage explicit paths only (`git add <path>`, never `git add -A`).
- Never push to a remote repository without explicit user instruction.
- `.env` is git-ignored; verify with `git check-ignore .env` if unsure.

## Configuration

- Copy `.env.example` to `.env` and fill in the 5 required values: `TELEGRAM_BOT_TOKEN`, `AUTHORIZED_TELEGRAM_USER_ID`, `AI_BASE_URL`, `AI_API_KEY`, `AI_MODEL`.
- `ai_base_url` must start with `http://` or `https://` (validated at runtime).
- `pharma_reference_mode` allows: `off`, `auto`, `always`.

## Python verification

Preferred verification commands (run only those relevant to the change):

```
.venv\Scripts\python.exe -m pytest -q
.venv\Scripts\python.exe -m ruff check .
.venv\Scripts\python.exe -m ruff format --check app tests
.venv\Scripts\python.exe -m compileall app
```

## Development behavior

- Inspect relevant code before editing; prefer minimal patches.
- Preserve the existing architecture unless a concrete redesign is requested.
- Reproduce bugs before fixing them; use targeted tests during debugging; run the full suite before claiming completion.
- Never weaken valid tests to get green results; never hide failures with skip/xfail without explicit justification.
- Preserve asyncio cancellation semantics — do not swallow `asyncio.CancelledError`.
- Avoid unrelated refactors; distinguish verified defects from optional improvements.

## External APIs

Live calls to PubMed, DailyMed, PubChem, or openFDA must NOT be executed unless the user explicitly asks for live testing. Offline mocked tests are allowed (tests use `httpx.MockTransport` / fake clients; reference network policy is unit-tested without live calls).

## Pharmaceutical reference layer

- `app/references/` implements a router plus four trusted adapters: PubMed/NCBI, DailyMed, PubChem, openFDA.
- Routing (`ReferenceRouter`): excludes coding/math questions, normalizes Persian/Arabic text, resolves entities via `app/references/data/pharma_aliases.json`, and requires a known pharmaceutical entity before retrieval. Intent keywords alone never trigger retrieval.
- Mode `off` disables retrieval; `auto` routes on entity + intent; `always` falls back to PubMed for general pharma keywords.
- Rules:
  - Unknown Persian entities must NOT be translated by guessing — return no retrieval rather than inventing a translation.
  - Aliases must be medically/linguistically reliable; treat the alias table as curated data.
  - PubChem names must be URL-encoded safely (`quote(..., safe="")`).
  - Preserve provider-specific query behavior: PubMed uses `AND`-joined terms, openFDA uses `patient.drug.openfda.generic_name:"..."`, DailyMed uses compact drug-name queries.
  - Citations must remain traceable: `[S#]` markers are validated and the bibliography is built only from actually retrieved items.
  - FAERS/openFDA spontaneous reports describe reported events, not proof of causality; adverse-event items carry a disclaimer.
- The reference layer is optional and never blocks chat: failures land in `ReferenceResult.errors`.

## Package boundaries

- `app/` — the main Python package (entry point: `app.main`).
- `tests/` — unit/integration tests; all fixtures use an isolated temp DB.
- `scripts/` — utility scripts (backup, health_check); not auto-run.
- `data/`, `backups/`, `temp/`, `logs/` — runtime directories; git-ignored (only `.gitkeep` tracked).
- `prompts/` — system prompt files; editable without code changes.
- `.opencode/` — OpenCode agents, commands, skills, and permission policy (see `.opencode/README.md`).

## Testing quirks

- Tests mock all external AI and Telegram calls — no real API is contacted.
- Each test gets an isolated temp DB (`tmp_path` fixture). Do not persist test state.
- conftest.py sets up `Database`, `SessionRepository`, `MessageRepository`, `ReminderRepository` per test session.
- Run tests with `.venv\Scripts\python.exe -m pytest -q` from the project root.

## Key directories & runtime paths

| Dir | Purpose |
|---|---|
| `data/assistant.db` | SQLite DB (path configurable via `DATABASE_PATH` in `.env`) |
| `backups/` | Timestamped DB backups; retention controlled by `BACKUP_RETENTION_COUNT` |
| `temp/` | Per-request temp files (cleared between requests) |
| `logs/` | App logs; level set by `LOG_LEVEL` in `.env` |
| `prompts/system_prompt.md` | System prompt; loaded on startup |

## Style / conventions

- Ruff configured in `pyproject.toml`: `line-length = 100`, target `py312`, ignore `B008`.
- `select = ["E", "F", "I", "UP", "B", "ASYNC"]` — errors/warnings, import organization, bare except, unused, async.
- `asyncio_mode = "auto"` in pytest config.

## What to avoid

- Do not run `python -m pytest` without `.venv` — system Python may have incompatible dependency versions.
- Do not `git add .` or `git add -A` — stage explicit file paths only.
- Do not remove `.ruff_cache/` or `.venv/` — they are local developer artifacts.
- Do not run live reference scripts unless the user explicitly asks.

---
> Source: [arashsharifi8282-create/AI-medical-assistant](https://github.com/arashsharifi8282-create/AI-medical-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
