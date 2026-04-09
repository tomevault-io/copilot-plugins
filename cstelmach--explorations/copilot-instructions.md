## explorations

> Experimental frameworks and skills for Claude Code.

# Explorations Framework

Experimental frameworks and skills for Claude Code.
Each skill lives under `skills/` as a self-contained unit following Claude Code skill conventions.

## Critical Rules

1. Skills follow the 6-category resource taxonomy: `actions/`, `references/`, `templates/`, `scripts/`, `troubleshooting/`, `examples/`
2. SKILL.md files must be <3000 words — they are hub files that dispatch to action files
3. Action files use `<system>` + `<instructions>` + `<task>` tags (see edu-course-creator pattern)
4. Reference files are loaded on-demand by action files, never duplicated in SKILL.md
5. All file names use kebab-case, descriptive, self-documenting for LLM tools
6. Templates contain placeholder guidance as markdown comments (`<!-- guidance -->`)

## Git Protocol

- Atomic commits per logical unit using conventional commit messages (feat, fix, docs, refactor)
- Never commit `.forge/` runtime state (it's gitignored)
- Commit after each implementation phase completes

## Skills

| Skill | Purpose | Entry Point |
|-------|---------|-------------|
| [idea-forge](skills/idea-forge/SKILL.md) | Idea → Product pipeline via Double Diamond | `/forge:ignite` |

## Research Persistence

**All research reports MUST be saved permanently — never leave them only in `/tmp/`.**

- **Inside forge pipeline**: Reports are saved to `docs/forge/{slug}/research/layers/` (project directory exists before research starts)
- **Outside forge pipeline** (ad-hoc research, pre-idea exploration): Save to `initial-research/{YYYY-MM-DD}_{descriptor}/`
  - Example: `initial-research/2026-03-20_markdown-pdf-landscape/`
  - When a forge project is later created from this research, move the reports into `docs/forge/{slug}/research/layers/`
- `/tmp/` is the agent staging area only — always copy to a permanent location after agent completes
- See `skills/idea-forge/references/research-agent-dispatch.md` — Output Protocol section for `/tmp/` naming conventions

## Testing

Each skill includes `examples/` with a full walkthrough.
Validate pipeline artifact health: `bash skills/idea-forge/scripts/validate-pipeline.sh <slug>`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cstelmach)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cstelmach)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
