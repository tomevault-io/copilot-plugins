## glinter

> - Install deps: `bun install`

# Copilot Instructions for Glinter

## Build, test, and lint commands

- Install deps: `bun install`
- Build: `bun run build` (outputs `dist/index.js`)
- Lint: `bun run lint`
- Auto-format: `bun run lint:fix`
- Run all tests: `bun test` or `bun run test`
- Run one test file: `bun test src/test/add/app/add-command.test.ts`
- Run tests by name: `bun test -t "SwitchCommand"`

## High-level architecture

- `src/index.ts` is the CLI entrypoint:
  - `g add` (without extra args) runs the interactive add workflow
  - `g switch` (without extra args) runs the interactive branch switch workflow
  - everything else is passed directly to `git` via `Bun.spawn(..., { stdio: 'inherit' })` to preserve native git interactivity/colors
- Features are vertical slices under `src/modules/*` with Hexagonal layering:
  - `domain/`: entities + repository interfaces (ports)
  - `app/`: use cases + command orchestration
  - `infra/`: Bun/git implementations (adapters)
  - `main.ts`: composition root wiring repo -> use cases -> command
- Current slices:
  - `modules/add`: parses status, filters sensitive files, interactive multiselect staging
  - `modules/switch`: lists local/remote branches, interactive select, checkout execution
- Shared cross-cutting modules:
  - `src/utils/*`: prompt wrappers, colors, terminal icons
  - `src/error/*`: typed error constructors + centralized console rendering
  - `src/test/*`: test structure mirrors feature slices

## Key conventions in this repo

- Use **Bun** tooling and runtime (no npm/yarn flows in this repo’s workflow).
- Keep the “transparent wrapper” behavior for non-enhanced commands: forward to git with inherited stdio instead of rebuilding git UX.
- For status parsing in add flow, use `git status --porcelain -z` and NUL-splitting (`'\0'`) for robust filename handling.
- `Change.fromPorcelain()` intentionally skips already-staged entries and handles rename/copy records spanning multiple porcelain entries.
- Sensitive paths (`.env`, `node_modules`) are filtered from selectable staging while collecting warning messages.
- Interactive prompts should go through `src/utils/select.ts` and `src/utils/multiselect.ts`; cancel paths call `cancel(...)` and `process.exit(0)`.
- Error reporting is centralized through `errorHandler(...)` with typed errors from `error-instance.ts` (`ServerError`, `NotFound`, etc.) and icon/color formatting.
- Tests use Vitest and commonly mock prompt wrappers (`src/utils/select`, `src/utils/multiselect`) to test command orchestration without interactive TTY input.

---
> Source: [Jannael/Glinter](https://github.com/Jannael/Glinter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
