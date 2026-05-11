## dspy-micro-agent

> This repository implements a minimal agent runtime on top of DSPy. It is intentionally small and opinionated: use DSPy modules for planning/tool-calls and a thin Python loop for orchestration, tracing, and evaluation.

# AGENTS.md — Guidance for agents working in this repo

This repository implements a minimal agent runtime on top of DSPy. It is intentionally small and opinionated: use DSPy modules for planning/tool-calls and a thin Python loop for orchestration, tracing, and evaluation.

What to preserve
- Keep the runtime loop thin (see `micro_agent/agent.py`). Avoid adding heavyweight frameworks.
- Respect provider modes:
  - OpenAI → native tool-calls via DSPy `PlanWithTools` + `JSONAdapter`.
  - Others (e.g., Ollama) → robust JSON decision loop with few-shot demos and JSON repair.
- Keep observability simple and durable: append JSONL records under `TRACES_DIR` (default `traces/`).
- Do not remove the JSON repair/lenient parsing fallback — models can drift.

Policy and behavior (planning/acting)
- Tools are required when the question implies:
  - math → must use `calculator` before finalize
  - time/date → must use `now` before finalize
- Violations are recorded as steps (`tool: "⛔️policy_violation"`) and the loop continues.
- Tool args are validated against JSON Schema before execution; invalid inputs add a `⛔️validation_error` step. The loop then requests a corrected call next iteration.
- One tool per step. Favor strict JSON decisions.
- For OpenAI, the agent composes final answers from tool results (calculator/now) when available to preserve key values.

Provider configuration
- Environment variables:
  - `LLM_PROVIDER`: `openai` | `ollama` | `mock` (default: `openai`)
  - OpenAI: `OPENAI_API_KEY`, `OPENAI_MODEL` (default `gpt-4o-mini`)
  - Ollama: `OLLAMA_HOST` (default `http://localhost:11434`), `OLLAMA_MODEL`
  - Optimization demos: `COMPILED_DEMOS_PATH` (default `opt/plan_demos.json`)
  - Costs (OpenAI): `OPENAI_INPUT_PRICE_PER_1K`, `OPENAI_OUTPUT_PRICE_PER_1K` (defaults provided for 4o/4o-mini/4.1)
- `configure_lm()` turns on `track_usage` and falls back in order: Ollama → OpenAI → mock LM.

Tools and safety
- Tools are defined in `micro_agent/tools.py` with a small registry and JSON Schemas.
- Runtime arg validation uses `jsonschema`. Unknown/malformed inputs do not crash the loop; they are surfaced in the trace for self-correction.
- Calculator constraints: factorial ≤ 12, exponent bound, AST node-count limit, and number magnitude cap. The operator set is allow‑listed.
- Plugin loader: set `TOOLS_MODULES="pkg1.tools,pkg2.tools"` to merge additional tool dicts.

Tracing and API
- Every ask appends a record to `traces/<id>.jsonl`: `{id, ts, question, steps, answer}`.
- Steps are `{tool, args, observation}`.
- CLI: `micro-agent replay --path traces/<id>.jsonl --index -1`.
- HTTP: `POST /ask` and `GET /trace/{id}` (CORS enabled) via `micro_agent/server.py`.

Optimization (teleprompting)
- `micro-agent optimize` compiles a few-shot set for the OpenAI planner using DSPy `BootstrapFewShot`:
  - Strict metric for this repo’s sample tasks: finals must contain the expected substring for math; time tasks must call `now`.
  - Saves demos to JSON; the agent auto-loads them if present.

Evals
- `evals/run_evals.py` runs a small dataset and prints:
  - `success_rate`, `contains_hit_rate`, `key_hit_rate`, `avg_latency_sec`, `avg_lm_calls`, `avg_tool_calls`, `avg_steps`, `avg_cost_usd`, `n`.
- Scoring combines substring hits and key-in-observation hits per `evals/rubrics.yaml`.

Directory map (key files)
- `micro_agent/agent.py` — runtime loop, provider branches, policy, usage aggregation.
- `micro_agent/signatures.py` — DSPy signatures (`PlanWithTools`, etc.).
- `micro_agent/tools.py` — tool registry, validation, safe calculator, plugin loader, dspy.Tool bridge.
- `micro_agent/runtime.py` — tracing & robust JSON extraction (json_repair fallback).
- `micro_agent/config.py` — LM configuration and fallbacks; `track_usage=True`.
- `micro_agent/server.py` — FastAPI app (`POST /ask`, `GET /trace/{id}`) with CORS.
- `micro_agent/optimize.py` — compile few‑shot demos and save for the OpenAI planner.
- `evals/run_evals.py` — metrics harness with cost/latency/usage.

Conventions
- Keep changes minimal and focused; match code style and existing patterns.
- Prefer explicit, strict JSON outputs from models; use repair/lenient paths as fallbacks.
- Avoid adding non‑deterministic behavior that would complicate traces/replay.

---
> Source: [evalops/dspy-micro-agent](https://github.com/evalops/dspy-micro-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
