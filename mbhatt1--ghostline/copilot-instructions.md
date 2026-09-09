## ghostline

> > _Read this before touching the codebase. Mirrors the conventions a human maintainer would expect._

# AGENTS.md — Guide for AI coding agents working on GhostLine

> _Read this before touching the codebase. Mirrors the conventions a human maintainer would expect._

## Project summary

GhostLine is a model-agnostic, LLM-fueled vishing operative for **authorized** security assessments. It places outbound PSTN calls via Twilio Media Streams, transcribes target speech with Deepgram, generates a social-engineered reply through the OpenAI Agents SDK (backed by any LLM via LiteLLM), synthesizes the reply with ElevenLabs (optionally a cloned voice), and streams µ-law audio back to Twilio.

**Ethical guardrail:** GhostLine must only be used against targets covered by a signed Rules of Engagement. Never weaken the `Guardrail`s in `src/ghostline/agent/` or the consent disclosures in `README.md`.

## Tech stack

- **Project / deps:** uv-managed Python 3.12 (`pyproject.toml`, `uv.lock`). No `requirements.txt`, no venv hand-rolling.
- **Layout:** `src/` layout. Package is `ghostline` under `src/ghostline/`.
- **LLM runtime:** OpenAI Agents SDK (`openai-agents`) with the LiteLLM adapter so the LLM provider is swappable via `LITELLM_MODEL` env var.
- **Web:** FastAPI + Uvicorn. One WS endpoint (`/twilio`) consumes Twilio Media Stream frames.
- **Async I/O:** `aiohttp` for Deepgram WSS and ElevenLabs HTTP. `aiosqlite` for persistence. **No sync I/O in the call hot path.**
- **DSP:** numpy + scipy. `audioop` is gone (deprecated in 3.13); µ-law encoding is implemented in pure numpy.
- **Telephony:** `twilio` (REST) + `pyngrok` (tunnel).
- **CLI:** Typer. Entrypoint: `ghostline` (see `[project.scripts]`).

## Common commands

```bash
uv sync --extra dev                 # install everything
uv run ruff check .                 # lint
uv run ruff format .                # format
uv run ruff check --fix .           # autofix
uv run mypy                         # strict typecheck (config in pyproject)
uv run pytest                       # all tests (unit + integration)
uv run pytest tests/unit            # just unit
uv run pytest -m "not live"         # skip live PSTN/API tests
uv run pytest -k playbook           # by name
uv run ghostline --help             # run the CLI
uv run pre-commit run --all-files   # all hooks
```

## Architecture map

```
src/ghostline/
├── __main__.py          # `ghostline` entrypoint (Typer app)
├── cli.py               # clone / serve / call / analytics commands
├── app.py               # SalesAutomationApp composition root
├── settings.py          # pydantic-settings: loads .env, validates secrets
├── taxonomy.py          # SalesStage enum, PSYCH_TRIGGERS, STAGE_TIMINGS, ...
├── playbook.py          # YAML loader + schema validation
├── playbook_runner.py   # stage execution: success_regex / goto / max_cycles / silent_until
├── persistence/
│   ├── schema.sql
│   └── repository.py    # async sqlite repo (no module-level DB_CONN)
├── audio/
│   ├── codec.py         # µ-law encode/decode (pure numpy)
│   └── ambient.py       # AmbientNoise: mix ambient into PCM
├── stt/deepgram.py      # typed DeepgramClient (aiohttp WSS, heartbeat, reconnect)
├── tts/elevenlabs.py    # VoiceService: clone + synth (aiohttp)
├── telephony/
│   ├── twilio.py        # outbound call + TwiML builders
│   ├── tunnel.py        # ngrok wrapper
│   └── media_stream.py  # Twilio Media Stream WS protocol (pump_in / pump_out)
├── agent/
│   ├── model.py         # LiteLLM model factory (model-agnostic)
│   ├── analysis.py      # structured AnalysisResult (replaces JSON hack)
│   ├── triggers.py      # psychological trigger selector (function tool)
│   ├── stage_handoffs.py # one Agent per SalesStage; handoff = stage transition
│   └── axel.py          # root Agent, dynamic instructions, persona blending
└── server/
    ├── fastapi.py       # app factory, /twilio WS, /voice, /api/stats
    └── dashboard.py     # HTML dashboard (replaces stub)
```

## Key invariants (don't break these)

1. **No secrets in source.** `keys.py` is gone. Secrets come from `.env` via `Settings()`. Never write real API keys into `keys.py`, `keys.example.py`, `.env`, or test fixtures. Use `tests/fixtures/` for fake values.
2. **No module-level side effects.** No `DB_CONN = init_db(...)` at import time. Database connections are created by `Repository.open()` and injected as dependencies.
3. **No sync I/O on the event loop.** Use `aiosqlite`, `aiohttp`, `httpx`. The only sync entrypoints are the CLI commands themselves.
4. **Type everything.** `mypy --strict` must pass. Public functions need docstrings (Google convention, enforced by ruff `D`).
5. **Backward-compatible playbooks.** Existing YAML playbooks in `playbooks/` must load unchanged. The previously-no-op fields (`success_regex`, `goto_on_success`, `goto_on_fail`, `max_cycles`, `silent_until`, `ambient_ratio`, `language_hint`) are now implemented.
6. **Model-agnostic LLM.** The persuasion layer must work with any LiteLLM-supported model. Never hard-code `openai.ChatCompletions` calls — go through the Agents SDK + `agent/model.py`.
7. **Each `SalesStage` is one Agent with a handoff.** Stage transitions are handoffs, not inline `if` ladders.

## Testing conventions

- **Unit tests** under `tests/unit/` — pure logic, no network. Mock external APIs with `respx` (HTTP) or `pytest-mock`.
- **Integration tests** under `tests/integration/` — exercise FastAPI routes with `httpx.AsyncClient` and a mocked Twilio Media Stream WebSocket.
- **Live tests** are marked `@pytest.mark.live` and skipped by default. They hit real Twilio/Deepgram/ElevenLabs and require creds in `.env`. Run with `uv run pytest -m live`.
- Async tests are auto-mode (`asyncio_mode = "auto"` in `pyproject.toml`).
- Use `pytest` fixtures in `conftest.py` for: tmp sqlite DB, fake settings, fake ElevenLabs MP3, async HTTP client.

## When adding a playbook stage

1. Add the stage name to `SalesStage` in `taxonomy.py` only if it's truly new; otherwise reuse.
2. Add a default `STAGE_PROMPTS` and `STAGE_CHECKINS` entry.
3. Add an Agent + handoff in `agent/stage_handoffs.py`.
4. Update `agent/axel.py`'s dynamic-instructions switch.
5. Add a unit test exercising the new stage's transitions and any `success_regex`/`goto_on_*` semantics.

## Pre-commit

`pre-commit install` is required before pushing. Hooks: ruff (lint+format), mypy (strict), whitespace/yaml/large-file checks.

## Things that are intentionally NOT here

- No Docker image yet (roadmap).
- No caller-ID spoofing (intentionally loud demo tool).
- No on-prem Whisper STT plugin (roadmap).
- No `goto_on_*` YAML branching tests for playbooks that don't define them — keep it simple.

---
> Source: [mbhatt1/GhostLine](https://github.com/mbhatt1/GhostLine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
