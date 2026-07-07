## domestique

> Domestique is a TypeScript MCP server integrating Intervals.icu, Whoop, and TrainerRoad. See @README.md for the project overview, feature list, and full environment variable reference.

# Agent Instructions

Domestique is a TypeScript MCP server integrating Intervals.icu, Whoop, and TrainerRoad. See @README.md for the project overview, feature list, and full environment variable reference.

## Development

Run commands inside the dev container:

```bash
docker compose up                                       # start dev (port 3000, hot reload of src/)
docker compose exec domestique npm run typecheck
docker compose logs domestique -f
```

For commands on the host, run `nvm use` first (version in @.nvmrc).

**On Claude Code Web**, Docker isn't available — run `npm install`, `npm test`, `npm run typecheck`, `npm run build` directly.

## Testing

```bash
nvm use && npm test
```

Tests run on the host (or the test Docker target), **not** in the dev container — the dev container only mounts `src/` and `tsconfig.json`, not `tests/`. Always add tests for new functionality.

## Common Tasks

### Adding a tool

1. Implement in `src/tools/{current,historical,planning}.ts`
2. Define the response shape as a Zod schema in `src/schemas/index.ts` and pass it as `outputSchema` when registering
3. Register in `src/tools/index.ts` (`registerTools`)
4. Add any new API methods to the relevant `src/clients/*.ts`
5. Cover the tool, new client methods, and any new clients with tests
6. Update tool descriptions, field descriptions, and add a one-sentence entry to @README.md (no implementation details, no exhaustive field lists)

### Adding an API client

1. Create the client in `src/clients/`
2. Wire up config in `src/auth/middleware.ts` `getConfig()`
3. Add to `ToolRegistry` constructor in `src/tools/index.ts`

## Code Style

- TypeScript strict mode, async/await, named exports only (no default exports)
- JSDoc on public APIs

### Logging

- Never call `console.*` directly in `src/` (except inside `src/utils/logger.ts` and operator scripts in `src/scripts/`). Use the helpers from `src/utils/logger.ts`: `logInfo` / `logWarn` / `logError` / `logDebug`, plus `logApiCall(source, path, method?)` for outbound API calls and `logToolCall(name)` for MCP tool entry.
- Every line is `[scope] message`; pass the service/module as the scope (e.g. `Whoop`, `Intervals`, `WhoopWebhook`, `CurrentTools`). Don't put the scope in the message — the helper adds the bracket.
- `logError` always emits and reports to Bugsnag (synthesizes an Error if none is passed). Use it only for genuine failures. For best-effort degradations whose underlying API error was already reported at the client layer (`logApiError`), use `logWarn` to avoid double-reporting. `logDebug` is suppressed unless `LOG_LEVEL=debug` — use it for high-volume internal churn (token refresh, lock dance).

### Unit conventions

Tool responses use **unit-in-value** strings.

- Emit `weight: "75.9 kg"`, `max_hr: "165 bpm"`, `recovery_score: "82%"` — **not** `weight_kg: 75.9` or `max_hr: 165`. Field names describe the metric, not the unit.
- Use the helpers in `src/utils/format-units.ts`: `formatPower`, `formatHR`, `formatWeight`, `formatHeight`, `formatStride`, `formatPercent`, `formatTemperature`, `formatLength`, `formatEnergy`, `formatEnergyKJ`, `formatCadence`, `formatMass`, `formatHRV`, `formatVO2max`, `formatBP`, plus `withUnit` for one-offs. `formatStride` for stride length, `formatHeight` only for athlete stature, `formatLength` for elevation/altitude.
- Field descriptions in `.describe()` must **not** name units. Describe the metric (`'Average heart rate'`), not the unit. Formatters consult athlete prefs via `runWithUnitPreferences` in `src/utils/unit-context.ts`, so values change per user — unit-named descriptions go stale.
- Bare numerics only for unitless scores (TSS, CTL/ATL/TSB, IF, RPE, 1–4 wellness scales) and counts (`activities`, `lengths`, `steps`).

## MCP Compatibility

- **`outputSchema` required for every tool.** Define the shape in `src/schemas/index.ts`. Handlers return a JSON object — wrap arrays at the registration site, e.g. `{ strain: await this.currentTools.getStrainHistory(args) }`. The return becomes `structuredContent`; it's also serialized into `content` for older clients. Field descriptions live in `.describe()` and ship via `tools/list`, not in every response.
- **No underscore-prefixed property names in `outputSchema`.** MCP Inspector treats `_*` as protocol-private and strips them, then flags the value as an unexpected additional property. Use plain names. `_meta` is reserved for genuine out-of-band metadata.
- **No MCP resources or elicitations.** ChatGPT doesn't support resources; neither client supports elicitations. Don't suggest either. Before adding any non-tool MCP feature, check https://modelcontextprotocol.io/clients.md.

## Project-specific gotchas

- **Whoop OAuth.** Tokens are stored in Redis and refreshed automatically. Initial setup is interactive: `docker compose exec domestique npm run whoop:auth`.
- **Tool registry is shared** across MCP sessions (created once at server start), but each session gets its own `McpServer`.
- **Location updates bust memoized athlete caches.** The `update_location` tool and `PUT /api/location` share `applyLocation` (`src/services/location-sync.ts`); it writes the profile + weather-config then calls `IntervalsClient.invalidateAthleteCaches()` (the memo in `src/utils/memo.ts` has no clear method, so skipping this leaves stale timezone/weather on the long-running server).
- **`/api` endpoints are paired with tools, not webhooks.** `PUT /api/location` and `POST /api/activities/descriptions` (`src/api/`) authenticate via `Authorization: Bearer` against `config.apiSecret` **only** — no query-string auth or inputs. The descriptions endpoint and the `regenerate_descriptions` tool share `regenerateDayDescriptions` (`src/services/description-regen.ts`); the endpoint is fire-and-forget (`202`), the tool awaits and returns the result. Keep an endpoint and its tool behaviorally in sync.
- **TrainerRoad annotation categorization.** With `ANTHROPIC_API_KEY` set, `src/utils/annotation-categorizer.ts` classifies TR annotations into Sick/Injured/Holiday/Note via Claude Haiku 4.5 (override with `ANTHROPIC_CLASSIFIER_MODEL`), cached in Redis by content hash. `mergeAnnotations` (`src/utils/annotation-utils.ts`) dedupes TR vs. Intervals.icu.
- **Race sources are split by sport.** Single-discipline races come from Intervals.icu (`IntervalsClient.getRaces`, `category=RACE_A,RACE_B,RACE_C` — priority is native). Triathlons come from TrainerRoad; `src/utils/race-priority-classifier.ts` extracts A/B/C from the umbrella description via Haiku and returns `null` rather than guessing. `mergeRaces` (`src/utils/race-utils.ts`) merges by `name + sport + date`; ICU wins non-tri collisions, TR wins for `sport: 'Triathlon'`.
- **`create_workout` / `update_workout` convert structure server-side.** Tools take a plain-language `structure` field; `src/utils/workout-generator.ts` calls Claude (Sonnet by default, override with `ANTHROPIC_WORKOUT_MODEL`) using the sport-specific system prompts in `src/prompts/` to emit Intervals.icu workout-doc syntax. Swimming bypasses the LLM and stores `structure` verbatim because Intervals.icu has no structured swim syntax. `ANTHROPIC_API_KEY` is therefore required for these tools on cycling and running. The prompts are bundled — `npm run build` copies `src/prompts/` to `dist/prompts/`.

---
> Source: [gesteves/domestique](https://github.com/gesteves/domestique) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
