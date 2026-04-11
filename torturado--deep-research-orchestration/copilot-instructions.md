## deep-research-orchestration

> <!-- tags: ai-assistants, onboarding, architecture, workflows, testing -->

# AGENTS.md

<!-- tags: ai-assistants, onboarding, architecture, workflows, testing -->

AI coding guide for `orchestraciondeepresearch`.

## Table Of Contents

- [Project Scope](#project-scope): what is first-party code and what is only local reference material.
- [Architecture](#architecture): how the orchestration loop is split across modules.
- [Directory Guide](#directory-guide): where to make changes for each concern.
- [Runtime Contracts](#runtime-contracts): CLI, env vars, workspace files, and output paths.
- [Development Patterns](#development-patterns): conventions to preserve when editing code.
- [Testing](#testing): the current verification commands and what each test file covers.
- [Documentation Map](#documentation-map): where to look in `.sop/summary/` for deeper context.

## Project Scope

<!-- tags: scope, boundaries, first-party, references -->

The maintained product is the Python deep research orchestrator formed by [`main.py`](/home/klepto/programacion/orchestraciondeepresearch/main.py), [`deepresearch/`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch), and [`tests/`](/home/klepto/programacion/orchestraciondeepresearch/tests).

Two large directories exist in the repo root as local references:

- [`DeepResearchAgent/`](/home/klepto/programacion/orchestraciondeepresearch/DeepResearchAgent)
- [`designing-multiagent-systems/`](/home/klepto/programacion/orchestraciondeepresearch/designing-multiagent-systems)

Treat them as inspiration/reference sources unless a task explicitly says to edit them. The first-party runtime does not import from them.

## Architecture

<!-- tags: architecture, control-loop, orchestration -->

The control loop is deterministic-first:

1. `Planner` creates a typed research plan.
2. `verify_plan()` validates task ids, dependencies, budgets, tools, servers, and depth.
3. `TaskQueue` and `Scheduler` decide what can run next.
4. `ResearchExecutor` runs explicit `search -> fetch -> summarize`.
5. `RunWorkspace` persists artifacts, summaries, and findings.
6. `PolicyEngine` decides whether to continue, replan, force completion, or stop.
7. `Synthesizer` writes/expands/normalizes the final report.
8. `Renderer` converts report markdown to editorial HTML and optional PDF.

The orchestration hub is [`deepresearch/executor.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/executor.py). Most behavior changes eventually pass through that file.

## Directory Guide

<!-- tags: directories, ownership, code-map -->

- [`deepresearch/config.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/config.py): runtime defaults, env loading, CLI flags.
- [`deepresearch/models.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/models.py): source of truth for plan/task/finding/artifact schemas.
- [`deepresearch/planner.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/planner.py): query classification, JSON plan parsing, heuristic plans, replan repair tasks.
- [`deepresearch/verifier.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/verifier.py): deterministic validation.
- [`deepresearch/queue.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/queue.py): status transitions and dependency readiness.
- [`deepresearch/scheduler.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/scheduler.py): bounded breadth/depth selection logic.
- [`deepresearch/mcp.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/mcp.py): Gemini client creation and SearXNG MCP discovery.
- [`deepresearch/memory.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/memory.py): workspace persistence contract.
- [`deepresearch/policy.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/policy.py): small control-state machine.
- [`deepresearch/synthesis.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/synthesis.py): report writing, expansion, normalization.
- [`deepresearch/render.py`](/home/klepto/programacion/orchestraciondeepresearch/deepresearch/render.py): editorial HTML/PDF generation.
- [`tests/`](/home/klepto/programacion/orchestraciondeepresearch/tests): regression coverage by subsystem.

## Runtime Contracts

<!-- tags: cli, env, workspace, outputs -->

CLI:

```bash
python main.py
python main.py "consulta"
python main.py --resume <run_id>
python main.py --max-parallel <n>
python main.py --target-words <n>
python main.py --no-pdf
python main.py --debug
```

Important env vars:

- `GEMINI_API_KEY`
- `SEARXNG_STDIO_ENTRYPOINT`
- `SEARXNG_BASE_URL`
- `TARGET_REPORT_WORDS`
- `WORKSPACE_ROOT`
- `REPORTS_ROOT`

Workspace layout per run:

- `workspace/<run_id>/plan.json`
- `workspace/<run_id>/queue.json`
- `workspace/<run_id>/findings.jsonl`
- `workspace/<run_id>/run_log.jsonl`
- `workspace/<run_id>/artifacts/`
- `workspace/<run_id>/branch_summaries/`
- `workspace/<run_id>/global_summary.md`
- `workspace/<run_id>/final_report.md`
- `workspace/<run_id>/final_report.html`
- `workspace/<run_id>/final_report.pdf`

`reports/` mirrors final outputs for easier browsing. The canonical run state lives in `workspace/`.

## Development Patterns

<!-- tags: patterns, editing, conventions -->

- Keep orchestration deterministic where possible; do not hide critical control flow inside prompts.
- When changing schemas in `models.py`, also update planner, verifier, memory, executor, and tests.
- Preserve the explicit `search -> fetch -> summarize` execution contract.
- Prefer artifact-backed context over transcript-backed context.
- Keep replan behavior bounded and repair-oriented.
- Preserve the no-shrink guard in final report normalization.
- Treat vendored upstream directories as read-only reference material unless the task says otherwise.

## Testing

<!-- tags: tests, verification -->

Primary commands:

```bash
python -m unittest discover -s tests -v
python -m py_compile main.py deepresearch/*.py tests/*.py
```

Current test coverage:

- [`tests/test_planner_verifier.py`](/home/klepto/programacion/orchestraciondeepresearch/tests/test_planner_verifier.py): planner parsing and deterministic validation.
- [`tests/test_queue_scheduler_policy.py`](/home/klepto/programacion/orchestraciondeepresearch/tests/test_queue_scheduler_policy.py): queue, scheduler, policy.
- [`tests/test_memory_render_synthesis.py`](/home/klepto/programacion/orchestraciondeepresearch/tests/test_memory_render_synthesis.py): persistence, renderer, report normalization.
- [`tests/test_executor.py`](/home/klepto/programacion/orchestraciondeepresearch/tests/test_executor.py): tool ordering.
- [`tests/test_integration.py`](/home/klepto/programacion/orchestraciondeepresearch/tests/test_integration.py): heuristic planning and fake-MCP end-to-end run.

Gap to remember:

- There is no automated live integration test against the real Gemini + SearXNG MCP stack.

## Documentation Map

<!-- tags: docs, summary, references -->

Use [`index.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/index.md) as the primary documentation entrypoint.

Deep links:

- [`architecture.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/architecture.md)
- [`components.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/components.md)
- [`interfaces.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/interfaces.md)
- [`data_models.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/data_models.md)
- [`workflows.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/workflows.md)
- [`dependencies.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/dependencies.md)
- [`review_notes.md`](/home/klepto/programacion/orchestraciondeepresearch/.sop/summary/review_notes.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/torturado)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/torturado)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
