## slow-ai

> The only file auto-loaded every session. Kept intentionally short: **identity, rules, and a doc index**. Deep detail lives in `docs/` and is fetched on demand.

# Slow AI — Claude Code context

The only file auto-loaded every session. Kept intentionally short: **identity, rules, and a doc index**. Deep detail lives in `docs/` and is fetched on demand.

## Session harness

**At the start of every session, run `/resume`** — it loads `SESSION.md` (current focus + next step), `FEATURES.md` (roadmap), and reconciles against `git status`.

- `SESSION.md` — current session state. First source of truth for "what are we working on right now?"
- `FEATURES.md` — feature roadmap (done / in-progress / planned / blocked / dropped). Change via `/feature`.
- `sessions/` — archive of past `SESSION.md` snapshots (created by `/handoff`).
- `/handoff` — run at end of session to archive current state and write a fresh SESSION.md.

**Keep SESSION.md updated as you work** — not only at handoff. When focus shifts, when a decision is made, when a file gets touched, reflect it in SESSION.md immediately. The next session should be able to pick up mid-thread without archaeology.

Do not start implementation work without reading `SESSION.md` and confirming direction with the user.

---

## What Slow AI is

An agentic research orchestration system. Multi-agent investigations run in parallel, phase by phase, on your local infrastructure. Everything is file-based and git-committed. No external platforms. No shared state you don't own.

**The name is the thesis: provenance over pace.** Deliberate, inspectable, reproducible work — the opposite of black-box hosted agents. Every agent decision, every path taken or skipped, every piece of evidence is recorded, versioned, and auditable. Domain-agnostic by design: same system runs data research, due diligence, competitive analysis, literature surveys. Built on distributed-systems primitives (blast radius, circuit breakers, evidence envelopes instead of verdicts, idempotency, MAPE-K). Human-in-the-loop is a first-class primitive, not an exception handler.

---

## Architecture in one page

### Two independent planes

```
  ┌─────────────────────────────────┐
  │  UI  (FastAPI + htmx)           │  reads files, renders state over SSE
  └──────────────┬──────────────────┘
                 │  files on disk only — no direct coupling
  ┌──────────────▼──────────────────┐
  │  Execution engine  (subprocess) │  writes files, commits to git
  │  plan → gate → waves → report   │
  └──────────────┬──────────────────┘
                 │
               git  (one branch per run · milestone commits)
```

The UI never calls the execution engine directly. It launches it as a subprocess (`python -m slow_ai.research`) and reads the files it writes. This isolation is intentional — threading caused asyncio event loop conflicts. **Do not suggest threading as an alternative.**

### Execution flow

```
Interview  →  ProblemBrief  →  ContextGraph  →  Viability check
  →  Skill synthesis (if gaps)  →  Wave loop:
       Phase N: [specialist-1 ‖ specialist-2 ‖ ...] → orchestrator assesses
         → synthesise  →  proceed / circuit_break / escalate
  →  ResearchReport  →  git commit
```

The `ContextGraph` is the approved work plan. `Phase` objects are sequential; `WorkItem` objects within a phase run in parallel. Every specialist agent maps to exactly one work item and only sees its own item + the phase's prior evidence.

Every specialist writes an `EvidenceEnvelope` to `runs/{run_id}/envelopes/`. Envelopes are the atomic unit of evidence — everything in the UI traces back to them.

Skills live in `src/slow_ai/skills/catalog/{skill_name}/SKILL.md` (YAML frontmatter + playbook body). Built-in: `web_search`, `web_browse`, `pdf_extraction`, `dataset_inspection`, `code_execution`. **Always use catalog names exactly — do not invent synonyms.** The synthesizer creates new SKILL.md files when catalog gaps are found.

---

## Key design decisions

| Decision | Rationale |
|---|---|
| Subprocess isolation for runner | Threading causes asyncio event-loop conflicts with nest_asyncio + anyio. Subprocess is the permanent fix. |
| File-based state (no DB) | Any reader (UI, CLI, script) can read run state without a running service. Plain JSON. Git-versioned. |
| Evidence envelopes as the atom | Every claim traceable to an agent, confidence score, and sources. No free-text summaries that can't be verified. |
| Two-plane architecture | UI is a thin render layer. Execution plane is independent. Any UI (CLI, Jupyter) can consume the same files. |
| Skill catalog over hardcoded tools | Domain knowledge lives in SKILL.md playbooks, not agent prompts. Skills compose. |
| Git as long-term memory | One branch per run. Milestone commits. The run corpus is future RL training data. |

---

## Doc index — read on demand

Do not preload these. Read only the ones relevant to the current task.

| If you're doing this... | Read this |
|---|---|
| Changing runner internals, agents, models, envelopes | `docs/architecture.md` (source map, on-disk layout, module boundaries) |
| Understanding the execution loop step by step | `docs/how-it-works.md` |
| Setting up env, API keys, running app/tests | `docs/getting-started.md` |
| Debugging a known failure mode | `docs/known-issues.md` |
| Understanding a past architectural choice | `other_docs/design_decisions.md` |
| Deep design narrative ("why is it built this way") | `other_docs/technical_deep_dive.md` |
| Full technical reference (models, config, agents) | `other_docs/technical.md` |
| Adding a new skill | `.claude/commands/new-skill.md` |
| Roadmap / what's planned | `FEATURES.md` (canonical), `docs/whats-next.md` (public-facing) |

---

## How to work with this codebase

**Before changing anything non-trivial:** discuss architecture first. The user is the domain expert and wants to align on approach before code is written.

**Do not:**
- Suggest threading as an alternative to subprocess isolation
- Add error handling for scenarios that can't happen
- Invent new skill names — always prefer catalog names
- Add docstrings, comments, or type annotations to code you didn't change
- Create new files unless strictly necessary

**Do:**
- Read the relevant files (and the relevant doc from the index) before proposing changes
- Keep the two-plane separation clean — UI reads files, engine writes files
- Trust pydantic model validation at the boundary; don't re-validate internally
- When adding a tool, also add a SKILL.md in the catalog that uses it

---
> Source: [ai-agents-for-humans/slow-ai](https://github.com/ai-agents-for-humans/slow-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
