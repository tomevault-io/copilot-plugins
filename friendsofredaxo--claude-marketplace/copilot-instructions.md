## claude-marketplace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

No build, no test suite, no lint step — this is a content-only repo. Changes are Markdown and JSON edits. The "runtime" is Claude Code itself: it reads `marketplace.json`, then each `plugin.json`, then activates skills based on their frontmatter `description`.

For the user-facing pitch (what the marketplace is, how to install it, plugin overview), see [`README.md`](./README.md). This file covers what changes when editing the repo.

## Repository layout

```
.claude-plugin/marketplace.json     # Registers all plugins exposed by this marketplace
plugins/<plugin-name>/
├── .claude-plugin/plugin.json      # Plugin manifest (name, version, metadata)
├── README.md                       # User-facing description shown in /plugin
├── skills/<skill-name>/SKILL.md    # Auto-activating instruction packages
└── commands/<cmd>.md               # User-invoked slash commands
```

The plugin set is intentionally modular — users only install plugins matching the addons their REDAXO project uses. The current list lives in `marketplace.json` and the README; don't duplicate it here (it goes stale).

## How skills activate

Skills live in `plugins/<plugin>/skills/<skill>/SKILL.md` and are auto-loaded by Claude Code based on their YAML frontmatter:

```yaml
---
name: redaxo-architecture
description: REDAXO project structure, request lifecycle, and core classes. Use when the user works on a REDAXO codebase, mentions REDAXO directory layout, asks about rex_addon/rex_config/rex_path/rex_url..., or when files under redaxo/src/ appear in the conversation.
---
```

The `description` is the *only* signal Claude has for whether to load the skill. Descriptions must enumerate concrete trigger keywords (class names, file paths, terminology) — vague descriptions silently fail to activate. When editing or adding a skill, the description field is the single most important thing to get right.

Skills are namespaced per plugin (e.g. `redaxo-core:redaxo-modules`) so different plugins can ship overlapping topic names without collision.

## How slash commands work

Commands live in `plugins/<plugin>/commands/<name>.md`. Frontmatter contains a `description`; the body is the prompt Claude executes when the user types `/<plugin>:<name>`. Commands are user-invoked, not auto-activated, so their descriptions just need to be human-readable in the slash menu.

## Adding a new plugin

1. Copy an existing plugin directory under `plugins/` as a template (e.g. `plugins/redaxo-yrewrite/`).
2. Update `.claude-plugin/plugin.json` (name, version, description, homepage).
3. Replace skills under `skills/` with content for the new addon. Keep one skill per topic; don't fold unrelated topics into one SKILL.md — that defeats targeted activation.
4. Register the plugin in `.claude-plugin/marketplace.json` (the file is the source of truth — a plugin missing from this list is invisible to users even if its directory exists).
5. Bump the marketplace `metadata.version` in `marketplace.json` for non-trivial changes so installed users see updates after `/plugin marketplace update redaxo-marketplace`.

## Testing locally

The marketplace can be installed from a local path for testing before publishing:

```bash
/plugin marketplace add ./   # from the repo root, inside Claude Code
/plugin install redaxo-core@redaxo-marketplace
```

After editing skills or commands, run `/plugin marketplace update redaxo-marketplace` in the testing Claude Code session to pick up changes. There's no hot reload — skills are read at activation time.

## Content conventions

- **Code examples must reflect REDAXO idioms**: `rex::getTable('foo')` not `'rex_foo'`; `rex_path::*` / `rex_url::*` instead of hardcoded paths; `rex_request::*` instead of `$_GET`/`$_POST`; `rex_escape()` on every editor-provided value rendered to HTML.
- **Skills should include a "Common pitfalls" section** when a topic has known footguns. The existing skills follow this pattern — keep it.
- **Skill scope = one topic.** `redaxo-modules` is about modules; SQL patterns belong in `redaxo-sql-patterns`. Cross-link in prose if needed but don't duplicate.
- **No personal/workstation data** in any tracked file — no absolute paths from a developer's machine, no real names, mails, tokens. Generic placeholders only.

---
> Source: [FriendsOfREDAXO/claude-marketplace](https://github.com/FriendsOfREDAXO/claude-marketplace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
