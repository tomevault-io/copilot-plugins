## nutriplan

> **NutriPlan** is an AI-powered calorie and meal management app.

# NutriPlan — CLAUDE.md

## Project Overview

**NutriPlan** is an AI-powered calorie and meal management app.
Users enter their body metrics and a goal (fat loss, muscle gain, or recomposition),
and the Anthropic Claude API automatically generates a personalized 7-day meal plan and grocery list.

- Unidirectional flow: Onboarding → Meal Plan Config → Daily View → Grocery List
- Two onboarding paths: **Quick Estimate** (default — body type → estimated body composition) and **InBody / Precision** (exact measurements)
- Supports Korean and English, and multiple cuisine styles (Korean, Japanese, Western, etc.)
- Designed and deployed on Vercel

Refer to UX_FLOW.md in the project root for the complete screen inventory, user flows, navigation map, and known pain points before making any UI changes.

---

## Tech Stack

| Category | Technology |
|----------|------------|
| Framework | Next.js 16.2 (App Router) |
| UI Library | React 19 + TypeScript 5.7 |
| Styling | Tailwind CSS v4 + shadcn/ui (Radix UI based) |
| Icons | Lucide React |
| Global State | Zustand v5 (with persist middleware) |
| AI | Anthropic SDK (`claude-haiku-4-5-20251001`) |
| Charts | Recharts |
| Forms | React Hook Form + Zod |
| Notifications | Sonner (toast) |
| Deployment | Vercel (with `@vercel/analytics`) |

---

## File Structure

```
app/
├── page.tsx                        # Root entry point — controls screen switching via currentStep
├── layout.tsx                      # Global layout (fonts, metadata)
├── globals.css                     # Global CSS (Tailwind layers)
└── api/
    ├── generate-meal-plan/
    │   └── route.ts                # Generates 7-day meal plan via Claude
    ├── swap-meal/
    │   └── route.ts                # Generates 3 swap candidates for a single meal via Claude
    └── claude-nutrition/
        └── route.ts                # Looks up nutrition data for a custom ingredient via Claude

components/
├── onboarding.tsx                  # Multi-step onboarding wizard (Step 0) — two entry paths
├── meal-plan-config.tsx            # Meal plan configuration screen (Step 1)
├── daily-view.tsx                  # Daily meal view with swap, edit-profile modal (Step 2)
├── grocery-list.tsx                # Full-week grocery list screen (Step 3)
├── grocery-item-row.tsx            # Individual grocery item row component
├── meal-swap-sheet.tsx             # Bottom sheet overlay for meal swapping (used inside DailyView)
├── settings-screen.tsx             # App preferences + profile quick-edit (Step 4)
├── theme-provider.tsx              # Dark mode theme provider
└── ui/                             # shadcn/ui components (can be modified directly)

lib/
├── meal-store.ts                   # Zustand global state + core type definitions
├── nutrition.ts                    # Calorie and macro target calculation (Katch-McArdle)
├── meal-validator.ts               # Validates generated meal plans and swap candidates
├── grocery.ts                      # Grocery list utilities
├── recipe-units.ts                 # Unit conversion (metric / imperial)
└── utils.ts                        # Shared utilities (clsx + tailwind-merge)

hooks/
├── use-mobile.ts                   # Mobile detection hook
└── use-toast.ts                    # Toast notification hook

.github/workflows/
├── ci.yml                          # Build verification on push/PR to main
└── ai-review.yml                   # AI-powered PR review via Claude Code Action

.claude/commands/
├── prd.md                          # /project:prd slash command
├── implement.md                    # /project:implement slash command
└── pipeline.md                     # /pipeline full automation command (PRD → prompts.json → execute)

prompts.json                        # Current feature prompts consumed by run-prompts.sh
run-prompts.sh                      # Automation script: runs prompts → build check → commit
```

### App Flow (by currentStep)

```
0: Onboarding → 1: MealPlanConfig → 2: DailyView → 3: GroceryList
                                                  ↕ (FAB button)
                                              4: SettingsScreen
```

---

## Onboarding Flow

The onboarding wizard (`components/onboarding.tsx`) collects all data needed to compute calorie and macro targets. It has two entry paths that converge at the same steps from "activity level" onward.

### Path 1 — Quick Estimate (default)

The default path. Estimates body fat % and muscle mass from visible body type so users without lab results can still get accurate targets.

1. User enters: **sex**, **weight**, **height**, **age**, **body type** (lean / average / athletic / heavy-set).
2. Taps "Use quick estimate" → `applyQuickEstimate()` derives `bodyFat %` and `muscleMass` from a lookup table (`quickEstimateMap[sex][bodyType]`).
3. Continues to: activity level → goal → target weight (conditional) → cuisine → diet type → ingredient mode → ingredients → "Build my plan".

### Path 2 — InBody / Precision

For users with exact body composition measurements (e.g., InBody scanner).

1. User taps "Enter exact body stats" → body stats form: **weight**, **body fat %**, **muscle mass**, **sex**, **unit system**.
2. Steps from activity level onward are identical to Quick Estimate.

### Calorie & Macro Calculation (`lib/nutrition.ts`)

Both paths ultimately call `calculateNutritionTargets()` which uses the **Katch-McArdle formula** (LBM-based BMR):

- `BMR = 370 + 21.6 × LBM(kg)` (female multiplier: ×0.9)
- `TDEE = BMR × activity multiplier`
- Calorie adjustment by goal: fat loss (deficit by pace), muscle gain (+300 kcal), recomposition (−200 kcal)
- Calorie floor: **1500 kcal (male)**, **1200 kcal (female)**
- Protein: `LBM × g/kg multiplier` (varies by goal + diet type), clamped 1.2–2.2 g/kg LBM
- Remaining calories split into carbs/fat by diet type ratios

When `bodyFat` is not provided (Quick Estimate path), body fat % is estimated from `bodyType + sex` using midpoint lookup tables in `nutrition.ts`.

### Draft Persistence

Mid-onboarding state is saved to `localStorage["nutriplan-onboarding-draft-v3"]` on every change and restored on mount. It is cleared when `handleComplete()` runs.

### Ingredient Selection

Users can pick ingredients from a built-in catalog (by category tab or search) or choose a budget preset (Low / Medium / High). If a searched ingredient is not in the catalog, `fetchIngredientViaClaude()` calls `POST /api/claude-nutrition` to look up nutrition data on the fly. Minimum valid selection: 2 proteins + 1 carb + 1 fat.

---

## API Notes

All three API routes live in `app/api/` and are server-side only. All use `process.env.ANTHROPIC_API_KEY`.

### `POST /api/generate-meal-plan`

- **Model:** `claude-haiku-4-5-20251001` | **max_tokens:** 4000 | **Timeout:** 35 s
- Asks Claude to generate **3 breakfast + 3 lunch + 3 dinner options**, then builds a 7-day rotation via `MEAL_ROTATION` (a fixed index pattern) so the week has variety without 7 separate AI calls.
- Returns `{ days: DayPlan[] }`. Each day has `meals[]` with name, calories, protein, carbs, fat, ingredients, recipe (prepTime, cookTime, instructions).
- Applies `adjustMacros()` post-processing: if fat or carbs are <80% of target, it adds olive oil or white rice as correction ingredients.
- **Language rule:** defaults to English; switches to Korean only when `language === "ko"`. Korean cuisine style adds a strict rule disallowing Western meal names.

### `POST /api/swap-meal`

- **Model:** `claude-haiku-4-5-20251001` | **max_tokens:** 2000 | **Timeout:** 35 s
- Generates 3 alternative meals for a given slot (breakfast / lunch / dinner), each within ±15% of the current meal's macros.
- Candidates are filtered through `validateSwapCandidate()` in `lib/meal-validator.ts` before being returned.
- Returns `{ candidates: SwapCandidate[] }` (up to 3).

### `POST /api/claude-nutrition`

- **Model:** `claude-haiku-4-5-20251001` | **max_tokens:** 300 | **Timeout:** 20 s
- Looks up per-100g nutrition data for a single ingredient string.
- Returns `{ name, category, calories, protein, carbs, fat, cost }`.

### Shared Constraints

- All routes use `export const maxDuration = 60` (Vercel function timeout).
- Claude is called with `system: "Output strict JSON only."` — no markdown, no explanation.
- A `Promise.race` wraps every Claude call with a hard timeout that rejects before Vercel's limit.
- Response parsing defensively handles alternate Claude output shapes (e.g. `foods` vs `ingredients`, `macros.protein` vs `protein`, camelCase vs snake_case field names).

---

## Known Patterns

### Zustand Store (`lib/meal-store.ts`)

Store key: `meal-plan-storage`. Created with `zustand/middleware/persist`.

| Slice | Persisted | Description |
|-------|-----------|-------------|
| `userProfile` | Yes | All body metrics, goals, preferences, calorie/macro targets |
| `weekPlan` | Yes | 7-day generated `DayPlan[]` |
| `appPrefs` | Yes | `language`, `unitSystem`, `darkMode` |
| `currentStep` | No | Active screen index (0–4) |
| `mealPlanConfig` | No | `dietType` + `mealsPerDay` (rebuilt from `userProfile` on rehydration) |
| `isGeneratingMealPlan` | No | Loading flag for AI generation |
| `mealPlanValidation` | No | Errors/warnings from `validateMealPlan()` |
| `selectedDay` | No | Active day index in DailyView (0–6) |
| `swapCandidates` | No | Temporary candidates from swap API |

**Rehydration:** On load, if `userProfile` exists → rebuild `mealPlanConfig` + navigate to step 2 (or step 1 if no `weekPlan`). If no profile → step 0.

**Profile version:** `PROFILE_VERSION = 2`. `migrate()` normalizes stored profiles on version mismatch. `normalizeUserProfile()` sanitizes every field with typed guard functions — always run profile data through it before storing.

### localStorage Keys

| Key | Owner | Contents |
|-----|-------|----------|
| `meal-plan-storage` | Zustand persist | `appPrefs`, `userProfile`, `weekPlan` |
| `nutriplan-onboarding-draft-v3` | `onboarding.tsx` | All mid-onboarding form state — cleared on `handleComplete()` |

### Meal Swap Logic

1. User taps Swap on a meal card in `DailyView` → `MealSwapSheet` (bottom sheet) opens.
2. On mount, `MealSwapSheet` calls `POST /api/swap-meal` directly (not via Zustand).
3. API returns up to 3 validated candidates (±15% macro tolerance).
4. User selects a candidate → `swapMeal(dayIndex, mealIndex, newMeal)` in Zustand updates `weekPlan` and recalculates all day totals in-place.

### Meal Plan Generation Flow

1. `generateMealPlan()` in `lib/meal-store.ts` POSTs the full profile payload to `/api/generate-meal-plan`.
2. Response is mapped from Claude's JSON format to internal `DayPlan[]` / `Meal[]` types via defensive field-name normalization.
3. `validateMealPlan()` runs; if invalid → `weekPlan = []` + error state.
4. On success → `weekPlan` is set; `DailyView` exits its loading state.

---

## Coding Guidelines

### Language & Comments
- All comments and documentation must be written in **English**.
- Only add comments where the logic is not self-evident.

### UI Strings & API Responses
- All user-facing strings default to **English**.
- The app supports Korean (`language === "ko"`), but English is the default.
- API route prompts explicitly enforce English output unless `language === "ko"` is set — never assume language from cuisine preference alone.

### Components
- Use **function components only**. No class components.
- Client components must have `"use client"` at the top.
- Place new components in `components/` or `components/ui/`.

### State Management
- All global state through **Zustand store in `lib/meal-store.ts`** only.
- Local UI state (modal open/closed, etc.) may use `useState`.

### UI Components
- Prefer shadcn components from `components/ui/`.
- Use `cn()` from `lib/utils.ts` for composing Tailwind classes.

### Package Installation
- **Always notify the user and get approval before installing any new package.**

### Types
- Avoid `any`. Core types live in `lib/meal-store.ts`.
- The only sanctioned exceptions are AI response parsers in `app/api/` routes, where the Claude response shape is inherently untyped — document each use with a comment explaining why.

---

## Security Rules

- **Never commit `.env` or `.env.local`.**
- **Never hardcode API keys.** Use `process.env.ANTHROPIC_API_KEY`.
- Never reference server-only env vars inside client components.
- `node_modules/`, `.next/`, `tsconfig.tsbuildinfo` must not be committed.
- API routes (`app/api/`) are server-side only.

---

## Automation Pipeline

This project uses a Claude Code automation pipeline for all feature development.

### How It Works

```
/pipeline [--major] "feature description"
        ↓
Product Agent generates PRD → saved to Notion (NutriPlan_PRD_v{X.Y})
        ↓
Engineering Agent generates prompts.json (feature/branch/prompts wrapper)
        ↓
User checkpoint: yes / edit / no
        ↓
./run-prompts.sh
        ↓
Claude Code implements → build check → commit → push
```

### `run-prompts.sh` Behavior

- Reads `prompts.json` (fields: `feature`, `branch`, `prompts[]`).
- Creates or checks out the `branch` specified in `prompts.json` before running anything.
- For each prompt: runs `claude --print --dangerously-skip-permissions -p "<prompt>"`, then verifies `npm run build`.
- On build failure: retries up to **3 times**, sending the build error back to Claude as a fix prompt.
- Hard cost cap: **$5 USD** (estimated by token count). Stops early if exceeded.
- On success: commits with message `prompt #<id>: <description>` and pushes the branch.
- **Always `git commit` any in-progress work before running this script** — it will create new commits on the current or specified branch.

### Slash Commands

| Command | What it does |
|---------|-------------|
| `/pipeline` | **Full end-to-end: PRD → Notion → prompts.json → run-prompts.sh** (primary command) |
| `/project:prd` | Generate a PRD only (no prompts.json, no execution) |
| `/project:implement` | Generate prompts.json from an existing PRD (no execution) |

Use `/pipeline` for the normal feature loop. Use the other two only when you need to split the flow (e.g. draft a PRD today, implement tomorrow).

### Git Rules for Automation
- Always `git commit` before running any automation script.
- Always work on a **feature branch** — never run `run-prompts.sh` directly on `main`.
- Commit message format: `feat: [feature name]`, `fix: [bug description]`, `chore: [task]`
- Never force push to `main`. Use feature branches for risky changes.

### GitHub Actions

#### CI (`ci.yml`)
- Triggers on push and pull_request to `main` and `design/codex-ui-redesign`.
- Runs `npm ci` + `npm run build` with `ANTHROPIC_API_KEY` injected from GitHub Secrets.
- Purpose: verify the build never breaks on those branches.

#### AI PR Review (`ai-review.yml`)
- Triggers on pull_request (opened / synchronize / reopened) targeting `design/codex-ui-redesign`.
- Generates a git diff (truncated to ~12,000 chars), reads `CLAUDE.md` for coding standards, then runs `claude-code-action@beta` to post a structured review comment on the PR.
- Review covers: CLAUDE.md compliance, security, TypeScript type safety, code quality.
- Uses `ANTHROPIC_API_KEY` + `GITHUB_TOKEN` from Secrets.
- **Note:** AI review currently only runs on PRs to `design/codex-ui-redesign`, not `main`.

---

## Pipeline Command

Full automation pipeline is available via Claude Code custom command:

```
/pipeline [--major] "<feature or bug description>"
```

**Flow:**
1. **Product Agent** — generates PRD, saves to Notion (Private → NutriPlan page) as `NutriPlan_PRD_v{X.Y}`, also saves local copy at `docs/prd/`.
2. **Engineering Agent** — reads PRD, generates `prompts.json` (wrapper structure: `{ feature, branch, prompts: [...] }`), overwrites root file.
3. **Execute** — runs `./run-prompts.sh` after user confirms.

**Versioning:**
- Minor (+0.1) auto-incremented by scanning Notion for latest `NutriPlan_PRD_v*`.
- Major (+1.0) requires explicit `--major` flag.
- Always creates a new Notion page — never overwrites existing PRDs.

**Checkpoint:** After Step 2, user must confirm `yes` / `edit` / `no` before execution.

**Source:** `.claude/commands/pipeline.md`

---

## Agent System Prompts

The Product Agent and Engineering Agent prompts are defined inline inside `.claude/commands/pipeline.md`. That file is the single source of truth — do not duplicate the prompts here. If you need to adjust agent behavior, edit `pipeline.md` directly.

Legacy standalone commands (`/project:prd`, `/project:implement`) share the same agent prompts via their own command files in `.claude/commands/`.

---
> Source: [jeelimm/nutriplan](https://github.com/jeelimm/nutriplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
