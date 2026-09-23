## 00-project-source-of-truth

> Canonical project rules, skills, workflows, and MCP live under .cursor/ and packages/


# Project source of truth — Work Agent

When working in **this repository**, treat the project `.cursor/` tree and documented package paths as authoritative. Do not default to global `~/.cursor/` or SDD plugin copies unless the user asks to sync or promote changes globally.

## Draft vs production

| Concern | Draft (authoring) | Production (runtime) |
| --- | --- | --- |
| Rules | `agent/define-rules/` | `.cursor/rules/*.mdc` |
| Skills | `agent/define-skills/` | `.cursor/skills/<name>/SKILL.md` |
| Workflows | `agent/define-workflows/` | `.cursor/workflows/` |
| Knowledge | — | `./knowledge/` |

Promote drafts into production before expecting Cursor or MCP to treat them as runtime assets.

## Runtime paths (Cursor)

| Asset | Canonical path in this repo |
| --- | --- |
| Project rules | `.cursor/rules/*.mdc` |
| Project skills | `.cursor/skills/<skill-name>/SKILL.md` |
| Project workflows | `.cursor/workflows/` (YAML) |
| Knowledge | `./knowledge/` |
| MCP registration | `.cursor/mcp.json` (server name **work-agent**) |
| MCP implementation | `packages/mcp-server/` → shared logic in `packages/agent-core/` |
| MCP operator docs | `.cursor/MCP.md` |
| Architecture | `agent/architecture.md` |
| Task notes (samectx) | `samectx-notes/` |
| ADRs | `./adr/` |
| Active IDE | `agent/ide-target` → `IDE_DIR` (`.cursor`, `.claude`, …) — see [`agent/IDE-ONBOARD.md`](../../agent/IDE-ONBOARD.md) |

Use **lowercase** `rules`, `skills`, and `workflows` under `.cursor/` for case-sensitive filesystems and [Cursor docs](https://cursor.com/docs/rules).

## MCP in Agent mode

For knowledge search/read/write and workflow list/run, prefer **work-agent** MCP tools when the server is built and enabled. Tool contracts must match `agent/architecture.md` and `packages/mcp-server/README.md`.

Do not implement duplicate knowledge or workflow logic only in prompts; use MCP + agent-core.

## Precedence in this repo

1. **Project rules** in `.cursor/rules/` (including this file and **`99-task-completion-gate.mdc`** for closing major tasks)
2. **Project skills** in `.cursor/skills/` — read full `SKILL.md` when a workflow applies
3. **MCP tools** for `./knowledge` and `.cursor/workflows/` side effects
4. User global rules/skills and plugins — only when this project has no equivalent or the user requests global behavior

When adding or changing agent behavior for Work Agent, **edit production paths** (`.cursor/`, `./knowledge`, packages) and update drafts in `agent/define-*` when you use them for authoring.

## Rule index

| File | Role |
| --- | --- |
| `00-project-source-of-truth.mdc` | This map; draft vs production; MCP |
| `continuous-learning.mdc` | Done gate, samectx sync, retrospective |
| `99-task-completion-gate.mdc` | Mandatory major-task checklist; research yes ≠ task done |
| `persistent-proactive-developer.mdc` | Finish work, real integrations, proactive next steps |
| `writing-style.mdc` | Tone and structure for replies and docs |

## Skill index (project)

| Skill folder | Typical use in Work Agent |
| --- | --- |
| `agent-builder` | Design and build the personalized work agent |
| `research-ops` | Evidence-first market and discovery research |
| `knowledge-ops` | Manage durable knowledge artifacts |
| `retrospective` | ADRs and knowledge after major tasks |
| `samectx` | Session context sync to `samectx-notes/` |
| `rule-creator` / `skill-creator` / `skill-lookup` | Maintain rules and skills in `.cursor/` |
| `Fullstack` / `Frontend Design` / `rag-implementation` | When implementation stories need them |

## Installing new skills in this project

1. Draft under `agent/define-skills/` if needed, then create `.cursor/skills/<name>/SKILL.md` with valid frontmatter (`name`, `description`).
2. Confirm the skill appears under **Customize → Skills** for this workspace.

Do not install Work Agent–specific skills only under `~/.cursor/skills/` unless the user wants them global across all projects.

---
> Source: [TechLah/AgentOne](https://github.com/TechLah/AgentOne) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
