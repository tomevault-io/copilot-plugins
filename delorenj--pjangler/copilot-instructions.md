## pjangler

> My opinionated project bootstrapper, tuned to my personal development ecosystem.

# PJangler

My opinionated project bootstrapper, tuned to my personal development ecosystem.

## Features

- Architected as a command pattern for easy extension (using recipes to chain commands)
- Accompanying Claude Skill for easy agent integration
- Idempotent operations (safe to re-run)
- Supports multiple project types (Node.js, Python, Go, Rust, etc.)
- Dry run first with validation before making changes (i.e. mutually exclusive recipes issues warning)

## Typer Interface
```bash
pjangler [OPTIONS] COMMAND [ARGS]...
pjangler recipe list # Lists available recipes
pj recipe describe [RECIPE_NAME] # Lists commands in recipe
pj recipe run [RECIPE_NAME] [--dry-run] # Runs specified recipe
pj command list [--group] # Lists available commands
pj command describe [COMMAND_NAME] # Describes specified command
pj command create [COMMAND_NAME] [PROMPT] [--template TEMPLATE_NAME] [--model=OPENROUTER_MODEL]# Creates new command from template
pj describe # Runs a prompt to describe current project with output matching a template for consistency
```

## Basic Package Management

Default to using Bun instead of Node.js.

- Use `bun <file>` instead of `node <file>` or `ts-node <file>`
- Use `bun test` instead of `jest` or `vitest`
- Use `bun build <file.html|file.ts|file.css>` instead of `webpack` or `esbuild`
- Use `bun install` instead of `npm install` or `yarn install` or `pnpm install`
- Use `bun run <script>` instead of `npm run <script>` or `yarn run <script>` or `pnpm run <script>`
- Bun automatically loads .env, so don't use dotenv.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/delorenj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/delorenj)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
