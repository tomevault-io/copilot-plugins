## devsync

> | Command         | Description                       |

# AGENTS.md - Developer Guide

## Quick Commands

| Command         | Description                       |
| --------------- | --------------------------------- |
| `bun install`   | Install dependencies              |
| `bun run build` | Build CLI (outputs to `dist/`)    |
| `bun test`      | Vitest (test path mirrors source) |

**Single test:** `bun test test/cli/utils/run-bun-command.test.ts`  
**Web docs:** `cd apps/web && bun dev`

## Project Structure

```
apps/
  cli/          # CLI app (entry: apps/cli/index.ts)
  @core/        # DEVSYNC.json schema + zod validation
  web/          # Astro + Starlight docs
  devsync/      # Core logic
test/cli/       # Tests mirror apps/cli structure
```

## Architecture

- **Mixin pattern**: Infrastructure (readFile, writeFile, createPdf) extends base constructor
- **Error handling**: Custom classes via `CreateError<T>()` factory (`NotFound`, `Forbidden`, `Conflict`, `ServerError`, `BadRequest`)
- **Paths**: `@/*` → `apps/cli/*`, `@devsync/*` → `apps/devsync/*`

## Conventions

- **Files:** kebab-case
- **Functions/Variables:** camelCase
- **Types:** PascalCase
- **Constants:** UPPER_SNAKE_CASE
- **Errors:** PascalCase with suffix (e.g., `ServerError`)
- **Imports:** Type imports for types

## Gotchas

- Bun runtime (not Node)
- TypeScript `noEmit: true` (bundler workflow)
- Astro web uses Starlight + Tailwind v4

## Web (apps/web)

**Styling: Tailwind only.** Do not mix `<style>` blocks with Tailwind classes.

`<style>` is only allowed for CSS that has no Tailwind equivalent:

- `text-shadow`
- `@keyframes` with staggered animation delays
- Combined transforms (e.g. `skewY(-2deg) rotate(-1deg)`)

Everything else — colors, spacing, layout, opacity, blur, borders, transitions — must use Tailwind classes, including arbitrary values (e.g. `opacity-[0.04]`, `blur-[80px]`).

---
> Source: [Jannael/Devsync](https://github.com/Jannael/Devsync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
