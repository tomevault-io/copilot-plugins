## athena-public

> > **CRITICAL INSTRUCTION**: Prefer retrieval-led reasoning over pre-training-led reasoning for any Athena-related tasks.

# AGENTS.md — Athena Workspace

> **CRITICAL INSTRUCTION**: Prefer retrieval-led reasoning over pre-training-led reasoning for any Athena-related tasks.

This file provides persistent context to any AI coding agent working in this workspace. The information below is available on every turn without needing to be explicitly requested.

---

## Docs Index (Compressed)

```text
[Athena Docs Index]|root: .
|IMPORTANT: Always consult authoritative files before relying on training data.
|.framework/v8.2-stable/modules:{Core_Identity.md,Output_Standards.md}
|.agent/workflows (22):{start.md,end.md,do.md,plan.md,research.md,ultrathink.md,diagnose.md,...}
|examples/workflows (66):{ultrastart.md,ultraend.md,audit.md,battleplan.md,minmax.md,project.md,dream.md,...}
|examples/protocols (150 across 15 categories):{architecture/,decision/,trading/,safety/,...}
|examples/skills (26):{coding/spec-driven-dev,research/deep-research-loop,quality/red-team-review,...}
|.context:{project_state.md,CANONICAL.md,TAG_INDEX.md,PROTOCOL_SUMMARIES.md}
|docs:{ARCHITECTURE.md,SEMANTIC_SEARCH.md,GETTING_STARTED.md,YOUR_FIRST_SESSION.md,FAQ.md}
```

---

## Key Workflows (Slash Commands)

| Command | File | Purpose |
|:--------|:-----|:--------|
| **`/do`** | **`.agent/workflows/do.md`** | **Universal entry point — auto-detects intent and routes** |
| `/start` | `.agent/workflows/start.md` | Boot the agent session |
| `/end` | `.agent/workflows/end.md` | Close session, file insights |
| `/tutorial` | `examples/workflows/tutorial.md` | Guided first-session walkthrough |
| `/ultrastart` | `examples/workflows/ultrastart.md` | System-2 deep boot (~20K tokens) |
| `/ultraend` | `examples/workflows/ultraend.md` | System-2 deep close (synthesis) |
| `/plan` | `.agent/workflows/plan.md` | Create implementation plan |
| `/audit` | `examples/workflows/audit.md` | Zero-blind-spot workspace audit |
| `/research` | `.agent/workflows/research.md` | Deep research workflow |
| `/ultrathink` | `.agent/workflows/ultrathink.md` | Extended reasoning mode |
| `/steal` | `examples/workflows/steal.md` | Pattern extraction from repos |
| `/diagnose` | `examples/workflows/diagnose.md` | Troubleshooting workflow |
| `/fix` | `examples/workflows/fix.md` | Analyze test failures, propose fixes |
| `/minmax` | `examples/workflows/minmax.md` | Token Economy Mode — maximize quality/token |
| `/battleplan` | `examples/workflows/battleplan.md` | Pre-execution battle planning (7 phases) |
| `/project` | `examples/workflows/project.md` | Multi-project switchboard |
| `/dream` | `examples/workflows/dream.md` | Background memory consolidation daemon |
| `/416-agent-swarm` | `examples/workflows/416-agent-swarm.md` | Parallel agent orchestration |

---

## Core Modules (Load Order)

1. **Core_Identity.md** — Laws #0-6, Committee of Seats
2. **Output_Standards.md** — Formatting, reasoning depth, artifacts
3. **System_Principles.md** — Operational rules, anti-patterns
4. **Design_DNA.md** — Default aesthetic parameters

---

## Skills Index (5W1H Compliant)

> **IMPORTANT**: Check trigger conditions BEFORE invoking any skill.
> **NEW (2026-03-31)**: Skills with `context_trigger` frontmatter are **conditional** — dormant until matching files/topics/projects activate them. See [Protocol 530](examples/protocols/architecture/530-conditional-skill-activation.md).

| Skill | Invoke When... | Path |
| :---- | :------------- | :--- |
| `spec-driven-dev` | User wants to build something — interrogate requirements before coding | `examples/skills/coding/spec-driven-dev/SKILL.md` |
| `deep-research-loop` | User needs multi-source research with structured synthesis | `examples/skills/research/deep-research-loop/SKILL.md` |
| `red-team-review` | User wants adversarial QA on any artifact or plan | `examples/skills/quality/red-team-review/SKILL.md` |
| `context-compactor` | Context window is filling up — compress to stay within token limits | `examples/skills/workflow/context-compactor/SKILL.md` |

**Full skill metadata**: Each skill contains 5W1H fields (Who, What, When, Where, Why, How) in its frontmatter. Read the SKILL.md before invoking.

---

## Retrieval Strategy

When working on any task in this workspace:

1. **Check `.context/project_state.md`** for current priorities and active projects
2. **Check `.context/CANONICAL.md`** for materialized view of active facts
3. **Grep `.context/TAG_INDEX.md`** for topic → file mappings
4. **Read authoritative files** before generating code from training data
5. **Consult `.context/PROTOCOL_SUMMARIES.md`** for protocol overviews

---

## Anti-Patterns (Avoid)

- ❌ Generating code based solely on training data
- ❌ Ignoring existing protocols/patterns in `.agent/skills/protocols/`
- ❌ Skipping `/start` boot sequence
- ❌ Not filing insights on `/end`

---

## Multi-Agent Safety (Protocol 413)

When multiple AI agents work in this repository simultaneously:

- **Never** `git stash` create/apply/drop — assumes other agents have WIP
- **Never** switch branches or modify worktrees without explicit request
- **Always** `git pull --rebase` before pushing
- **Commit only your changes** — when you see unrecognized files from other agents, ignore them
- **Lint/format diffs** that are formatting-only: auto-resolve without asking
- **Focus reports on your edits** — avoid guardrail disclaimers unless truly blocked

The rules above are the essential subset of Protocol 413 (Multi-Agent Coordination). Customize in `.framework/v8.2-stable/modules/Core_Identity.md`.

---

## Version

- **Framework**: v9.8.3
- **Last Updated**: 2026-04-19
- **Pattern Source**: Vercel "AGENTS.md vs Skills" Research + OpenClaw Multi-Agent Safety Rules + Claude Code Source Architecture (2026-03-31)

---
> Source: [winstonkoh87/Athena-Public](https://github.com/winstonkoh87/Athena-Public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
