## class-production-ai-app

> Contract for any AI coding agent (Claude Code, Cursor, etc.) contributing

# AGENTS.md

Contract for any AI coding agent (Claude Code, Cursor, etc.) contributing
to this repo.

## Do

- Read [CLAUDE.md](CLAUDE.md) and [.claude/rules/](.claude/rules/) first.
- Keep each concern in its layer: retrieval in `components/`, orchestration
  in `services/`, intelligence in `agents/`, guardrails in `security/`.
- Add or update tests in [tests/](tests/) for any behavioural change.
- Register new prompts in
  [app/prompts/registry.py](app/prompts/registry.py) — never inline.
- Wire new LLM calls through [observability/tracer.py](observability/tracer.py)
  and [observability/cost_tracker.py](observability/cost_tracker.py).

## Don't

- Don't edit `evaluation/golden_dataset.json` to make a test pass.
- Don't bypass the security layers for "internal" endpoints.
- Don't introduce a new top-level directory without updating
  [README.md](README.md) and this file.
- Don't commit secrets, model keys, or data/raw contents.

## Review checklist

- [ ] New prompts are versioned in the registry
- [ ] New LLM calls are traced and costed
- [ ] Output passes through `output_filter`
- [ ] Tests added / updated and passing
- [ ] Docs updated if the contract changed

---
> Source: [fdhhhdjd/Class-Production-AI-App](https://github.com/fdhhhdjd/Class-Production-AI-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
