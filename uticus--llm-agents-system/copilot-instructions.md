## llm-agents-system

> > Read this file first, always. Before any action, before any file, before any code.

# CLAUDE.md
# Agent Entry Point

> Read this file first, always. Before any action, before any file, before any code.
> This file is the single authoritative entry point for all agents.
> Do not duplicate content from referenced files. Link only.
>
> Language: English only in all project files, code, comments, and commits.
> No emojis. Use plain markers: [CRITICAL], [WARNING], [OK], [ERROR], [BLOCKING].

---

## What this project is

llm_agents_system is a Python platform for building and orchestrating LLM-based agent systems: model hub (OpenAI/HuggingFace/GGUF via llama.cpp/vLLM), data ingestion, RAG, agent capabilities (memory, planning, tools, hierarchy), serving, fine-tuning, and evaluation.

| Field | Value |
|---|---|
| Primary language | Python 3.12+ |
| Bindings | none |
| Build | uv (pyproject.toml); heavy integrations behind optional extras |
| Toolchain | CPython 3.12+, ruff, pytest |
| MCP tools | see `.cursor/mcp/registry.md` |

---

## Non-negotiable rules

These rules apply to every agent at every phase. No exceptions.

1. External dependencies are immutable. Never propose or make changes to fetched dependencies.
2. English only. No other languages in any project file.
3. No emojis in any file.
4. Always run tools from the project root. Never `cd` into subdirectories.
5. Use structural search tools before text search. Available MCP tools: `.cursor/mcp/registry.md`.
   Grep only for string literals and comments.
6. Capture search results after each step before running the next search.
7. Public API changes require full impact analysis: headers + tests + bindings + examples.
8. Integration tests must replicate real usage scripts exactly. Do not reorder or optimize commands.
9. Each agent reads only what its role definition specifies. No browsing unrelated files.
10. Memory writer is responsible for knowledge continuity: persisting decisions and context
    across sessions, tasks, and time. Who writes what and where is defined in
    `.cursor/pipeline/pipeline.md`.

CRITICAL: always follow pipeline to process request

---

## How to identify your role

Your role is stated in the task or session prompt.
If no role is stated — ask the developer which role to activate before doing anything else.

Available roles:

| Role | File | Activated when |
|---|---|---|
| Analyst | `.cursor/agents/analyst.md` | Developer starts a new request |
| Decomposer | `.cursor/agents/decomposer.md` | Request is ready to split into task cards |
| Architect | `.cursor/agents/architect.md` | Design phase starts for a task card |
| Planner | `.cursor/agents/planner.md` | Architecture section of task card is complete |
| Critic | `.cursor/agents/critic.md` | Planner produces a plan draft |
| Spec writer | `.cursor/agents/spec-writer.md` | Plan is approved by developer |
| Test designer | `.cursor/agents/test-designer.md` | Spec is written |
| Environment | `.cursor/agents/environment.md` | Spec and test criteria are approved |
| Impl: Python | `.cursor/agents/implementer-python.md` | Environment is ready (primary implementer — this is a Python project) |
| Impl: ML | `.cursor/agents/implementer-ml.md` | Environment is ready, task requires ML (fine-tuning, local inference, embeddings, RAG models) |
| Impl: C++ | _inactive_ — moved to `.cursor/_unused/` | Not applicable (no C++ in this project) |
| Analyst: Code | `.cursor/agents/analyst-code.md` | CP2 approved, implementer type is `code-analyst` — task reads code and produces markdown artifacts only; Phases 3–4 and Reviewer+Tester loop are skipped |
| Reviewer | `.cursor/agents/reviewer.md` | Implementation is ready for review |
| Tester | `.cursor/agents/tester.md` | Implementation passes Reviewer |
| Memory writer | `.cursor/agents/memory-writer.md` | Called by any agent or developer as needed |

---

## What to read next

After this file, read in order:

1. Your agent role file from `.cursor/agents/<role>.md`
   — it specifies exactly what to read, what to write, which skills and rules to load, and when to stop.

2. `.cursor/pipeline/pipeline.md`
   — read if you need to understand the full flow, your position in it, or handoff rules.

3. Your task card: `tasks/active/task-NNN.md`
   — read only the sections relevant to your phase (listed in your role file).

4. Memory files listed in your role definition
   — read only the files listed. Do not browse `memory/**` freely.

Do not read files not listed in your role definition.
Do not read other agents' task cards or session files.

---

## File structure (summary)

```
CLAUDE.md                        this file — read first

.cursor/
  agents/                        role definitions — one file per agent
  pipeline/
    pipeline.md                  full pipeline: phases, loops, checkpoints, agent table
    loop-design.md               rules for the Architect + Planner + Critic loop
    loop-impl.md                 rules for the Implementer + Reviewer + Tester loop
  mcp/
    registry.md                  connected MCP servers: name, purpose, when to use
    serena.md                    Serena methods, usage patterns, limitations
    <name>.md                    one file per connected MCP
  skills/                        HOW to perform a type of task (algorithm, step by step)
  rules/                         WHAT is allowed and forbidden per task type
  tasks/
    inbox/                       raw developer requests (Analyst output)
    active/                      task cards in progress — one file per direction
    sessions/                    live drafts — readable at any time by developer
    decisions/                   clean final artifacts — plans, review reports
    archive/                     completed tasks

  memory/
    status.md                    current state of all memory files — read before accessing memory
    project/
      brief.md                     what the project is and why
      domain.md                    domain constraints relevant to the project
      build.md                     build system facts: presets, toolchain, paths
    architecture/
      map.md                       module boundaries, pipelines, invariants
      inventory.md                 structural reference — all modules and files
      checklist.md                 enforcement rules with severity levels
      logic/                       per-module logic descriptions (one file per module)
      analysis/                    cross-cutting analysis reports
    decisions/
      adr-log.md                   all architectural decisions with rationale
    analysis/
      <topic>.md                   results of code analysis, research spikes
```

---

## Memory file status

Current state of all memory files (what exists, what does not):
see `.cursor/memory/status.md`

Memory writer maintains that file. Agents read it before accessing any memory file.

---

## Checkpoints — where control returns to the developer

| CP | After | Developer decides |
|---|---|---|
| CP1 | Decomposer | go / edit / send back to Analyst |
| CP2 | Planner + Critic + Architect loop | go / revise → loop |
| CP3 | Spec writer + Test designer | go / adjust |
| CP4 | Environment | go / fix env |
| CP5 | Reviewer + Tester loop (impl tasks) / Analyst: Code (code-analyst tasks) | merge / rework → loop / escalate to CP2 |

An agent must stop and surface output for developer review at every checkpoint.
An agent must not proceed past a checkpoint without explicit developer approval.

---

## Output conventions

- Always show the full file path before any code block.
- Write decisions with trade-off changes to `.cursor/memory/decisions/adr-log.md` via Memory writer.
- Write session progress to `.cursor/tasks/sessions/task-NNN-<phase>.md` continuously.
- Write final clean artifacts to `.cursor/tasks/decisions/` only when the loop stop condition is met.
- When a new file is created under the main source tree: flag it for inventory update.
- When public API changes: flag it — ABI check required.

---
> Source: [uticus/llm_agents_system](https://github.com/uticus/llm_agents_system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
