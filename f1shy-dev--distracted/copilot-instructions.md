## distracted

> - Bun monorepo: browser extension + local server.

# CLAUDE.md

## 1) Project Overview (2-3 lines max)
- Bun monorepo: browser extension + local server.
- Purpose: block distracting websites, unlock via challenges.

## 2) Critical Commands
```bash
bun install --frozen-lockfile   # install deps (always use frozen)
bun run check                   # ONE command: lint+format+dead-code+type-aware checks (may rewrite files)
```
- Root scripts are workspace-wide (see @package.json `--workspaces --if-present`).
- There is no separate typecheck step; `bun run check` is the required gate.

## 3) Package-Specific Commands
Extension (@packages/extension)
```bash
cd packages/extension
bun run dev            # dev mode (Chrome)
bun run dev:firefox    # dev mode (Firefox)
bun run build          # production build (Chrome)
bun run build:firefox  # production build (Firefox)
```

Server (@packages/server)
```bash
cd packages/server
bun run dev    # run locally
bun run build  # bundle to dist/
```

## 4) Tech Stack Quick Reference
- Runtime/package manager: Bun (not npm/yarn/pnpm)
- Extension framework: WXT (entrypoints in @packages/extension/src/entrypoints/)
- Server framework: Hono + WebSocket (@packages/server/src/server.ts)
- Linting + type-aware checks: oxlint (not eslint) — config: @.oxlintrc.json
- Formatting: oxfmt (not prettier) — config: @.oxfmtrc.json
- Dead code: knip (via `bun run check`)

## 5) Codebase Structure
```
packages/
├── extension/           # WXT browser extension
│   └── src/
│       ├── entrypoints/ # background, popup, blocked page, content scripts
│       ├── components/  # React components (shadcn/ui pattern)
│       ├── lib/         # shared utilities, storage, challenges
│       └── assets/      # tailwind.css entry
└── server/              # Hono server for Claude Code integration
    └── src/
        ├── bin.ts       # CLI entrypoint
        └── server.ts    # HTTP + WebSocket server
```

## 6) Key Patterns to Follow
- Code quality
  - Avoid premature abstraction: don’t extract helpers only used once. Keep logic inline in the caller until there’s real reuse.
  - Prefer readability over DRY for single-use code.
- Storage
  - Use helpers from @packages/extension/src/lib/storage.ts
  - Types live in @packages/extension/src/lib/storage/types.ts and @packages/extension/src/lib/storage/shared.ts
  - Storage migrations/normalization are in @packages/extension/src/lib/storage/migrations.ts
- Extension APIs
  - WXT abstracts browser APIs — use `browser.*` (never `chrome.*`)
  - Entry points are WXT “entrypoints” (see @packages/extension/src/entrypoints/)
- UI components
  - Follow shadcn pattern in @packages/extension/src/components/ui/
  - Prefer existing primitives/utilities (e.g. `cn`, variants) over rolling new patterns
- Path aliases
  - In extension package, `@/` maps to `packages/extension/src/` (see @packages/extension/tsconfig.json)
- Server
  - CLI entry: @packages/server/src/bin.ts
  - HTTP + WS server: @packages/server/src/server.ts
  - The CLI can install/remove Claude Code hooks on the user machine (see @packages/server/README.md; edits `~/.claude/settings.json`)
- Generated outputs
  - Extension build/dev artifacts: @packages/extension/.wxt/, @packages/extension/.output/
  - Server build artifact: @packages/server/dist/

## 7) Common Pitfalls to Avoid
- DON’T use npm/yarn/pnpm; use Bun and commit @bun.lock updates when deps change
- DON’T use eslint/prettier; use oxlint/oxfmt
- DON’T skip `bun run check` before commits (includes lint/format/dead-code + type-aware checks)
- DON’T access `browser.storage.*` directly; use @packages/extension/src/lib/storage.ts helpers
- DON’T use `chrome.*`; use `browser.*` (WXT handles cross-browser)
- DON’T edit generated directories: @packages/extension/.wxt/, @packages/extension/.output/, @packages/server/dist/

## 8) Workspace Catalog
The root @package.json uses Bun's workspace catalog for shared deps:
```json
"catalog": {
  "typescript": "^5.9.3",
  "oxfmt": "^0.21.0",
  "oxlint": "^1.36.0"
}
```
Reference with `"oxlint": "catalog:"` in workspace package.json files.

---
> Source: [f1shy-dev/distracted](https://github.com/f1shy-dev/distracted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
