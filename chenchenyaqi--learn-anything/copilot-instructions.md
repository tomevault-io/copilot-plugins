## learn-anything

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Learn Anything is a CLI tool (`learn-anything`) that generates skill and command files for AI coding assistants, turning them into interactive learning tutors. It supports 30+ AI tools (Claude Code, Cursor, Gemini CLI, Codex, Copilot, Windsurf, etc.) and outputs localized files in `en` and `zh-CN`.

The generated skills implement 5 learning workflows: topic (initialize a subject), explain (recursive Socratic deep-dive), practice (TDD-style exercises), review (spaced repetition), and status (knowledge map visualization).

## Commands

```bash
pnpm build          # Compile TypeScript via tsc (runs node build.js)
pnpm dev            # tsc --watch
pnpm test           # Run all tests once (vitest run)
pnpm test:watch     # Run tests in watch mode (vitest)
pnpm lint           # ESLint on src/
pnpm dev:cli        # Build and run the CLI locally
```

## Architecture

```
src/
  cli/index.ts          # Commander.js CLI: `learn-anything init [path]` and `learn-anything update [path]`
  core/
    init.ts             # InitCommand — orchestrates tool detection, interactive selection,
                        #   skill generation, and command generation
    config.ts           # AI_TOOLS array (30+ tools with skillsDir mappings), LEARN_DIR
    command-generation/ # Adapter pattern: each tool has an adapter that knows its file format
                        #   and directory conventions (Claude → .claude/commands/, YAML frontmatter;
                        #   Gemini → .gemini/commands/, TOML; Codex → ~/.codex/prompts/)
      types.ts          # CommandContent, ToolCommandAdapter, GeneratedCommand interfaces
      registry.ts       # CommandAdapterRegistry — maps tool IDs → adapters
      generator.ts      # generateCommand / generateCommands — applies adapter to content
      adapters/         # claude.ts, cursor.ts, codex.ts, gemini.ts
    templates/
      types.ts          # SkillTemplate, CommandTemplate interfaces
      skill-templates.ts # Re-exports all 5 workflow template getters
      workflows/        # learn-topic.ts, learn-explain.ts, learn-practice.ts,
                        #   learn-review.ts, learn-status.ts
                        #   Each exports getXxxSkillTemplate(locale) and getXxxCommandTemplate(locale)
    shared/
      skill-generation.ts  # Aggregates templates; generateSkillContent() writes YAML frontmatter
  i18n/
    index.ts            # getMessages(locale), detectSystemLocale(), resolveLocale()
    types.ts            # LocaleMessages, SkillsMessages, CLIMessages, InitMessages types
    locales/
      en.ts, zh-CN.ts   # Top-level locale messages (CLI strings, init messages)
      skills/{en,zh-CN}/ # Per-workflow skill/command content (topic.ts, explain.ts, etc.)
  utils/
    file-system.ts      # ensureDir, writeFile, fileExists, dirExists, removeDir
    interactive.ts      # isInteractive() — checks process.stdin/stdout.isTTY
```

### Key Patterns

- **Templates are locale-aware**: every template getter takes `locale: SupportedLocale` and pulls strings from i18n. The same template produces different content for `en` vs `zh-CN`.
- **Adapter pattern for multi-tool output**: adding support for a new AI tool means creating a new adapter in `command-generation/adapters/` that implements `ToolCommandAdapter` (specifying file path conventions and file format) and registering it.
- **Shared data in `./.learn/`**: the CLI creates `./.learn/topics/` in the project directory for learning state that stays with the project.
- **Interactive by default**: when no `--tools` flag is passed and stdin/stdout are TTYs, `learn-anything init` shows an interactive checkbox prompt (via `@inquirer/prompts`) with detected tools pre-selected.

### Adding a new AI tool

1. Add an entry to `AI_TOOLS` in `src/core/config.ts` with the tool's `skillsDir` path.
2. If the tool has custom command file conventions, create an adapter in `src/core/command-generation/adapters/` and register it in `src/core/command-generation/adapters/index.ts` and `registry.ts`.

---
> Source: [ChenChenyaqi/learn-anything](https://github.com/ChenChenyaqi/learn-anything) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
