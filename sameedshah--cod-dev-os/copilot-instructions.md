## cod-dev-os

> **Purpose:** Reusable Claude Code template with SDD/TDD, Obsidian memory, MCP servers, skills, hooks, and sub-agents.

# cod-dev-os — AI Developer OS Template

**Purpose:** Reusable Claude Code template with SDD/TDD, Obsidian memory, MCP servers, skills, hooks, and sub-agents.

## Core References
- Constitution & principles: @.specify/memory/constitution.md
- SDD workflow rules: @.specify/templates/
- PHR template: @.specify/templates/phr-template.prompt.md
- **Skill routing (which skill to use when):** @.specify/memory/skill-router.md

## Agents
See `.claude/agents/` — invoke by name or describe what you need.

| Agent | Trigger |
|---|---|
| `project-bootstrap` | `/bootstrap <pdf>` — PDF → scaffold |
| `spec-curator` | `/sp.specify-plus` — context-aware spec |
| `qa-playwright` | After `/sp.implement` — UI test |
| `obsidian-scribe` | Session start/end — vault memory |
| `ui-auditor` | After any UI component — design check |
| `db-architect` | Schema/migration work — Supabase |
| `deploy-agent` | `/deploy` — VPS deployment |
| `error-hunter` | Debug sessions — root cause |
| `phr-adr-curator` | Automatic — PHR + ADR management |

## Slash Commands
- `/bootstrap <pdf>` — initialise project from PDF brief
- `/sp.specify-plus <feature>` — spec with prior context injected
- `/sp.plan`, `/sp.tasks`, `/sp.implement` — SDD workflow
- `/sp.adr <title>` — document architectural decision
- `/sp.phr` — create Prompt History Record

## Session Start Protocol
**BEFORE responding to any user prompt**, check for `./obsidian/session-logs/.pending-sync`.
If it exists → invoke `obsidian-scribe` in Session Start Mode immediately.
This is non-negotiable. Stale context is worse than a 10-second delay.

## Memory (Obsidian Vault)
Vault at `./obsidian/` — loaded automatically at session start via obsidian-scribe.
- `codebase-state.md` — live snapshot: file status, git log, task progress, TS health ← **read first**
- `overview.md` — project goals + open questions
- `decisions.md` — architectural decisions log
- `phr-index.md` — index of all PHRs (last 5 loaded at session start)
- `spec-history.md` — all generated specs
- `session-logs/` — daily session summaries

## Active Context (always load these)
- Codebase State: @obsidian/codebase-state.md
- PHR Index: @obsidian/phr-index.md

## Skills — Reusable Intelligence
> Before any non-trivial task, check `skill-router.md` for the applicable skill.
> Skills are invoked via the `Skill` tool. The router maps intent → skill name.

### Frontend & UI
| Skill | When to invoke |
|---|---|
| `ui-anti-ai` | Generating or auditing any UI component |
| `frontend-design` | React/Next.js component architecture |
| `refactoring-ui` | Fixing visual hierarchy, spacing, colour |
| `top-design` | Award-winning, Awwwards-level experiences |
| `ios-hig-design` | Native iOS interface design |
| `web-typography` | Font selection, pairing, implementation |
| `theme-factory` | Applying a brand theme to any artifact |
| `microinteractions` | Animations, triggers, feedback loops |
| `high-perf-browser` | Web performance, loading, rendering |

### Product & UX
| Skill | When to invoke |
|---|---|
| `inspired-product` | Product discovery, empowered teams |
| `lean-ux` | Hypothesis-driven UX design |
| `lean-startup` | MVP design, validated experiments |
| `continuous-discovery` | Weekly customer touchpoint cadence |
| `jobs-to-be-done` | Customer needs / "job" analysis |
| `design-sprint` | 5-day prototype and validate process |
| `design-everyday-things` | Affordances, signifiers, feedback |
| `ux-heuristics` | Usability evaluation (Nielsen 10) |
| `hooked-ux` | Habit-forming product loop design |
| `improve-retention` | Diagnosing and fixing retention drop |

### Marketing & Growth
| Skill | When to invoke |
|---|---|
| `obviously-awesome` | Product positioning & messaging |
| `blue-ocean-strategy` | Uncontested market space creation |
| `crossing-the-chasm` | Tech adoption lifecycle navigation |
| `contagious` | Virality, word-of-mouth engineering |
| `made-to-stick` | Sticky, memorable message crafting |
| `storybrand-messaging` | Brand narrative & hero framing |
| `one-page-marketing` | Full-funnel marketing plan |
| `predictable-revenue` | B2B outbound sales process |
| `cro-methodology` | Conversion rate optimisation |
| `scorecard-marketing` | Quiz / assessment funnels |
| `hundred-million-offers` | Irresistible offer construction |
| `influence-psychology` | Ethical persuasion principles |

### Software Engineering
| Skill | When to invoke |
|---|---|
| `clean-architecture` | Software structure, dependency rule |
| `clean-code` | Naming, functions, readability |
| `domain-driven-design` | Bounded contexts, aggregates, DDD |
| `pragmatic-programmer` | DRY, orthogonality, craftsmanship |
| `software-design-philosophy` | Complexity management, deep modules |
| `refactoring-patterns` | Named refactoring transformations |
| `system-design` | Distributed systems architecture |
| `ddia-systems` | Data storage, replication, streaming |
| `release-it` | Production stability patterns |
| `supabase-patterns` | Supabase auth, RLS, queries |

### Tooling & Automation
| Skill | When to invoke |
|---|---|
| `playwright-audit` | UI QA after any frontend implementation |
| `browsing-with-playwright` | Browser automation, web scraping |
| `obsidian-memory` | Reading/writing the project vault |
| `vps-deploy` | Hostinger/VPS deployment |
| `fetch-library-docs` | Fetching library/framework docs |
| `skill-creator` | Authoring new skills |
| `skill-creator-pro` | Production-grade skill authoring |
| `skill-validator` | Validating skill quality (9-category) |

### Documents & Media
| Skill | When to invoke |
|---|---|
| `pdf` | PDF extraction, creation, manipulation |
| `docx` | Word document creation/editing |
| `pptx` | Presentation creation/editing |
| `xlsx` | Spreadsheet creation/editing |
| `remotion` | AI video production |
| `doc-coauthoring` | Collaborative documentation workflow |
| `internal-comms` | Internal memos, updates, announcements |

### Business & Teams
| Skill | When to invoke |
|---|---|
| `traction-eos` | EOS/Traction operating system |
| `drive-motivation` | Autonomy, Mastery, Purpose systems |
| `negotiation` | Negotiation tactics and preparation |
| `interview` | Discovery / user interview conversations |
| `mom-test` | Customer interviews without bias |

## Rules (Non-Negotiable)
- Never hardcode secrets — use `.env`
- Smallest viable diff — no unrelated edits
- PHR created after every significant prompt
- Run `/sp.adr` for any significant architectural decision
- No AI-default UI — `ui-anti-ai` skill enforced on all frontend output
- **Check `skill-router.md` before starting any non-trivial task**

---
> Source: [Sameedshah/cod-dev-os](https://github.com/Sameedshah/cod-dev-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
