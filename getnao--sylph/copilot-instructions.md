## sylph

> This file is the single source of truth for all AI agents working in this repo.

# Agent Workflow - Sylph

This file is the single source of truth for all AI agents working in this repo.
Read it before starting any task. Company and product context lives in `CONTEXT.md`.

> **Cross-tool support**: this file is read natively by Codex CLI and Cursor,
> and imported by Claude Code via `CLAUDE.md`. Skills follow the
> [Agent Skills](https://agentskills.io) open standard and live in `.claude/skills/`.

---

## 0. Definitions

The **CAO (Chief Agent Officer)** is the human operator who owns this repo. The CAO reviews, approves, and has final say on everything agents produce.

<!-- After running /sylph-setup, replace this with the CAO's name and pronouns -->

---

## 1. Orient first

Before doing anything, identify:
- **What domain** does this task belong to? (content, sales, product, brand, finance, etc.)
- **Which channel or subfolder** is relevant? (e.g. `content/linkedin/`, `sales/outbound/`)
- **Is there a skill** for this task? Check the skill index below.

If a skill exists, load and follow it. Skills encode channel-specific rules, tone, format, and workflow steps.

---

## 2. Read before writing

Before generating any content or making decisions:

1. Read the relevant `_insights.md` for the channel/domain - it contains what works, what doesn't, cadence, and audience notes.
2. Read the `_examples/` files for the channel - these are the best-performing pieces and serve as the quality bar.
3. Read any relevant `_templates/` - use them as structural starting points.

If these files are empty, proceed without them but note that examples and insights are missing.

---

## 3. Content lifecycle - always follow this

Every piece of content follows this path:

```
create -> _drafts/ -> review/edit -> _published/ -> update _insights.md -> promote to _examples/ if high-performing
```

| Stage | Folder | When |
|-------|--------|------|
| Working draft | `_drafts/` | While creating or iterating |
| Finalized/published | `_published/` | After it's approved and sent/posted |
| Insights update | `_insights.md` | After publishing - log what worked, what didn't, performance |
| Example | `_examples/` | Only when a piece performs well - used for future generation |

**Always save new content to `_drafts/` first.** Never write directly to `_published/` or `_examples/`.

---

## 4. File naming

```
YYYY-MM-DD_short-slug.md
```

Example: `2026-04-05_linkedin-context-engineering.md`

---

## 5. Frontmatter

All content files must include frontmatter:

```yaml
---
date: YYYY-MM-DD
channel: linkedin
topic: context-engineering
format: post
status: draft
performance:
notes:
---
```

---

## 6. Updating memory

After completing a task, if you learned something non-obvious:
- Log decisions and learnings in `.claude/MEMORY.md`
- Update the relevant `_insights.md` if content performance or patterns were observed

---

## 7. Repo map

```
AGENTS.md           - this file (canonical instructions for all agents)
CONTEXT.md          - stable company/product facts
CLAUDE.md           - Claude Code loader (imports this file)

.claude/
  skills/           - skill files (flat: skills/<skill-name>/SKILL.md)
  MEMORY.md         - living decision + learning log

.agents/
  skills -> .claude/skills   (symlink - Codex auto-discovery)

agents/             - AI employees (ROLE.md + PROMPT.md, scheduled tasks)

brand/              - brand assets, guidelines, voice
content/            - all content by channel
  <channel>/
    _drafts/        - works in progress
    _published/     - archive of published pieces
    _examples/      - best-performing pieces (few-shot reference)
    _templates/     - reusable formats
    _insights.md    - channel learnings

sales/              - outbound sequences, proposals, copywriting
customer-success/   - meeting follow-ups, user research
product/            - roadmap, issues
finance/            - investor updates
partnerships/       - consulting, freelance, influencer
events/             - hackathons, meetups, remote
admin/              - internal ops
compliance/         - compliance notes
legal/              - contracts and legal
  templates/        - contract templates
  _drafts/          - contracts in progress
  _published/       - signed contracts archive
```

---

## 8. Skill index

Skills follow the [Agent Skills](https://agentskills.io) open standard. Each has a `SKILL.md` with `name` and `description` frontmatter.

**How to invoke:**
- **Claude Code**: type `/skill-name` (auto-discovered from `.claude/skills/`)
- **Cursor**: type `/skill-name` (auto-discovered from `SKILL.md` frontmatter)
- **Codex CLI**: use `$skill-name` or ask by name (auto-discovered from `.agents/skills/`)

| Skill | Domain | Path |
|-------|--------|------|
| chief-of-staff | agents | `.claude/skills/chief-of-staff/SKILL.md` |
| cmo | agents | `.claude/skills/cmo/SKILL.md` |
| customer-success | agents | `.claude/skills/customer-success/SKILL.md` |
| executive-assistant | agents | `.claude/skills/executive-assistant/SKILL.md` |
| head-of-data | agents | `.claude/skills/head-of-data/SKILL.md` |
| head-of-sales | agents | `.claude/skills/head-of-sales/SKILL.md` |
| product-manager | agents | `.claude/skills/product-manager/SKILL.md` |
| update-docs | agents | `.claude/skills/update-docs/SKILL.md` |
| brand-designer | brand | `.claude/skills/brand-designer/SKILL.md` |
| brand-guidelines | brand | `.claude/skills/brand-guidelines/SKILL.md` |
| blog-writer | content | `.claude/skills/blog-writer/SKILL.md` |
| linkedin | content | `.claude/skills/linkedin/SKILL.md` |
| newsletter | content | `.claude/skills/newsletter/SKILL.md` |
| reddit | content | `.claude/skills/reddit/SKILL.md` |
| slack-community | content | `.claude/skills/slack-community/SKILL.md` |
| substack-writer | content | `.claude/skills/substack-writer/SKILL.md` |
| website | content | `.claude/skills/website/SKILL.md` |
| x | content | `.claude/skills/x/SKILL.md` |
| event-manager | events | `.claude/skills/event-manager/SKILL.md` |
| investor-update | finance | `.claude/skills/investor-update/SKILL.md` |
| hr-screening | hr | `.claude/skills/hr-screening/SKILL.md` |
| create-contract | legal | `.claude/skills/create-contract/SKILL.md` |
| review-contract | legal | `.claude/skills/review-contract/SKILL.md` |
| zero-inbox | ops | `.claude/skills/zero-inbox/SKILL.md` |
| create-issue | product | `.claude/skills/create-issue/SKILL.md` |
| create-campaign | sales | `.claude/skills/create-campaign/SKILL.md` |
| crm | sales | `.claude/skills/crm/SKILL.md` |
| customer-report | sales | `.claude/skills/customer-report/SKILL.md` |
| email-writer | writing | `.claude/skills/email-writer/SKILL.md` |
| sylph-create-skill | sylph | `.claude/skills/sylph-create-skill/SKILL.md` |
| sylph-setup | sylph | `.claude/skills/sylph-setup/SKILL.md` |
| sylph-setup-agent | sylph | `.claude/skills/sylph-setup-agent/SKILL.md` |
| sylph-setup-skill | sylph | `.claude/skills/sylph-setup-skill/SKILL.md` |

---

## 9. Adding new skills

### How to create a skill

1. Create `.claude/skills/<skill-name>/SKILL.md`
2. Include `name` and `description` in YAML frontmatter
3. Add the skill to the index table above

```yaml
---
name: my-skill
description: One line explaining what the skill does and when to use it.
---

# Skill Title

Instructions here...
```

### How each tool discovers skills

| Tool | Discovery path | Mechanism |
|------|---------------|-----------|
| Claude Code | `.claude/skills/` | Scans this directory at session start |
| Codex CLI | `.agents/skills/` | Symlink to `.claude/skills/` - Codex auto-discovers from here |
| Cursor | `.claude/skills/` | Finds `SKILL.md` files with `name`/`description` frontmatter project-wide |

All three tools also read `AGENTS.md` for workflow instructions and the skill index.

### Why this structure - do not change it

Skills must be flat under `.claude/skills/<skill-name>/SKILL.md` (one level).
Claude Code only discovers skills at this depth - adding domain subdirectories
(e.g. `.claude/skills/agents/chief-of-staff/`) breaks discovery.

Codex CLI requires `.agents/skills/` for auto-discovery, so we symlink it to
`.claude/skills/`. This gives both tools the same skills from a single source.

Cursor scans the whole project for `SKILL.md` files. `.cursorignore` excludes
`.agents/skills/` (symlink duplicate) and `.claude/worktrees/` (Claude Code
git worktrees that contain full repo copies) to prevent duplicates.

Do not:
- Add domain subdirectories inside `.claude/skills/` - Claude Code only scans one level deep
- Move skills to `.agents/skills/` - Claude Code would stop finding them
- Remove the `.agents/skills` symlink - Codex would lose auto-discovery
- Remove `.cursorignore` - Cursor would show duplicate slash commands

---
> Source: [getnao/sylph](https://github.com/getnao/sylph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
