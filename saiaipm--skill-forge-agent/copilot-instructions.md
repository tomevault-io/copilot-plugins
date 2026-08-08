## skill-forge-agent

> Read the README for what this project is. This file carries what is *not*

# Skill Forge — working context

Read the README for what this project is. This file carries what is *not*
obvious from reading the code: conventions to follow, traps already fallen
into, and where things stand.

## Commands

```bash
uv run pytest -q                       # 117 tests, all offline
uv run python -m skill_forge.preflight # check the model can drive the pipeline
uv run skill-forge "TOPIC" --hours 8   # full run, ~$0.02, 2-4 minutes
```

Runs cost real money and Serper quota. Prefer `pytest` while iterating; do a
full run only when something can genuinely only be seen end to end.

## Conventions

**Prompts live in `src/skill_forge/config/*.yaml`, not in Python.** They are the
most-edited part of the system.

**Rules that must hold are enforced in code, never asked of an agent.** This is
the project's organising principle and it was learned the hard way. A model told
"verify your links" will skip it when cut off mid-run and then invent the
resources it never searched for. Anything load-bearing belongs in `schemas.py`,
`guardrails.py`, `verify.py`, or `render.py`.

**Every guardrail names the real failure that motivated it.** When adding one,
write the comment explaining what was observed, and add a regression test whose
name states the defect. See the table in the README.

**Tests never touch the network.** HTTP is stubbed. A test that needs a live
service is a test that will fail in CI for reasons unrelated to the change.

## Traps — do not rediscover these

**Never set `output_pydantic` on a task that has a `guardrail`.** CrewAI's retry
loop calls `_export_output()` on every regenerated attempt without re-consulting
the guardrail (`task.py` ~line 1151), so a second failed attempt raises fatally
and the retry budget is silently lost. Typed objects come from
`guardrails.typed_output()` instead. Full explanation at the top of
`guardrails.py`.

**No `from __future__ import annotations` in `guardrails.py`.** CrewAI validates
a guardrail by introspecting its return annotation with `get_origin()`.
Postponed evaluation turns that into a string and Task construction fails with a
message that points nowhere near the cause.

**URLs are `str`, validated by `schemas.Url`, never `HttpUrl`.** `HttpUrl`
objects are not JSON-serialisable and blew up 120 seconds into a run, deep
inside CrewAI's output handling.

**YouTube returns HTTP 200 for invalid video IDs.** Link checking cannot catch a
fabricated watch URL; only the 11-character ID format can.

**Parallelism uses threads, not CrewAI's `async_execution`.** CrewAI rejects any
crew ending in more than one async task, and the only task that could consume
them cannot be built until they finish.

**This is an Intel Mac.** `crewai >= 1.11` requires `lancedb`, which ships no
macOS x86_64 wheels. `crewai-tools` pins `crewai==1.15.6` and so is unusable
here — hence the hand-rolled Serper tools. Pins in `pyproject.toml` are
deliberate; read the comments before changing them.

## State

Complete, tested, and public at
<https://github.com/saiaipm/Skill-Forge-Agent> (account `saiaipm`, not
`saipmvibri09` — that one owns an unrelated RAG demo).

Running on `openai:gpt-4.1-mini`. Validated end to end on Kubernetes, Git, Gen
AI for Product Managers, and Agents & Agentic Architectures, each producing a
document with zero dead links.

## Known weaknesses

**Hour estimates are model-invented.** The arithmetic is validated for internal
consistency; nothing validates it against reality. Describe them as structured
guidance, not measurement.

**Phase summaries tend to restate the phase name.** Raising the length floor did
not fix this — models pad to whatever minimum is allowed. Needs a better prompt
or a stronger model, not another validator.

**Recency filtering for video is prompt-only.** Serper returns relative dates
("2 years ago") that cannot be parsed into a reliable filter. The one design-spec
rule with no structural enforcement.

## Possible next steps

- A GitHub Actions workflow running `pytest`.
- A Streamlit front end with a bring-your-own-key form (in progress).
  `crew.generate_roadmap(request) -> RunReport` is already the right
  integration boundary. Two things to fix before the UI exists rather than
  after:

  **Keys must stop coming from the environment.** `llm.load_settings()` and
  `tools/serper.py::_api_key()` both read `os.getenv`, and environment is
  process-global. With two people using a hosted app at once, one visitor's
  key would serve the other's request. Thread keys through explicitly —
  `load_settings(api_key=...)`, `SerperWebSearchTool(api_key=...)` — before
  anything multi-user exists.

  **A run blocks for two to four minutes with no progress signal.** Timings
  are only recorded once each stage completes, so a naive `st.spinner` sits
  frozen. CrewAI exposes an event bus (`crewai.events.event_bus`) that could
  emit per-agent progress; alternatively report at stage boundaries, which is
  cruder but nearly free.
- Consolidating the two GitHub accounts, if this is meant to read as one
  portfolio.

---
> Source: [saiaipm/Skill-Forge-Agent](https://github.com/saiaipm/Skill-Forge-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
