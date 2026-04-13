## awesome-agents

> <!-- Knowledge priming document for AI agents and new contributors. -->

# AGENTS.md

<!-- Knowledge priming document for AI agents and new contributors. -->
<!-- Follows the Fowler Knowledge Priming structure (7 sections). -->
<!-- Keep this file to 1–3 pages. Contributor conventions are in this same file below. -->
<!-- Last updated: 2026-02-25 -->

---

## Architecture Overview

This repository is a **collection of reusable agent skills** following the Agent Skills open standard. Each skill is a self-contained directory containing a `SKILL.md` file (with YAML frontmatter) and optional executable scripts, reference docs, and static assets. Skills are portable: they work with Claude Code, Cursor, Codex CLI, OpenCode, Gemini CLI, and Amp — any agent that supports the standard.

The repo has no runtime server, build system, or package manager. All logic lives in Markdown instructions and Bash/Go scripts. Agents read `SKILL.md` upfront and load `references/` on demand, keeping token usage proportional to need. An `agentic-docs/` plugin provides a full documentation suite, while `shared/prompts/` holds common safety instructions injected across skills.

---

## Tech Stack and Versions

| Layer | Technology | Notes |
|-------|-----------|-------|
| Primary language | Bash | Hook scripts, setup scripts, skill orchestration |
| Secondary language | Go | HTML-to-Markdown binary in `markdown_crawl/scripts/` |
| Skill format | Markdown + YAML frontmatter | Parsed by agent runtimes |
| Config | JSON | `.claude-plugin/marketplace.json`, metrics files |
| Agent platforms | Claude Code, Cursor, Codex, OpenCode, Gemini CLI, Amp | Consuming runtimes, not dependencies |

No package.json, pyproject.toml, or go.mod at the repo root. The only `go.mod` is inside `skills/markdown_crawl/scripts/` for the Go binary.

---

## Curated Knowledge Sources

- [Agent Skills standard](https://agentskills.io) — official spec for SKILL.md format, frontmatter fields, and skill directories
- [Cursor Skills docs](https://cursor.com/docs/context/skills) — how Cursor loads and triggers skills
- [Claude Code hooks docs](https://code.claude.com/docs/en/hooks) — lifecycle events used by `setup-agent-cli-hooks`
- [Cloudflare Markdown for Agents](https://blog.cloudflare.com/markdown-for-agents/) — background for `markdown_crawl` skill
- [Cloudflare Markdown for Agents developer docs](https://developers.cloudflare.com/fundamentals/reference/markdown-for-agents/) — HTTP header spec (`Accept: text/markdown`, `X-Markdown-Tokens`)
- [Fowler — Knowledge Priming](https://martinfowler.com/articles/reduce-friction-ai/knowledge-priming.html) — rationale behind AGENTS.md structure
- `AGENTS.md` (this file) — contributor conventions (skill naming, frontmatter rules, review checklist)

---

## Project Structure

```
agent-skills/
├── skills/                        # One directory per skill
│   ├── _template/                 # Starter template — copy this to create a new skill
│   │   └── SKILL.md
│   ├── make-plan/                 # Instruction-only skill (no scripts)
│   │   └── SKILL.md
│   ├── markdown_crawl/            # Script-backed skill (Bash + Go binary)
│   │   ├── SKILL.md
│   │   ├── metrics.json           # Persistent usage metrics (auto-updated at runtime)
│   │   └── scripts/
│   │       ├── markdown_crawl.sh  # Main fetch/compare/stats script
│   │       ├── html_to_markdown   # Pre-built Go binary (Linux AMD64)
│   │       └── build.sh           # Rebuilds the Go binary
│   ├── prepare-agents-md/         # Instruction-only skill with reference docs
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── fowler-structure.md
│   └── setup-agent-cli-hooks/     # Script-backed skill
│       ├── SKILL.md
│       └── scripts/
│           └── setup-agent-hooks.sh
├── agentic-docs/                  # Full documentation suite plugin (separate from skills)
│   ├── agents/doc-setup.md        # Agent instructions for generating docs
│   └── commands/                  # (placeholder)
├── shared/
│   └── prompts/common.md          # Shared safety prompt injected across skills
├── .claude-plugin/
│   └── marketplace.json           # Plugin marketplace metadata
├── AGENTS.md                      # This file
├── CLAUDE.md                      # Claude Code project settings
└── README.md                      # Human-readable project overview
```

---

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Skill directory names | lowercase, hyphens | `make-plan`, `prepare-agents-md` |
| `name` field in SKILL.md frontmatter | Must match folder name exactly | `name: prepare-agents-md` |
| Script files | kebab-case `.sh` or descriptive binary name | `setup-agent-hooks.sh`, `html_to_markdown` |
| Reference files | `SCREAMING_SNAKE.md` or descriptive kebab | `fowler-structure.md` |
| Metrics/data files | kebab-case `.json` | `metrics.json` |
| YAML frontmatter tags | lowercase strings | `["hooks", "cli", "setup"]` |

**Note**: AGENTS.md documents `_template` uses underscores, but all real skills use hyphens. Follow hyphens for new skills.

---

## Code Examples

### Minimal SKILL.md (instruction-only skill)

```yaml
# from skills/make-plan/SKILL.md
---
name: make-plan
description: Make a detailed plan to fix or resolve an issue/problem. Follows a 6-step
  workflow: understand the problem, review docs, explore the codebase, create a detailed
  plan, write an exec plan file, and update PLANS.md.
license: MIT
metadata:
  author: v.duc
  version: "2.0.0"
---
```

Instruction-only skills have no `scripts/` directory. All logic is in the Markdown body.

### Script-backed skill invocation pattern

```bash
# from skills/markdown_crawl/SKILL.md — how the skill calls its script
bash skills/markdown_crawl/scripts/markdown_crawl.sh fetch <url>
bash skills/markdown_crawl/scripts/markdown_crawl.sh compare <url>
bash skills/markdown_crawl/scripts/markdown_crawl.sh stats
```

Scripts are always invoked from the repo root using the full relative path. Skills never rely on `cd` into the script directory.

### Metrics JSON shape

```json
// from skills/markdown_crawl/metrics.json
{
  "total_requests": 0,
  "successful_markdown_requests": 0,
  "failed_requests": 0,
  "total_markdown_tokens": 0,
  "requests_history": []
}
```

---

## Anti-patterns to Avoid

- **Do not set `name` in frontmatter to anything other than the exact folder name.** Agent runtimes use this field to identify and trigger skills — a mismatch causes silent failures.
- **Do not use underscores in new skill folder names.** The convention migrated to hyphens (see `make-plan`, `prepare-agents-md`). The `_template` and `markdown_crawl` names are legacy.
- **Do not create README.md, INSTALLATION_GUIDE.md, or CHANGELOG.md inside skill directories.** Skills are for AI agents, not human onboarding. Documentation for humans lives in the repo root `README.md`.
- **Do not add `disable-model-invocation: true` unless the skill is a strict slash command.** Setting this prevents the skill from being triggered automatically by description match, which is how most skills are discovered.
- **Do not put all documentation inline in SKILL.md.** Move reference material to `references/` for progressive loading. SKILL.md should stay under ~500 lines so it doesn't bloat the agent's context on every invocation.
- **Do not invent version numbers, URLs, or technology names in skill instructions.** Every factual claim must be verifiable from the codebase. Fabricated data causes agents to hallucinate confidently incorrect answers.
- **Do not commit the pre-built `html_to_markdown` binary for non-Linux-AMD64 platforms.** Run `bash build.sh` locally to rebuild for the target platform.

---

## Contributor Conventions

*(This section is a condensed version of the full contributor guide. See the original AGENTS.md for the complete checklist.)*

### Skill Naming
- Lowercase with hyphens for directory names (`prepare-agents-md`, not `PrepareAgentsMd`)
- `name` field in YAML frontmatter **must** match folder name exactly

### Skill Structure
- Required: `SKILL.md` with valid YAML frontmatter
- Optional: `scripts/` (executables), `references/` (on-demand docs), `assets/` (static files)

### Development Workflow
1. Copy `skills/_template/` to `skills/<your-skill-name>/`
2. Update YAML frontmatter (`name`, `description`, `metadata`)
3. Write instructions in the Markdown body
4. Add scripts/references if needed
5. Verify `name` matches folder name

### Code Review Checklist
- [ ] `name` in frontmatter matches folder name exactly
- [ ] `description` clearly states what the skill does and when to trigger it
- [ ] YAML frontmatter is valid
- [ ] Instructions include a "When to Use" section
- [ ] No sensitive data or credentials
- [ ] Scripts are executable and have error messages
- [ ] Detailed docs are in `references/`, not inlined in SKILL.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ducva) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
