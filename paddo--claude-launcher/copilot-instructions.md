## claude-launcher

> bun dev              # run from source

# Development

## Setup

```bash
bun install
```

## Scripts

```bash
bun dev              # run from source
bun run build        # bundle for npm
bun run check        # typecheck + lint
bun run lint:fix     # auto-fix lint issues
bun run clean        # remove dist/
```

## Before Committing

```bash
bun run check        # must pass
bun run build        # verify build works
```

## Project Structure

- `src/` - TypeScript source
- `dist/` - built JS (gitignored, built on publish)

## Code Style

- TypeScript strict mode
- ESLint for linting
- Keep it simple - no over-engineering

---
> Source: [paddo/claude-launcher](https://github.com/paddo/claude-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
