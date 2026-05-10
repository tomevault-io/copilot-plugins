## cc-costline

> Enhanced statusline for Claude Code — adds cost tracking, usage limits, and leaderboard rank.

# cc-costline

Enhanced statusline for Claude Code — adds cost tracking, usage limits, and leaderboard rank.

## Tech Stack

- TypeScript (ESM), Node.js >= 22
- Zero runtime dependencies (devDep: `typescript`)
- Tests: `node:test` + `node:assert/strict`

## Commands

```bash
npm test        # Build + run unit tests
npx tsc         # Build only
npm link        # Install locally for testing
npm publish     # Publish to npm
```

## Project Structure

```
src/
├── cli.ts          # CLI entry point (install/uninstall/config/refresh/render)
├── statusline.ts   # Render logic, inline data refresh with unified TTL caching
├── collector.ts    # Scan ~/.claude/projects/**/*.jsonl for token usage
├── calculator.ts   # Per-model pricing and cost calculation
└── cache.ts        # Read/write cost cache and config (~/.cc-costline/)
test/
├── statusline.test.ts  # Unit tests for pure formatting/color functions
├── calculator.test.ts  # Unit tests for pricing lookup and cost calculation
├── cache.test.ts       # Cache/config read/write roundtrip tests
├── collector.test.ts   # Cost collection with mock jsonl files
└── render.test.ts      # Render output format and edge cases
```

## Key Design Decisions

- **TTL-based caching**: All data sources (local cost, usage API, ccclub rank) use a unified 2-minute TTL, refreshed inline during render
- **No User-Agent header**: The Anthropic usage API rate-limits requests with `claude-code` User-Agent
- **Failure caching**: On API failure, a cache entry with null data is written to prevent retry floods
- **Deduplication**: Token cost collection deduplicates by requestId; fallback key includes model + all token types to avoid false dedup
- **Stale fallback**: If API fetch fails or collectCosts returns 0 with existing non-zero cache, retains stale data
- **Safe settings**: `readSettings()` aborts if `settings.json` exists but is invalid JSON, preventing config wipe

## Conventions

- Keep zero runtime dependencies
- All formatting functions should be pure and tested
- Cache files go to `/tmp/sl-*`, config to `~/.cc-costline/`

---
> Source: [Ventuss-OvO/cc-costline](https://github.com/Ventuss-OvO/cc-costline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
