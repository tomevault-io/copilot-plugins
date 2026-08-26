## mortang-meals

> Read this before changing the app. It describes the **current** code, not the original v1 spec.

# Mortang Meals — agent guide

Read this before changing the app. It describes the **current** code, not the original v1 spec.

`docs/superpowers/` is a historical design and implementation plan from 2026-08-12. It is stale: it lists web import and a recipe library as non-goals, says generate writes a new current plan, and treats Kitchen as an appliance checklist only. Do not implement from those files.

## What this is

A local, single-household meal planner. The user describes who they cook for and how they cook; the app generates a week of recipes, keeps a meal library, and derives a shopping list.

No auth, no multi-household, no hosted deploy. One Next.js process. The browser never calls an AI provider.

Success path: set up household + kitchen → pick slots on This Week → generate → cook from a card → pin keepers / swap duds / place from the library → shop from the merged list.

## Stack and commands

- Next.js 15 App Router (`src/`), React 19, TypeScript, Tailwind 4
- SQLite via `better-sqlite3` + Drizzle (queries only — schema is created in `src/lib/db.ts`, not drizzle-kit)
- Zod for AI JSON and HTTP bodies
- OpenAI SDK against xAI (`https://api.x.ai/v1`) or a custom OpenAI-compatible base URL
- Vitest. Component tests set `// @vitest-environment happy-dom`

```
npm run dev     # localhost:3000
npm test        # vitest run
npm run build
```

Env: copy `.env.example` to `.env.local` and set `XAI_API_KEY`. Optional `MORTANG_DB_PATH` overrides the SQLite file (default `data/mortang.db`, gitignored). Tests should set `MORTANG_DB_PATH` to a temp file and call `resetDbForTests()` when they open the db.

`src/app/layout.tsx` is `force-dynamic`. `next.config.ts` marks `better-sqlite3` as a server external package.

## Hard constraints

- Browser talks only to local API routes / server actions. Keys stay on the server.
- Grok key is `process.env.XAI_API_KEY` only. Never store it in SQLite. Custom-provider keys may live in settings.
- Automated tests mock the adapter (`complete`). No live model calls.
- One household. Week is Monday–Sunday (`mondayOf` in `src/lib/week.ts`). Slots are `breakfast | lunch | dinner` (21 cells).
- At most one plan has `isCurrent = 1`.
- Last good plan / meal is never replaced by a failed generate, swap, or import.
- Ingredient `quantity` is a **string** (`"1"`, `"1/2"`, `"1/4"`). Never a number. Never `0` for a used ingredient.
- Duplicate = normalized title match only (`src/meals/duplicates.ts`: lowercase, strip non-alphanumerics, collapse spaces). No fuzzy matching.
- Allergen = case-insensitive substring of an ingredient **name** (`src/meals/allergen.ts`).
- Generate and swap retry **once** on transport / invalid JSON / schema / allergen / duplicate. Then keep the previous data.
- AI traces: always record, keep last 25, redact `Bearer` tokens and `api_key=` values. Developer nav is hidden unless Settings → developer tools is on.
- Visual language lives in `src/app/globals.css` (olive / linen / paper). Match existing components; do not invent a parallel design system.

## Architecture

```
UI (server pages + client components)
    │  fetch / server actions
    ▼
HTTP handlers
    src/ai/http.ts        generate, swap, settings, traces
    src/meals/http.ts     library, place, pin, import, update, delete
    │
    ├── domain (pure, easy to test)
    │     brief, schema, allergen, duplicates, shopping-list, slot-mask, catalog
    ├── repos (SQLite)
    │     household, kitchen, prefs, meals, settings, traces
    └── adapter
          src/ai/adapter.ts  → xAI / custom OpenAI-compatible endpoint
```

Shared types: `src/lib/types.ts`. Drizzle tables: `src/lib/schema.ts`. Schema bootstrap + additive columns: `src/lib/db.ts` (`ensureSchema` + `ensureColumn`). There are no foreign keys.

Thin `src/app/api/*/route.ts` files parse JSON and call a handler. Keep logic in the handler modules so tests can call them without Next.

## Screens

| Route | Role |
| --- | --- |
| `/setup` | First-run wizard: household → kitchen checklist → slot mask. Redirect target when there is no household or no named people. |
| `/` This Week | Home. Slot picker, generate, pin all, “use what I have” ingredients, week grid, recipe flyout, library flyout. `?plan=` opens a historical plan. |
| `/meals` | Library catalog: search / filter / group, import-from-URL form. |
| `/meals/[id]` | Full recipe editor (title, why, time, method, ingredients, steps). Swap only if the meal is on the current plan. |
| `/shopping-list` | Derived list for the open plan (`?plan=` supported). Not stored. |
| `/household` | People, leftovers of diet style/notes/servings. |
| `/kitchen` | Cook prefs + appliance/method checklist. |
| `/settings` | Provider mode, base URL, model, optional custom key, web search toggle, developer tools. |
| `/developer` | Last 25 AI traces. Hidden unless the toggle is on. |

Nav: This Week, Meals, Shopping list, Household, Kitchen, Settings, optional Developer (`src/components/nav.tsx`).

Generation UX is global (`GenerationProvider` in `AppShell`): NDJSON stream. Modal on `/` and `/meals*`; compact banner on other routes. Cancel aborts the fetch.

## Data model

**Household** — one row. Name, `dietStyle` (legacy / fallback), notes, servings, people.

**Person** — name, age, optional sex, allergies (hard exclude), avoidances (soft prefer-to-skip). Blank-name people are dropped on save (`normalizePeople`).

**Kitchen item** — appliance or method. Built-ins in `src/kitchen/defaults.ts` (crockpot, air fryer, Instant Pot, oven, stovetop, sheet pan, grill). Only `enabled` items go into the brief. Seeded on first generate if empty.

**Kitchen prefs** — one row (`id = default`). Expertise `newbie | novice | intermediate | expert`, involved `low | medium | high`, `maxCookMinutes` (floor 5, default 45), `overallDiet`, plus per-slot diets. Slot diet resolution (`resolvedDiet`): slot field → overall kitchen diet → household `dietStyle`. Generate is allowed if **either** household diet **or** kitchen overall diet is non-empty.

**Week plan** — `weekStart` (Monday `YYYY-MM-DD`), `isCurrent`, `slotMask` JSON. History stays readable from This Week / shopping list.

**Meal** — belongs to a plan **or** stands alone. Fields: day, slot, title, whyItFits, cookMinutes, method, ingredients[], steps[], `usedWebSearch`, `pinned`, `weekStart`, `createdAt`, optional `sourceUrl`. Imported meals are saved with `planId = ""`. Deleting a plan deletes the plan row only; meals stay so the library keeps the recipes.

**UseIngredient** — `{ name, day, slot }`. Session-only (`sessionStorage` key `mortang.useIngredients`). Instructs generate/swap that that slot must feature that ingredient. Cleared after a successful generate.

**Slot mask** — which of the 21 cells are requested. Default: all dinners on (`defaultSlotMask`). Also session-backed (`mortang.slotMask`). Pinned slots are treated as locked in the picker (`maskMinusPinned`, `toggleSlot` / `toggleDay` / `toggleMealRow`).

**Settings** — `mode: grok | custom`, `baseUrl` (default `https://api.x.ai/v1`), `model` (default `grok-4.6`), optional `customApiKey`, `developerTools`, `webSearch`. API responses expose `customApiKey` as a boolean only.

**AI trace** — kind `generate | generate-retry | swap | swap-retry | test`, request/response text, validation `ok | invalid-json | schema | allergen | duplicate | transport`. Import is **not** traced today.

## Core flows

### Generate

1. UI requires named people and a diet (household or kitchen overall). Slot picker must have at least one unpinned cell on.
2. `GenerateButton` sends the mask **already minus pinned slots**, plus `useIngredients`, plus optional `weekStart`.
3. `handleGenerate` (`src/ai/http.ts`) re-strips pinned slots. If nothing remains: `"Everything you asked for is pinned."` Use-ingredients aimed at pinned slots are dropped. Reserved titles = pinned meal titles.
4. `generateWeekPlan` builds a brief (`buildHouseholdBrief`) + hard rules + optional web-search rules + cook-time cap. Asks only for the effective slots. Validates: JSON → Zod meal schema → exact slot set → allergens → no duplicate titles inside the batch **or** vs reserved titles. One retry, then fail. Progress phases: `brief → calling → validating → [retry] → saving`.
5. On success, `mergeGeneratedPlan`:
   - No current plan → `saveGeneratedPlan` (new current row).
   - Else update that plan’s slot mask and replace **unpinned** occupants only. Pinned rows are left untouched. New meals get new ids.
6. Generate **always writes the current plan**, even if the user is viewing `?plan=` history. `weekStart` from the request is used only when creating the first plan.

Do not revive “save a brand-new plan on every generate.” Pins and the library depend on merge.

### Swap

`POST /api/swap` `{ planId, mealId, useIngredients? }`. Same brief, but only the use-ingredient for that day/slot, plus a do-not-repeat list of the current title and every other title on that plan. Response is `{ meal: ... }`. `replaceMeal` keeps the same meal id, `pinned`, `createdAt`, `sourceUrl`, and `weekStart`. Failure leaves the card as-is.

Swap is not streamed. The flyout SwapButton is a single request + `router.refresh()`.

### Pin

`POST /api/pin` `{ pinned, mealId }` or `{ pinned, planId }` (exactly one id). Pinned meals survive generate. The slot picker locks those cells. Pin/replace/use-ingredient UI is only offered when the open plan is current (`editable = !plan || plan.isCurrent`).

### Place from library

Empty or filled cell on the current week opens `MealLibraryFlyout` → `GET /api/library?slot=` (unique titles for that slot, newest week first) → `POST /api/place` `{ sourceMealId, day, slot, weekStart? }`. Copies recipe fields onto the current plan (creates an empty current plan if none). If a meal already occupies the cell, that row is overwritten and its pin flag is kept.

### Import from URL

Meals page form → `POST /api/import` NDJSON stream. Always uses the Grok adapter **with web search on**, regardless of Settings mode. Model reads the page and returns one meal. Saved via `saveImportedMeal` (`planId ""`, `sourceUrl` set, `usedWebSearch true`). Not placed on the week until the user places it.

### Edit / delete

`POST /api/update` uses `mealEditSchema` (no day/slot). `POST /api/delete` `{ mealId }` removes the row. `POST /api/plans/delete` `{ planId }` removes the plan, keeps meals.

### Shopping list

`mergeShoppingList` (`src/meals/shopping-list.ts`) from the **open** plan’s meals. Normalize names (lowercase, naive English plural strip). Merge quantities when name + unit match and both quantities parse (`1`, `1/2`, `1 1/2`, decimals). Non-numeric quantities do not merge. Group by aisle: produce, meat, dairy, pantry, other.

## AI adapter

`src/ai/adapter.ts` → `createAdapter(settings)`.

| Mode | Key | Protocol |
| --- | --- | --- |
| `grok` + `webSearch` | `XAI_API_KEY` | `client.responses.create` with `tools: [{ type: "web_search" }]` and `text.format = json_schema` |
| `grok` without search | `XAI_API_KEY` | `chat.completions` + `response_format.json_schema` (strict) |
| `custom` | optional `customApiKey` | `chat.completions` + `response_format.json_object`; we still Zod-parse |

`grokWebSearchEnabled` is `mode === "grok" && webSearch`. Web search is a Grok-only setting. Meals found that way get `usedWebSearch` and a star badge.

JSON shapes (`src/meals/schema.ts`): generate `{ meals: Meal[] }`, swap/import `{ meal: Meal }`. Each meal: `day`, `slot`, `title`, `whyItFits`, `cookMinutes`, `method`, `ingredients[]` (`name`, `quantity` string, `unit`, `aisle`), `steps[]`.

Brief (`src/household/brief.ts`) includes people, diet, notes, allergies, avoidances, enabled kitchen items, expertise/involved/time, per-slot diets, requested slots, use-ingredients, servings, extra rules (do-not-repeat).

## Module map

| Path | Responsibility |
| --- | --- |
| `src/lib/types.ts` | Domain types and constants (`DAYS`, `SLOTS`, `AISLES`) |
| `src/lib/db.ts` | Open SQLite, create/alter tables, test reset |
| `src/lib/schema.ts` | Drizzle table defs |
| `src/lib/slot-mask.ts` | Mask helpers + session persistence |
| `src/lib/use-ingredients.ts` | Session persistence for assigned ingredients |
| `src/lib/generate-progress.ts` | Progress % / step labels for generate and import |
| `src/lib/week.ts` | `mondayOf` |
| `src/household/*` | Household repo, brief, people normalize |
| `src/kitchen/*` | Items repo, prefs repo, built-in defaults |
| `src/meals/schema.ts` | Zod + JSON Schema for model output |
| `src/meals/repo.ts` | Plans and meals persistence (merge, place, pin, library, import) |
| `src/meals/http.ts` | Library / pin / place / import / update / delete handlers |
| `src/meals/catalog.ts` | Search / filter / group for `/meals` |
| `src/meals/{allergen,duplicates,shopping-list}.ts` | Pure validators / list merge |
| `src/ai/adapter.ts` | Provider client |
| `src/ai/generate-plan.ts` | Generate loop + validation |
| `src/ai/swap-meal.ts` | Swap loop + validation |
| `src/ai/http.ts` | Generate / swap / settings / traces handlers |
| `src/ai/settings-repo.ts` | Settings row |
| `src/ai/traces.ts` | Trace log |
| `src/components/this-week-planner.tsx` | This Week client orchestrator |
| `src/components/generation-provider.tsx` | Shared generate/import stream client |
| `src/components/meals-catalog.tsx` | Library UI |
| `src/components/meal-detail.tsx` | Recipe editor |
| `src/app/api/smoke.test.ts` | HTTP-level smoke tests with a fake adapter |

## API

All mutating meal/AI routes are `POST` JSON unless noted. Generate and import respond with `application/x-ndjson` (`{type:progress|done|error}` lines).

| Route | Handler |
| --- | --- |
| `POST /api/generate` | `handleGenerate` — stream |
| `POST /api/swap` | `handleSwap` |
| `POST /api/import` | `handleImportRecipe` — stream |
| `GET /api/library?slot=` | `handleListLibrary` |
| `POST /api/place` | `handlePlaceMeal` |
| `POST /api/pin` | `handlePin` |
| `POST /api/update` | `handleUpdateMeal` |
| `POST /api/delete` | `handleDeleteMeal` |
| `POST /api/plans/delete` | `handleDeletePlan` |
| `GET/PUT /api/settings` | settings (PUT body is a patch; key never echoed) |
| `POST /api/settings/test` | tiny `pong` call + `test` trace |
| `GET/DELETE /api/traces` | list / clear |

Household and kitchen writes are server actions (`src/app/household/actions.ts`, `src/app/kitchen/actions.ts`), not REST.

## UI behavior worth keeping

- Desktop This Week is a 7-column grid (days as columns, B/L/D as rows). Narrow viewports stack by day. Empty cells are dashed; on the current plan they open the library flyout.
- Recipe cards are a flyout, not a navigation, except “Open full recipe”.
- Star badge = web search. Arrow badge = imported (`sourceUrl`).
- Historical plans are view-only for pin/place/use-ingredient. Generate still targets the current plan.
- Session slot mask and use-ingredients survive in-tab navigation.

## How to change things

- New generate/swap constraint → brief line and/or Zod + the retry loop. Add a unit test next to the domain function. Do not put prompt-only rules that the server cannot enforce if they matter (allergies, slots, duplicates, cook time if you start enforcing it).
- New meal field → types, drizzle table + `ensureColumn`, `mealInsertValues` / `mapMeal`, Zod + JSON Schema, UI.
- New API → handler in `src/ai/http.ts` or `src/meals/http.ts`, thin route file, test in `src/app/api/smoke.test.ts` or a focused `*.test.ts`.
- New screen → `src/app/.../page.tsx`, add a nav link if it is first-class, keep the olive/linen styles.
- AI provider changes → `adapter.ts` only if possible. Keep `complete({ messages, jsonSchema, schemaName, signal })` so tests stay fakeable.

Cook time is currently a **prompt** rule (`Keep cookMinutes at or under N`). It is not rejected in validation. Slot diets and expertise/involved are prompt-only too. Allergies, requested slots, and duplicate titles are enforced.

## Testing

`npm test`. Prefer extending existing tests over new runners.

- Domain: `src/**/*.test.ts` next to the module.
- UI: `*.test.tsx` with happy-dom.
- HTTP: `src/app/api/smoke.test.ts` and `src/meals/http.import.test.ts` inject `deps.complete`.

When you add a generate/swap/import validation path, cover the retry-then-fail case and assert the previous plan/meal is unchanged.

---
> Source: [wmfmorgan/mortang-meals](https://github.com/wmfmorgan/mortang-meals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
