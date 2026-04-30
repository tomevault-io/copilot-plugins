## claude-obsidian-assistant

> This is an Obsidian vault organized with [Johnny Decimal](https://johnnydecimal.com/). Claude Code can read and write every file in it directly — no plugins or CLI tools needed.

# Claude Code + Obsidian Vault

This is an Obsidian vault organized with [Johnny Decimal](https://johnnydecimal.com/). Claude Code can read and write every file in it directly — no plugins or CLI tools needed.

Everything here is a starting point. The folder structure, templates, themes, and skill phases are all meant to be changed. If the user wants a different organization system, different file formats, extra automation, or fewer steps — adjust. This framework works best when shaped around how the user actually thinks and works, not the other way around.

## Vault Structure

```
00-09 System      — Meta (home dashboard, vault map), templates, vault config
10-19 Projects    — Active project files (11.XX numbering), archive, skills
20-29 Writing     — Essays, notes, daily journal
30-39 Knowledge   — Resources, reading list
40-49 Tracking    — Finance, routines, to-dos
50-59 Creative    — Drawings, media, experiments
```

Active project files live in `10-19 Projects/11 Active/` and follow the naming pattern `11.XX {Project Name}.md`. The XX number auto-increments.

## Key Files to Read First

| File | What it is |
|------|------------|
| `claude-code/obsidian-sync.md` | The `/obsidian` skill — full instructions for syncing projects into the vault |
| `claude-code/canvas-reference.md` | JSON format reference for Obsidian `.canvas` files |
| `claude-code/SETUP.md` | Quick-start guide for the `/obsidian` skill |
| `00-09 System/01 Templates/01.01 Project.md` | Template that defines the standard project file format |
| `00-09 System/00 Meta/00.01 Home.md` | Dashboard home page (uses Dataview queries) |
| `00-09 System/00 Meta/00.02 Vault Map.canvas` | Visual map of vault contents |

## How Obsidian Files Work

- **Markdown files** (`.md`) — standard markdown with optional YAML frontmatter between `---` fences
- **Canvas files** (`.canvas`) — JSON with two arrays: `nodes` and `edges`. See `canvas-reference.md` for the full spec
- **Internal links** — use `[[filename]]` syntax (e.g., `[[00.01 Home]]`)
- **Tags** — in frontmatter as YAML arrays, or inline as `#TagName`
- **CSS snippets** — `.obsidian/snippets/` contains theme files. Each is self-contained with sidebar colors, icons, and styling

## Project File Format

Every project file follows this structure (see the template for the full version):

1. **YAML frontmatter** — `jd-id`, `status`, `tags`
2. **Backlink** to Home — `[[00.01 Home]]`
3. **Title + description** — 2-3 sentences from actual codebase analysis
4. **Metadata** — Location (absolute path), GitHub URL, Port (if web app)
5. **Stack table** — Layer / Tech / Purpose
6. **Architecture** — Core systems, data flow using arrow notation
7. **Key files table** — Important files and their roles
8. **Dev log** — Dated entries, newest first, categorized by feature/area
9. **Footer** — Parent link + tags

## Rules When Editing Vault Files

- **Never remove existing content** without asking — only add or update
- **Dev logs are append-only** — never edit past entries, newest entries go first
- **Show diffs** before overwriting existing sections
- **Canvas files must be valid JSON** — always validate before saving
- **Descriptions should be specific** — write from actual codebase analysis, not filler
- **Auto-increment JD numbers** — scan existing `11.XX` files to find the next number

## Canvas Color Codes

| Code | Color | Use for |
|------|-------|---------|
| `"1"` | Red | Entry points, critical systems |
| `"2"` | Orange | State management, config |
| `"3"` | Yellow | Data layer, persistence |
| `"4"` | Green | UI, views, output |
| `"5"` | Cyan | API layer, external services |
| `"6"` | Purple | Labels, section headers |

## The `/obsidian` Skill

When a user runs `/obsidian` from a project directory, follow the phases defined in `claude-code/obsidian-sync.md`. The short version:

1. **Detect** — match the working directory to an existing vault project file
2. **Analyze** — read package.json, directory structure, git history
3. **Menu** — offer actions: update info, write dev log, refresh key files, generate architecture canvas, update vault map, or full sync
4. **Execute** — write changes to the vault
5. **Report** — summarize what was created or updated

---
> Source: [nemocake/claude-obsidian-assistant](https://github.com/nemocake/claude-obsidian-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
