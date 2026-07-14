## bakebook

> > Personal baking PWA. Self-hosted on the existing Proxmox + Coolify + Cloudflare Tunnel stack.

# Bakebook — CLAUDE.md

> Personal baking PWA. Self-hosted on the existing Proxmox + Coolify + Cloudflare Tunnel stack.
> Working name: **Bakebook**. Rename freely (`pnpm rename <new>` script provided in M0).

> **Read order:** this file → `DESIGN.md` → `phases/M{current}/task.md`. Do not skip ahead between phases.

---

## 1. What this is

A single-user baking companion that runs as an installable PWA on phone and desktop. It does five things, in priority order:

1. **Holds recipes** as structured data — ingredients with grams, ordered steps, optional per-step timers and target temperatures. Per-recipe macros, cost, full baker's ratios.
2. **Runs an active bake** — full-screen, large-tap, step-by-step mode with timers that survive backgrounding and a Wake Lock so the screen stays on.
3. **Captures structured outcomes** after each bake — rating, photos, measurements (rise, internal temp, weight loss), crumb/crust scores, free-text notes, and structured "tweaks" (e.g. *+10 g water, dough was tight*) that surface on the recipe next time.
4. **Versions recipes** — every meaningful change either edits in place (with audit trail) or forks. Lineage is visible. Tweaks one-click apply to a new version.
5. **Surfaces patterns** in an Insights screen once enough bakes have been logged — bakes per month, common tweaks, equipment success rates, calendar heatmap.

Non-goals (do not build in v1):
- Social / sharing / public profiles.
- Recipe scraping from URLs (manual entry only — the structure is the value).
- Meal planning, calendars, calorie tracking as a primary lens.
- Multi-user accounts. Single user, single household.
- Native iOS/Android apps. PWA only.
- Pantry inventory / restock alerts (deferred to v2).

---

## 2. Tech stack

Standard Nicolas-stack with no surprises. Every choice below is fixed; do not substitute without a written ADR in `/decisions/`.

| Layer | Choice | Notes |
|---|---|---|
| Frontend | Next.js 14 (App Router) + TypeScript + Tailwind + shadcn/ui | Mobile-first. Use `next-pwa` for service worker + manifest. |
| State | Zustand for client UI state, TanStack Query for server state | No Redux. |
| Backend | FastAPI (Python 3.12) | Pydantic v2 models. |
| DB | PostgreSQL 16 | One schema, no multi-tenancy. |
| ORM | SQLAlchemy 2.0 + Alembic | Async session. |
| Object storage | Cloudflare R2 | Photos. Signed URLs for read, presigned PUT for upload. |
| Auth | Cloudflare Access in front of the Tunnel | No app-level auth in v1. The Tunnel is the perimeter. App reads `Cf-Access-Authenticated-User-Email` header. |
| Deploy | Coolify on Proxmox | Two services: `web` (Next.js) and `api` (FastAPI). One Postgres. |
| Edge | Cloudflare Tunnel → `bakebook.nicolasschaerer.ch` | TLS terminated at Cloudflare. |
| Tests | Vitest (unit), Playwright (e2e + screenshots), Pytest (api) | Playwright is the source of truth for "feature done." |
| Package mgr | pnpm (web), uv (api) | |

**Deployment fallback noted, not chosen:** plain docker-compose on a Proxmox VM is viable if Coolify becomes annoying. If you switch, document it in `/decisions/0001-deployment.md` — the rest of the spec is Coolify-agnostic.

---

## 3. Architecture

```
┌─────────────────────┐
│   iPhone / Desktop  │  PWA, installable, offline reads
└──────────┬──────────┘
           │  HTTPS via bakebook.nicolasschaerer.ch
┌──────────▼──────────┐
│  Cloudflare Tunnel  │  + Cloudflare Access (email-restricted to me)
└──────────┬──────────┘
           │
┌──────────▼──────────┐    ┌──────────────────┐
│  Next.js (web)      │───▶│  FastAPI (api)   │
│  Coolify container  │    │  Coolify cont.   │
└─────────────────────┘    └────────┬─────────┘
                                    │
                          ┌─────────▼─────────┐    ┌───────────────┐
                          │  Postgres (cool.) │    │  Cloudflare R2│
                          └───────────────────┘    └───────────────┘
```

Two long-lived browser concerns:

- **Service worker** caches static shell + last-viewed recipes for offline reads. Network-first for `/api/*`, stale-while-revalidate for recipe GETs.
- **Active bake worker** (a dedicated Web Worker, not the SW) holds timer state in IndexedDB so a tab refresh or backgrounding doesn't lose timers.

Bundled at build time:
- `apps/api/data/nutrition.json` — ~80 baking ingredients with per-100g macros, derived once from USDA FoodData Central via a build script. No external API at runtime.

---

## 4. Data model

Postgres. All `id`s are `uuid v7` (time-ordered). All timestamps `timestamptz`. Soft-delete via `deleted_at` on user-facing tables.

### 4.1 Recipes (versioned)

```sql
create table recipes (
  id              uuid primary key,
  version_of_id   uuid references recipes(id),     -- null for v1
  parent_recipe_id uuid references recipes(id),    -- null unless this is a fork
  version_number  int not null default 1,
  title           text not null,
  slug            text not null,
  category        text not null,                   -- 'bread' | 'sweet' | 'quick' | 'pizza' | 'other'
  summary         text,
  yields          text,                            -- '1 loaf, ~900 g' (free text)
  servings        int not null default 1,          -- for per-serving math
  serving_g       numeric(7,2),                    -- weight per portion (nullable)
  total_time_min  int,
  active_time_min int,
  difficulty      int check (difficulty between 1 and 5),
  equipment       text[] default '{}',
  hero_photo_key  text,                            -- R2 key
  source          text,                            -- 'mom' | URL | book reference
  notes           text,
  target_dough_g  numeric(7,2),                    -- for "scale to dough weight" mode
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now(),
  deleted_at      timestamptz
);

create index on recipes (version_of_id, version_number desc);
```

### 4.2 Pantry items (cost + nutrition reference)

```sql
create table pantry_items (
  id              uuid primary key,
  name            text unique not null,             -- 'Zopfmehl', 'Bread flour', 'Fresh yeast'
  cost_per_kg     numeric(7,2),                     -- CHF, nullable
  cost_currency   text not null default 'CHF',
  nutrition_ref   text,                             -- key into nutrition.json, nullable
  default_role    text,                             -- pre-filled when added to a recipe
  created_at      timestamptz not null default now(),
  updated_at      timestamptz not null default now()
);

create table pantry_price_history (
  id              uuid primary key,
  pantry_item_id  uuid not null references pantry_items(id) on delete cascade,
  cost_per_kg     numeric(7,2) not null,
  observed_at     timestamptz not null default now(),
  source          text                              -- 'manual' | 'migros' | 'coop' | 'import'
);

create index on pantry_price_history (pantry_item_id, observed_at desc);
```

### 4.3 Ingredients & steps

```sql
create table recipe_ingredients (
  id              uuid primary key,
  recipe_id       uuid not null references recipes(id) on delete cascade,
  pantry_item_id  uuid references pantry_items(id),  -- nullable; free-typed names allowed
  ord             int not null,
  group_label     text,
  name            text not null,
  grams           numeric(8,2),
  unit_display    text,
  unit_display_qty numeric(8,2),
  role            text,                              -- 'flour'|'water'|'salt'|'leaven'|'fat'|'sugar'|'egg'|'dairy'|'other'
  leaven_flour_pct numeric(5,2) default 50,          -- only meaningful when role='leaven'
  cost_override_per_kg  numeric(7,2),                -- nullable; otherwise inherit from pantry_item
  nutrition_override    jsonb,                       -- {kcal, protein, fat, carbs, sugar, fiber, salt} per 100g
  notes           text,
  optional        boolean not null default false
);

create index on recipe_ingredients (recipe_id, ord);

create table recipe_steps (
  id              uuid primary key,
  recipe_id       uuid not null references recipes(id) on delete cascade,
  ord             int not null,
  title           text not null,
  body            text not null,
  timer_seconds   int,
  min_seconds     int,                              -- for ready-by calc; nullable
  max_seconds     int,                              -- for ready-by calc; nullable
  target_temp_c   numeric(5,1),
  temp_kind       text,                             -- 'oven' | 'internal' | 'dough' | 'water'
  ingredient_ids  uuid[] default '{}',
  parallelizable_with int[] default '{}'            -- list of step ords this can run during
);

create index on recipe_steps (recipe_id, ord);
```

### 4.4 Bakes (with structured outcomes)

```sql
create table bakes (
  id              uuid primary key,
  recipe_id       uuid not null references recipes(id),
  started_at      timestamptz not null default now(),
  finished_at     timestamptz,
  status          text not null default 'active',   -- 'active' | 'finished' | 'abandoned'
  current_step    int not null default 0,
  scale_multiplier numeric(6,3) not null default 1,

  -- environmental context (captured at start, optional)
  kitchen_temp_c    numeric(4,1),
  kitchen_humidity  numeric(4,1),
  flour_brand       text,                           -- free-text, in case it changes behavior

  -- structured outcomes (captured at reflection)
  rating          int check (rating between 1 and 5),
  outcome         text,                              -- 'disaster'|'meh'|'okay'|'good'|'best_yet'
  start_weight_g   numeric(7,2),
  final_weight_g   numeric(7,2),
  rise_height_cm   numeric(4,1),
  internal_temp_c  numeric(4,1),
  crumb_score      int check (crumb_score between 1 and 5),
  crust_score      int check (crust_score between 1 and 5),
  notes           text,
  context         jsonb default '{}'                 -- catch-all for anything else
);

create index on bakes (recipe_id, started_at desc);
create index on bakes (status) where status = 'active';
```

Computed (not stored) in API responses:
- `water_loss_pct = (start_weight_g - final_weight_g) / start_weight_g * 100`
- `oven_spring_pct` — computed at finish from rise_height_cm vs pre-bake reference if recorded

### 4.5 Photos & tweaks

```sql
create table bake_photos (
  id              uuid primary key,
  bake_id         uuid not null references bakes(id) on delete cascade,
  r2_key          text not null,
  caption         text,
  kind            text,                              -- 'process'|'crumb'|'final'|'oven'|'side'|'top'
  step_ord        int,
  taken_at        timestamptz not null default now()
);

create table bake_tweaks (
  id              uuid primary key,
  bake_id         uuid not null references bakes(id) on delete cascade,
  ingredient_id   uuid references recipe_ingredients(id),
  step_id         uuid references recipe_steps(id),
  change          text not null,
  reason          text,
  apply_next_time boolean not null default false,
  resolved_in_recipe_id uuid references recipes(id)  -- set when user "applies" the tweak to a new version
);

create index on bake_tweaks (bake_id);
create index on bake_tweaks (apply_next_time) where apply_next_time and resolved_in_recipe_id is null;
```

### 4.6 Starters

```sql
create table starters (
  id              uuid primary key,
  name            text not null,
  hydration_pct   numeric(5,1) not null default 100,
  peak_base_hours numeric(4,1) not null default 6,   -- at 20°C, tunable per starter
  notes           text,
  created_at      timestamptz not null default now()
);

create table starter_feedings (
  id              uuid primary key,
  starter_id      uuid not null references starters(id) on delete cascade,
  fed_at          timestamptz not null default now(),
  ratio           text,                              -- '1:5:5'
  flour_mix       text,
  kitchen_temp_c  numeric(4,1),
  peak_at         timestamptz,
  notes           text
);

create index on starter_feedings (starter_id, fed_at desc);
```

### 4.7 Migration discipline

- One Alembic migration per logical change. Never edit a migration that has been applied to prod.
- Seed data lives in `apps/api/seed/` as idempotent SQL or Python scripts (use `on conflict do nothing` keyed on slug+version, or unique-name for pantry items).
- The nutrition table is built by `apps/api/scripts/build-nutrition-table.py` and committed as `apps/api/data/nutrition.json`. Do not regenerate on every deploy.

---

## 5. API surface

REST, JSON, FastAPI. All routes under `/api/v1`. camelCase in JSON, snake_case in Python.

### 5.1 Recipes

| Method | Path | Purpose |
|---|---|---|
| GET | `/recipes` | List. Query: `?category=&q=&include_versions=false`. |
| GET | `/recipes/{id}` | Full recipe — includes computed `nutrition`, `cost`, `ratios`. |
| GET | `/recipes/by-slug/{slug}` | Latest non-deleted version of slug. |
| POST | `/recipes` | Create. |
| PUT | `/recipes/{id}` | Edit in place (only if no `bakes` reference, else 409). |
| POST | `/recipes/{id}/version` | Create new version. |
| POST | `/recipes/{id}/fork` | Create fork. |
| DELETE | `/recipes/{id}` | Soft delete. |
| POST | `/recipes/{id}/scale` | Scaled ingredient list. Body: `{mode: 'multiplier'\|'doughWeight'\|'flourWeight', value}`. **Pure function.** |
| GET | `/recipes/{id}/ready-by?target=ISO8601` | Reverse calc. Returns `{startAt, expectedEnd, rangeStartAt, rangeEndAt}`. |
| GET | `/recipes/{id}/pending-tweaks` | All unresolved `apply_next_time` tweaks across past bakes. |
| GET | `/recipes/{id}/bakes` | History. |

### 5.2 Pantry

| Method | Path | Purpose |
|---|---|---|
| GET | `/pantry` | List items with current cost + nutrition_ref. |
| POST | `/pantry` | Create. |
| PATCH | `/pantry/{id}` | Update. Triggers price-history insert if cost changed. |
| GET | `/pantry/{id}/price-history` | Time series. |
| GET | `/nutrition-table` | Returns the bundled JSON, for client-side preview during recipe edit. |

### 5.3 Bakes

| Method | Path | Purpose |
|---|---|---|
| POST | `/bakes` | Start. Body: `{recipeId, scaleMultiplier?, kitchenTempC?, kitchenHumidity?, flourBrand?}`. |
| GET | `/bakes/{id}` | Get one — includes computed `waterLossPct`. |
| PATCH | `/bakes/{id}` | Update (current_step, status, all outcome fields). |
| POST | `/bakes/{id}/photos` | Returns `{r2Key, presignedUrl, expiresIn}`. |
| POST | `/bakes/{id}/photos/{r2Key}/confirm` | Create the DB row after upload. |
| PATCH | `/bake-photos/{id}` | Update caption/kind/step_ord. |
| DELETE | `/bake-photos/{id}` | Delete (also from R2). |
| POST | `/bakes/{id}/tweaks` | Add tweak. |
| POST | `/recipes/{recipeId}/tweaks/apply` | Body: `{tweakIds: [...]}`. Creates new version with tweaks applied; marks tweaks resolved. |
| GET | `/bakes` | Journal feed. Query: `?from=&to=&category=&minRating=&hasTweaks=`. |

### 5.4 Insights

| Method | Path | Purpose |
|---|---|---|
| GET | `/insights/summary?range=month\|year\|all` | Stats strip: bakes count, avg rating, flour kg, total cost, deltas vs previous range. |
| GET | `/insights/bakes-per-month?months=12` | Bar chart data. |
| GET | `/insights/top-tweaks?limit=10` | Ranked changes across all bakes, with counts. |
| GET | `/insights/equipment` | Per-equipment bake count + avg rating. |
| GET | `/insights/calendar?year=YYYY` | 365-day intensity array for heatmap. |
| GET | `/insights/patterns` | Hand-coded rule-based insights (see §10). |

### 5.5 Starters

| Method | Path | Purpose |
|---|---|---|
| GET | `/starters`, POST, PATCH | Manage. |
| POST | `/starters/{id}/feedings` | Log. |
| GET | `/starters/{id}/status` | `{lastFedAt, hoursSinceFed, estimatedPeakAt}` from peak model. |

### 5.6 System

| Method | Path | Purpose |
|---|---|---|
| GET | `/health` | Liveness. |
| GET | `/me` | Returns email from CF Access header (or `null` in dev). |

Auth: every route reads `Cf-Access-Authenticated-User-Email`. In dev (`ENV=dev`), inject `dev@local`. Reject if missing in prod.

---

## 6. Pure computation services

These live in `apps/api/services/` and have full unit tests. No DB writes.

### 6.1 Scaling (`scaling.py`)

Three modes:
1. `multiplier` — multiply every ingredient gram by `value`.
2. `doughWeight` — `multiplier = value / sum(grams)`, then mode 1.
3. `flourWeight` — `multiplier = value / sum(grams where role='flour')`. Error if no flour.

### 6.2 Ratios (`ratios.py`)

When at least one ingredient has `role='flour'`:

- **Flour total** = sum of all flour-roled grams + leaven-roled grams × `(leaven_flour_pct / 100)`.
- **Hydration** = `(sum where role='water') / flourTotal * 100`.
- **Hydration with dairy** — treat `role='dairy'` as 90% water, add to numerator. Display both.
- **Salt %, sugar %, fat %** — straightforward `sum(grams where role=X) / flourTotal * 100`.
- **Prefermented flour %** — flour contributed by leaven / flourTotal * 100.
- **Inoculation rate** — `sum(grams where role='leaven') / flourTotal * 100`.

If no flour roles, return `null` for all — UI hides the panel entirely.

### 6.3 Macros (`nutrition.py`)

For each ingredient:
1. If `nutrition_override` is set, use it (per 100 g).
2. Else if `pantry_item.nutrition_ref` is set, look up in `nutrition.json`.
3. Else 0 (and flag in response so the UI can warn the user).

Sum per recipe, then divide:
- **Per recipe total** = sum of (grams / 100) × per_100g
- **Per serving** = total / `recipe.servings`
- **Per 100 g** = total / total_recipe_g × 100
- **Daily-value %** computed against a fixed reference (2000 kcal, 75 g fat, 275 g carbs, 50 g protein). Configurable per user later.

### 6.4 Cost (`cost.py`)

For each ingredient:
1. If `cost_override_per_kg` is set, use it.
2. Else if `pantry_item.cost_per_kg`, use it.
3. Else 0 (and flag missing).

Sum and divide for per-serving cost.

### 6.5 Ready-by (`schedule.py`)

Inputs: recipe's steps with `min_seconds`, `timer_seconds` (or `expected`), `max_seconds`, and `parallelizable_with`. Target end time.

Algorithm:
1. Build a directed graph where each step depends on the previous one unless `parallelizable_with` is set.
2. Compute critical path using `expected` durations → `expectedTotalSeconds`.
3. Compute min and max paths similarly → `minTotalSeconds`, `maxTotalSeconds`.
4. Return `{startAt: target - expected, rangeStartAt: target - max, rangeEndAt: target - min}`.

UI shows the expected start time prominently with a small "± X min" range.

### 6.6 Pattern detection (`patterns.py`)

Hand-coded rules over recent bakes. Examples:

- If `tweaks.change` matches `r"\+\s*\d+\s*g?\s*water"` ≥ 3 times in last 10 bakes for any recipe → emit `"You've been adding water to {recipe}; consider raising default hydration"`.
- If `outcome IN ('meh','disaster')` and `internal_temp_c < 92` → emit `"Underbaked outcomes recently — internal temp under 92°C"`.
- If `tweaks.change` matches `r"reduce.*bake|less.*time"` ≥ 3 times → emit `"Reducing bake time keeps coming up — lower the default"`.

Add 2-3 more rules per phase as patterns reveal themselves. **Do not LLM-generate these.** They should feel earned.

---

## 7. Frontend structure

App Router routes:

```
/                        Home: greeting, active bake card, pinned, recents
/recipes                 List + filters + search
/recipes/new             Editor (also /recipes/[id]/edit)
/recipes/[id]            View — denser, with macros/cost/ratios/ready-by panels
/recipes/[id]/bake       Active bake mode (full screen)
/bakes                   Journal (timeline)
/bakes/[id]              Single bake detail / reflection screen
/insights                Dashboard
/pantry                  Pantry items list (cost + nutrition refs)
/starter                 Starter dashboard
/settings                R2 status, export, manifest reinstall, daily-value targets
```

Key components are specified in `DESIGN.md` with exact tokens. The most important shared components: `<HandRule>`, `<SectionLabel>`, `<NutritionPanel>`, `<CostPanel>`, `<RatiosPanel>`, `<ReadyByPanel>`, `<TweakBanner>`, `<PendingTweakBadge>`, `<ActiveBakeTimer>`, `<MeasurementGrid>`, `<CrumbCrustSlider>`, `<StatsStrip>`, `<BakesPerMonthChart>`, `<TopTweaksList>`, `<EquipmentLeaderboard>`, `<CalendarHeatmap>`.

---

## 8. Active bake mode (the killer feature)

Full-screen route. No site chrome. Dark by default — see espresso palette in `DESIGN.md`.

### 8.1 Pre-bake context capture

On entry from `/recipes/[id]`, before going full-screen, show a small modal:

> *Quick context (skip if you don't care)*
> - Kitchen temp: [auto-fill from last bake | input]
> - Humidity: [optional slider]
> - Flour brand: [auto-fill if same recipe]

Stored on `bakes` row. Skippable. Defaults to last bake's values.

### 8.2 Layout

(see §8.2 of design v1 — this is unchanged from the original spec)

### 8.3 Behaviors

- **Wake Lock** acquired on entry. Released on exit or pause. If browser denies (Safari pre-16.4), show a small toast.
- **Timer state** in IndexedDB under `bake:{bakeId}:timers` — `{stepOrd, targetEpochMs, startedAt, pausedAt, label}`.
- **Tick loop** in a Web Worker (`/workers/timer.ts`) posting `tick` every 250 ms. UI subscribes via `useTimer(stepOrd)`.
- **Backgrounding** — worker keeps running until OS suspends it. On `visibilitychange → visible`, recompute remaining from `targetEpochMs - Date.now()`. Never trust accumulated tick counts.
- **Notification on done** — `Notification` with `requireInteraction: true`, vibration `[200,100,200,100,400]`, preloaded sound from `/public/sfx/timer-end.mp3`. Permission asked once on first timer start with a clear explainer.
- **Resume** — if user opens `/recipes/[id]/bake` and there's an active bake for that recipe, redirect to it and restore `current_step`.
- **Photos mid-bake** — capture → preview → upload in background. Tag with `step_ord` automatically.
- **Tweaks mid-bake** — modal: `change` (free text) + `apply_next_time` checkbox + optional ingredient/step pick.
- **Exit** — swipe-down confirms "Pause bake?" with *Pause and exit* / *Mark abandoned* / *Cancel*. Bake is only closed via the explicit *Finish bake* flow.

---

## 9. Reflection screen (`/bakes/[id]`)

Triggered when user finishes the last step or taps *Finish bake*. The screen turns each bake into structured data.

### 9.1 Sections, in order

1. **Stars** — 1–5 tap input.
2. **Outcome chip** — single select: `disaster | meh | okay | good | best_yet`.
3. **Photos** — 4-tile grid. Each tile has a `kind` chip (`final`, `crumb`, `side`, `top`, `crust`). One add tile.
4. **Measurements grid** — 2-column tiles, all optional:
   - Rise height (cm)
   - Internal temp (°C) — from a meat thermometer
   - Weight before bake (g)
   - Weight after bake (g)
   - **Computed**: water loss % (highlighted, derived) — `(before - after) / before * 100`
   - **Computed**: oven spring %, if rise was recorded both pre- and post-bake
5. **Sliders** — crumb structure (tight ↔ open), crust (pale ↔ blistered). Stored as 1–5.
6. **Tweaks logged** — auto-populated from in-bake tweaks. Editable.
7. **Free-text notes** — markdown.
8. **Save & finish** button — PATCH `bakes` with all fields, set `status='finished'`, release wake lock, navigate to `/bakes/[id]` view.

Form is mostly optional; only rating is required.

---

## 10. Insights screen (`/insights`)

Earned, not exuberant. Useful patterns from real data, not vanity charts.

### 10.1 Sections

1. **Range selector** — Month / Year / All time (chips).
2. **Stats strip** — 2×2 grid: bakes count, avg rating (with delta), flour kg, total cost (CHF, with rough store-bought comparison if recipe has yields).
3. **Bakes per month** — bar chart, 12 months, current month highlighted in `--amber-bright`.
4. **Top tweaks** — ranked list with counts. Below: a hand-written pattern callout from the patterns service if any rules fired.
5. **Equipment** — table: name, avg rating (stars), bake count.
6. **Calendar heatmap** — full-year grid, GitHub-style, intensity from bake count.

Data from `/insights/*` endpoints. All charts are SVG, no chart library — see `DESIGN.md` for the pattern.

### 10.2 Don't ship the scatter plots yet

Hydration vs rating, kitchen temp vs proof time — all valid, all useless before ~30 logged bakes. The conviction-filter discipline applies: don't show signals that aren't signals. Add these in a v1.5 phase once the journal has volume.

---

## 11. Seed data

Ship with these recipes, hand-authored, in `apps/api/seed/recipes/`:

1. **Buttermilk cornbread** (cast iron skillet).
2. **Swiss Butterzopf** (yeast, enriched dough, braiding).
3. **No-knead Dutch oven loaf**.
4. **Focaccia**.
5. **Cinnamon rolls**.
6. **Toastbread / sandwich loaf**.
7. **Bagels**.
8. **Banana bread**.

Each recipe must have:
- Hero photo placeholder
- 6–14 steps, with `timer_seconds` where appropriate, `min_seconds`/`max_seconds` populated for ready-by calc
- Ingredients tagged with `role`, linked to seeded `pantry_items`
- `servings` and `serving_g` populated for macro/cost math
- `equipment` array set
- 2–3 of them with a sample `bake` row already populated, so `/insights` has something to render on first launch

Seeded `pantry_items` (~30): all flours, all leavens, butter, milk, buttermilk, eggs, sugars, salt, common nuts, common seeds, olive oil, neutral oil, dark chocolate, cocoa, cinnamon, vanilla, common cheeses. Each with a Swiss-realistic `cost_per_kg` (best guesses; user adjusts).

`apps/api/data/nutrition.json` is built by `scripts/build-nutrition-table.py` from USDA FoodData Central. The script is run once during M1, output committed.

---

## 12. Implementation phases

Hard exit gate per phase: Playwright passes, manual checklist passes, deployed to staging on Coolify. Never start the next phase until the previous gate is green.

### M0 — Project skeleton (1 session)

- Monorepo: `apps/web`, `apps/api`, `packages/types` (TS types from Pydantic via `datamodel-code-generator`).
- pnpm workspaces. Compose file for local dev (Postgres + api + web).
- Lint/format: biome (web), ruff + black (api), pre-commit hooks.
- CI: GitHub Actions runs typecheck, ruff, vitest, pytest, playwright headless on PR.
- Coolify: two empty services + Postgres provisioned. Tunnel route configured to staging URL.
- Apply `DESIGN.md` tokens — Tailwind config has the cream + espresso palettes, Fraunces + JetBrains Mono loaded, the `@layer base` styles in place.
- **Gate:** `curl https://bakebook-staging.../api/v1/health` returns 200 from your phone. The web root shows a placeholder page styled with the design tokens (so you can verify Fraunces is loading correctly).

### M1 — Recipes + pantry + nutrition + ratios + cost (2 sessions)

- Schema migrations: `recipes`, `recipe_ingredients`, `recipe_steps`, `pantry_items`, `pantry_price_history`.
- API: §5.1 (recipes) and §5.2 (pantry).
- Pure services: `scaling.py`, `ratios.py`, `nutrition.py`, `cost.py` with full unit tests.
- Build script: `scripts/build-nutrition-table.py` → commit `data/nutrition.json`.
- Seed script: pantry + 8 recipes.
- Frontend: `/recipes`, `/recipes/[id]`, `/recipes/new`, `/recipes/[id]/edit`, `/pantry`.
- Recipe view shows nutrition panel, cost panel, ratios panel exactly per `DESIGN.md`.
- Per-serving / per-100g toggle on nutrition panel (state in URL hash so it persists).
- **Gate:** Playwright opens cornbread recipe, captures screenshot, asserts macros panel shows 4 macro cards with non-zero values, cost panel shows non-zero CHF, ratios panel renders for the bread recipes and is *hidden* for cornbread (which has chemical leavening, not flour-percentage logic).

### M2 — Active bake + photos + reflection (2 sessions)

- Schema: `bakes`, `bake_photos`, `bake_tweaks` + the env-context columns.
- R2 wired up. Presigned PUT working.
- Active bake route with timer worker, wake lock, notification on done.
- Pre-bake context modal (kitchen temp, humidity, flour brand).
- Photo capture from active bake.
- Reflection screen (§9): rating, outcome, photo grid, measurement grid with computed water loss, sliders, tweak list, notes.
- Journal timeline at `/bakes`.
- **Gate:** Run a real bake (cornbread) end-to-end on the phone with the screen locking once mid-bake. Timer fires correctly. Photo uploads. Reflection screen captured with 4 measurements + computed water loss visible.

### M3 — Versioning + tweaks + scaling + ready-by (1 session)

- "Pending tweaks" banner on recipe view.
- One-tap "apply to next version" creates new `recipes` row, copies ingredients/steps, applies tweaks via parsed `change` strings (e.g. `+10 g water` → resolve to ingredient by role, add 10 to grams), marks source tweaks resolved.
- `POST /recipes/{id}/scale` + scale UI (slider for multiplier, target dough weight input).
- `GET /recipes/{id}/ready-by?target=...` + ready-by panel on recipe view with target time picker.
- Fork action.
- Allergen badges — auto-derived from ingredients (gluten if any flour ∈ {wheat, spelt, rye}, dairy if any role='dairy', egg if any role='egg', + manual override).
- **Gate:** Bake the focaccia, log "+15 g water" as a tweak with `apply_next_time=true`, apply it to v2, verify v2 ingredient list reflects the change and history is intact. Set ready-by to 18:00 and verify start time is computed within ±10 min of expected.

### M4 — Starter + PWA polish (1 session)

- Starters + feedings.
- Starter status endpoint: `peak_hours = peak_base_hours * 2^((20 - kitchen_temp_c) / 10)`.
- PWA manifest, icons, install prompt.
- Offline cache verified — toggle airplane mode, last 5 viewed recipes still readable.
- Push notifications on iOS 16.4+ home-screen install.
- **Gate:** Install on the phone, force-quit Safari, open the icon, view cornbread with no network. Start a bake, lock the phone, get the timer notification.

### M5 — Insights (1 session, after at least 10 real bakes)

- Insights endpoints (§5.4).
- Patterns service (§6.6) with 5 hand-coded rules.
- `/insights` screen exactly per `DESIGN.md`: range selector, stats strip, bar chart, top tweaks with pattern callout, equipment table, calendar heatmap.
- All charts SVG, no chart library.
- **Gate:** Open `/insights` on the phone, verify all 5 panels render with real data from your actual journal. At least one pattern callout fires (or the panel is gracefully empty with placeholder copy).

Stop here. v1.5 (scatter plots, hydration vs rating, etc.) only after ~30 logged bakes.

---

## 13. Verification protocol

This project is built with Claude Code in autonomous sessions. Hard rules:

- **Never declare a feature done without a passing Playwright test that takes a screenshot.** Screenshots → `/test-results/screenshots/{phase}/{feature}.png`, committed.
- **Compare screenshots to design references.** For each new screen, the Playwright test takes a screenshot at 380×800 viewport. The design HTML files in `design/` show the intended look at the same viewport. Diff visually before declaring done.
- **Always run a real flow end-to-end at least once on the deployed staging URL from a real phone before closing a phase.** Note kitchen temperature and any timer drift in `phases/M{n}/done.md`.
- **Cross-check before declaring M2 and M3 done.** Run Codex CLI and Gemini CLI against the active-bake worker and the tweak-application logic with: *"Find race conditions, timer drift sources, ways the timer can be lost on backgrounding, edge cases in the tweak parser. Be adversarial."* Address every concrete issue raised.
- **Honest failure log.** Each phase has `phases/M{n}/issues.md` listing every bug, with the fixing commit. Empty file is suspicious — write down the small ones too.
- **Do not skip migrations.** Every schema change is a migration. No `Base.metadata.create_all` in app code.
- **Write the patterns rules by hand.** Do not have an LLM generate `patterns.py` rules at runtime. They live in code and are reviewed.

---

## 14. Repository layout

```
bakebook/
├── apps/
│   ├── web/                     # Next.js 14
│   │   ├── app/
│   │   ├── components/
│   │   │   ├── recipe/
│   │   │   ├── bake/
│   │   │   ├── insights/
│   │   │   └── shared/         # HandRule, SectionLabel, ...
│   │   ├── lib/
│   │   ├── workers/timer.ts
│   │   └── public/
│   └── api/
│       ├── app/
│       │   ├── routes/
│       │   ├── models/
│       │   ├── schemas/
│       │   ├── services/
│       │   │   ├── scaling.py
│       │   │   ├── ratios.py
│       │   │   ├── nutrition.py
│       │   │   ├── cost.py
│       │   │   ├── schedule.py
│       │   │   ├── patterns.py
│       │   │   ├── starter.py
│       │   │   └── r2.py
│       │   └── main.py
│       ├── alembic/
│       ├── data/
│       │   └── nutrition.json
│       ├── scripts/
│       │   └── build-nutrition-table.py
│       └── seed/
├── packages/types/              # shared TS types
├── playwright/
│   ├── recipes.spec.ts
│   ├── active-bake.spec.ts
│   ├── reflection.spec.ts
│   ├── insights.spec.ts
│   └── fixtures/
├── docker-compose.yml           # local dev
├── .github/workflows/ci.yml
├── decisions/                   # ADRs
├── phases/
│   ├── M0/{task.md, done.md, issues.md}
│   └── ...
├── design/                      # visual reference
│   ├── v1-screens.html
│   └── v2-data.html
├── CLAUDE.md
├── DESIGN.md
└── BOOTSTRAP.md
```

---

## 15. Local dev

```bash
# one-time
cp .env.example .env
pnpm install
docker compose up -d postgres
pnpm --filter api migrate
pnpm --filter api seed

# every session
pnpm dev                         # web (3000) + api (8000) concurrent
```

`web` dev proxies `/api/*` to `http://localhost:8000`. In prod they're separate origins behind the same hostname via Coolify routes.

---

## 16. Deployment

- Coolify project: `bakebook`. Two services: `web`, `api`. One Postgres.
- `web` Dockerfile: standalone Next.js output, multi-stage, final image runs `node server.js`.
- `api` Dockerfile: python 3.12-slim, `uv sync --frozen`, `uvicorn app.main:app --host 0.0.0.0 --port 8000`.
- Auto-deploy on push to `main`.
- Cloudflare Tunnel: route `bakebook.nicolasschaerer.ch` → Coolify proxy. CF Access policy: single email allowed.
- Migrations on api startup via `entrypoint.sh` running `alembic upgrade head`. Add `--no-migrate` flag for emergencies.

---

## 17. Things to deliberately not do

- No recipe URL scraping. Manual entry forces understanding; structure is the value.
- No cup/tablespoon math. `grams` is canonical. `unit_display` is display-only.
- No real-time collaboration, no public sharing.
- No AI features in v1. (You can add Claude API "describe what went wrong, suggest tweaks" in v2 if it earns its keep.)
- No native app wrapper. PWA only.
- No Redis, no queue, no microservices. One web, one api, one db.
- No machine-generated insights. All `patterns.py` rules are hand-coded.
- No premature analytics. Hold scatter plots until you have real data.

---

## 18. Open questions to resolve before M0

1. **Domain**: `bakebook.nicolasschaerer.ch` ok, or another subdomain?
2. **R2 bucket**: new dedicated bucket, or share an existing one with a prefix?
3. **Notification sound**: pick / record one, or generic chime?
4. **Kitchen temp source**: manual each bake, or pull from Home Assistant?
5. **Daily-value reference**: 2000 kcal default, or a personal target?

Default to the first option in each unless told otherwise.

---

*End of CLAUDE.md. Ship M0 first. Don't read ahead while building.*

---
> Source: [NlcoIas/bakebook](https://github.com/NlcoIas/bakebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
