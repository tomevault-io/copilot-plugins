## askscout

> The daily digest for vibe coders. Scout reads through your repo and tells you what you built, what changed, and where you left off.

# askscout

The daily digest for vibe coders. Scout reads through your repo and tells you what you built, what changed, and where you left off.

This file briefs AI agents and contributors on how the codebase is laid out, what commands to run, and what conventions to follow. For the system-level "how it works" view, see [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md).

## Project structure

pnpm monorepo with three packages:

- `packages/core`: shared library (`askscout-core`). Git reading, LLM summarization, output formatting, project state management.
- `packages/cli`: CLI tool published to npm as `askscout`. Thin wrapper around core.
- `apps/web`: Next.js web app deployed at askscout.dev. Imports from core but does not publish.

Cross-package imports use the `workspace:*` protocol. Build order is core → cli/web (cli and web both depend on core's compiled output).

## Commands

```bash
pnpm install              # install all dependencies
pnpm build                # build all packages (core first, then cli + web)
pnpm dev                  # start all packages in dev/watch mode
pnpm test                 # run all tests via vitest
pnpm typecheck            # type-check all packages via tsc --build
pnpm lint                 # lint all packages via eslint
pnpm lint:fix             # lint and auto-fix
pnpm format               # format all files via prettier
pnpm format:check         # check formatting without writing
pnpm changeset            # describe a version bump for the next release
```

## Conventions

- All source code lives in `src/` directories
- TypeScript strict mode everywhere, no `any` unless explicitly justified
- Cross-package dependencies use `workspace:*` protocol
- Shared devDependencies (eslint, prettier, typescript, vitest, tsup) live at root
- Runtime dependencies live in each package's own package.json
- Public exports from core go through the barrel file at `packages/core/src/index.ts`
- No em-dashes in user-facing copy
- No dog metaphors and no "horizon" metaphors in user-facing copy (the brand voice doesn't lean on either)

## Build order when adding features

1. Types in `packages/core/src/types.ts`
2. Core logic in `packages/core/src/`
3. Export from the barrel (`packages/core/src/index.ts`)
4. CLI integration in `packages/cli/`
5. Web integration in `apps/web/`

This order matters because `cli` and `web` consume core's compiled output, adding a new export upstream and consuming it downstream in the same PR works because pnpm rebuilds in order.

## Key product decisions

- **AI providers:** two LLM providers supported (auto-detected from API key prefix)
- **CLI:** BYOK (bring your own key), stored in `~/.askscout/config.json` with file mode 0600
- **Web:** GitHub OAuth for auth, Supabase for storage, hosted API key (no user-side key needed)
- **Three output modes:** digest (default), resume prompt (`--resume`), standup (`--standup`)
- **Project state (CLI):** `.askscout/state.json` per project, schema version 2 with rolling `digestHistory` for Pace Check
- **Quiet days (web):** users get a check-in record on quiet visits to keep streaks alive
- **Web ↔ CLI parity:** the CLI's terminal output is meant to mirror the web's "Copy Digest" markdown byte-for-byte where the same content exists on both. Don't introduce new wording on one surface without updating the other.

## Testing

vitest. Run `pnpm test` for the whole monorepo, or `pnpm --filter <package> test` for a single package. Web app has tests in `apps/web/src/lib/__tests__/`. `packages/core` is currently untested, adding coverage there is the highest-priority unfinished item.

## Where to look first

- **How a digest is generated:** start in `packages/core/src/summarize.ts` (prompts + orchestration). The web's specific generation path is in `apps/web/src/app/api/digest/stream/route.ts` (uses core's prompts but builds its own user prompt against the GitHub API).
- **How output is rendered:** `packages/core/src/format.ts` for the CLI; `apps/web/src/components/DigestView.tsx` for the web.
- **State schema:** `packages/core/src/types.ts` (look for `ProjectState`, `DigestRunSummary`).
- **CLI entrypoint:** `packages/cli/src/index.ts` → `packages/cli/src/commands/scan.ts`.
- **Pre-generation skeletons (web):** `apps/web/src/components/PreGeneration.tsx`.

---
> Source: [charleshonig5/askscout](https://github.com/charleshonig5/askscout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
