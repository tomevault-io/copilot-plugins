## brain-spec-skills-claude

> Spec-driven development via 4 Claude Code slash commands. No build step, no npm dependencies — just markdown skill files.

# Brain Spec Skills

Spec-driven development via 4 Claude Code slash commands. No build step, no npm dependencies — just markdown skill files.

## Skills

- `/brain-init` — Initialize `.brain-spec/` workspace, steering docs, CLAUDE.md generation
- `/brain-spec` — Spec lifecycle: create, interview, list, get, update, delete, archive
- `/brain-task` — Task management: create, update, list, log, progress
- `/brain-status` — Quick dashboard overview (runs in fork context)

## Key Directories

- `.claude/skills/brain-{init,spec,task,status}/` — Skill implementations
- `docs/legacy/SPEC.md` — Legacy MCP server reference (project pivoted to skills)

## Data Directory Convention

Skills operate on `.brain-spec/` in the target project:

```
.brain-spec/
  config.json
  specs/{slug}.md + {slug}.meta.json
  tasks/{slug}/tasks.json + logs/{id}.log.md
  steering/{product,tech,structure}.md
  archive/{slug}/archive-metadata.json
```

## Design Principles

- Zero startup cost — skills loaded only when invoked
- All schemas and templates embedded in SKILL.md files
- Write access constrained to `.brain-spec/` and `CLAUDE.md` only
- Git operations are read-only (auto-enrichment for logs)
- Interview state persists in .meta.json, resumable across sessions

---
> Source: [peopleforrester/Brain_spec_skills_claude](https://github.com/peopleforrester/Brain_spec_skills_claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
