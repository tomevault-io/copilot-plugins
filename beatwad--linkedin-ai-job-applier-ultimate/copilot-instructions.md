## linkedin-ai-job-applier-ultimate

> AI-powered job application bot for LinkedIn and Indeed using Playwright browser automation, LLM-based job evaluation and form answering, and resume generation.

# AGENTS.md

## Project Overview

AI-powered job application bot for LinkedIn and Indeed using Playwright browser automation, LLM-based job evaluation and form answering, and resume generation.

Primary entry point:
- `main.py`

Core areas:
- `src/job_manager/` - authentication, search, job parsing, Easy Apply flow
- `src/llm/` - LLM integration and prompts
- `src/resume_builder/` - resume generation
- `src/pydantic_models/` - config and data validation
- `src/utils/` - browser and shared helpers
- `src/telegram/` - notifications and error reporting

## Run Commands

Always use `uv run` inside this repo:

```bash
uv run python main.py
uv run python <python_file_name>
uv run pytest tests/
```

## Development Rules

- Use the centralized logger from `config/logger_config.py`
- Catch specific exceptions, not bare `except:`
- Log useful context like job URL, selector, field name, or current step
- Continue processing other jobs when one job fails unless the failure is global
- Keep prompts centralized in `src/llm/prompts.py`
- Validate config and structured data with Pydantic models in `src/pydantic_models/`
- Keep imports ordered as standard library, third-party, then local imports

## Security And Config

- Never commit secrets or credentials
- Store secrets in `.env` and use `.env_example` as the template
- Never log passwords, API keys, or unnecessary personal information
- Validate config at system boundaries using `Secrets`, `SearchConfig`, and related Pydantic models

Important runtime flags live in `config/app_config.py`, including:
- `JOB_SITE`
- `HEADLESS_MODE`
- `TEST_MODE`
- `COLLECT_INFO_MODE`
- `MONKEY_MODE`
- `EASY_APPLY_ONLY_MODE`
- `DEBUG_MODE`
- `MAX_APPLIES_NUM`
- `MINIMUM_WAIT_TIME_SEC`

## Playwright And LinkedIn Automation

- Use browser helpers from `src/utils/browser_utils.py`
- Prefer selectors in this order: CSS, aria-label-based selectors, then XPath
- Use `pause()` from `src/utils/utils.py` for human-like delays
- Save and reuse browser session state from `browser_session/`
- Always clean up browser resources in `finally` blocks

LinkedIn-specific guidance:
- Prefer actual Easy Apply selectors like `button[aria-label*='Easy Apply']`
- Confirm that an Easy Apply dialog really appeared before assuming the click succeeded
- Treat modern LinkedIn job cards as dynamic and be ready to fall back to `data-job-id` or `data-occludable-job-id`

## Tests

- Put tests in `tests/` with `test_*.py` filenames
- Mock external systems like LinkedIn, Playwright, LLM APIs, and Telegram in unit tests
- Use `TEST_MODE=True` for integration-style validation without real submissions

Common commands:

```bash
uv run pytest
uv run pytest tests/test_authenticator.py
uv run pytest tests/test_job_manager.py
```

## Logs And Debug Artifacts

When debugging, agents should check `logs/` by default.

Primary log files:
- `logs/app.log` - general runtime/application log
- `logs/error_log.log` - error-focused log output
- `logs/internal_logger.log` - early logging before main logger setup
- `logs/llm_api_calls.yaml` - LLM request, token, and cost tracking

Additional runtime outputs:
- `data/debug/` - screenshots, HTML captures, and Playwright traces for debugging selector and interaction failures
- `data/output/` - `success.yaml`, `failed.yaml`, `skipped.yaml`, `interesting_jobs.yaml`, `skill_stat.yaml`, `last_run.yaml`, `resume_recommendations.txt`

If `DEBUG_MODE = True`, also inspect:
- `data/debug/*.png` - screenshots on selector or interaction failures
- `data/debug/*.html` - captured page HTML
- `data/debug/trace.zip` - Playwright trace, viewable in `https://trace.playwright.dev`

When a user asks about a runtime issue, inspect these locations before guessing.

## Notes For Future Agents

- This repo includes guidance in `.claude/` and `.cursor/rules/`; keep new work consistent with both
- For debugging sessions, treat workspace log files as available context when the user points to them

---
> Source: [beatwad/LinkedIn-AI-Job-Applier-Ultimate](https://github.com/beatwad/LinkedIn-AI-Job-Applier-Ultimate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
