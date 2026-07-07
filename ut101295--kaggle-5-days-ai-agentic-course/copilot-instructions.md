## kaggle-5-days-ai-agentic-course

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal course notes + code for the **Kaggle 5-Day AI Agents Intensive** (with Google), built on Google's **Agent Development Kit (ADK)** and the Gemini API. Each `dayN/` folder is a self-contained lesson: prose notes in `dayN/README.md`, code in `dayN/src/`, tests in `dayN/tests/`. Day 1 is implemented; days 2–5 are scaffolded (`.keep` placeholders) and unlock progressively.

## Commands

```bash
# One-time setup
python3 -m venv .venv && source .venv/bin/activate
pip install -e .                       # installs deps from pyproject.toml (incl. google-adk)
cp .env.example .env                   # then set GOOGLE_API_KEY (from aistudio.google.com/apikey)
pre-commit install                     # enable format/lint on commit

pytest                                 # all days
pytest day1/tests                      # one day
pytest day1/tests/test_agent_intro.py::test_greet_agent   # single test

python day1/src/agent_intro.py         # run a day's agent (hits the Gemini API)

black . && isort . && ruff check --fix .   # manual format/lint (line length 100)
```

## Architecture & conventions

- **Per-day pythonpath, not a package.** `pyproject.toml` adds every `dayN/src` to both pytest `pythonpath` and `testpaths`, and `[tool.setuptools.packages.find]` *excludes* the day folders. Consequence: tests import modules by bare name (e.g. `from agent_intro import greet_agent`), **not** via a package path — there is no `dayN.src.foo` import. New code in a day's `src/` is importable from that day's `tests/` by module name automatically. `ruff.src` is likewise the list of day `src/` dirs.

- **When adding a new day's work,** put runnable code in `dayN/src/`, tests in `dayN/tests/`, and update `pyproject.toml`'s `testpaths`/`pythonpath`/`ruff.src` if you add a day beyond 1–5. Keep module names unique across days since they share a flat import namespace under pytest.

- **ADK agent pattern (see `day1/src/agent_intro.py`).** An agent = `Agent(name, model=Gemini(...), instruction=..., tools=[...])` driven by an `InMemoryRunner`. Async entry via `asyncio.run(main())`, where `main` awaits `runner.run_debug(prompt)`. Model calls wrap `types.HttpRetryOptions` (exponential backoff on 429/500/503/504) — reuse this resilience pattern for new agents.

- **Secrets.** `load_dotenv()` reads `.env`; code expects `GOOGLE_API_KEY` (Gemini). `.env.example` also lists `MCP_SERVER_URL` and `A2A_ENDPOINT` for later days (MCP tools, Agent2Agent). `.env` is gitignored.

- **Docs are a first-class deliverable.** Every day's `README.md` follows a fixed shape (Overview → Key concepts → Code walkthrough → Setup & run → Takeaways → Resources with 📄 whitepaper / 🧪 codelab / 🎧 podcast / 📺 livestream). Keep code examples in the READMEs in sync with `src/`, and update the progress checklist + course map in the root `README.md` when a day is completed.

---
> Source: [ut101295/Kaggle-5-Days-AI-Agentic-Course](https://github.com/ut101295/Kaggle-5-Days-AI-Agentic-Course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
