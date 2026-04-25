## bubble-retirement

> NZ-market Monte Carlo retirement calculator with fan chart, auth, and annual review workflow.

# Bubble Retirement — CLAUDE.md

NZ-market Monte Carlo retirement calculator with fan chart, auth, and annual review workflow.

## Skills

Project skills live in `.claude/skills/`. Invoke them with the Skill tool or `/skill-name`.

- **`frontend-design`** (`UI skill`) — Use when building or restyling any UI component, page, or layout. Enforces the project's brutalist design system and avoids generic AI aesthetics. Invoke whenever the user asks to build, redesign, or visually improve any frontend element.

## Dev

```bash
npm run dev      # http://localhost:3000
npm run build    # production build
npm run lint
NODE_TLS_REJECT_UNAUTHORIZED=0 npx prisma generate   # needed on this machine (SSL cert issue)
npx prisma migrate dev   # local DB migrations
```

## Architecture

Next.js 15 App Router. Monte Carlo runs client-side; auth + state persistence via API routes + PostgreSQL.

```
app/
  page.tsx               — main page, all state, useMemo over simulation, 4 result tabs
  login/page.tsx         — NextAuth v5 login + register
  api/
    register/            — POST: create user (bcrypt)
    state/               — GET/PUT: upsert SimulationState (JSONB auto-save)
    snapshots/           — GET/POST: annual review snapshots
    snapshots/[id]/      — PATCH/DELETE: individual snapshot
components/
  ui/                    — button, card, input, label, slider, tabs, badge
  InputForm.tsx          — Assets section (add/delete, per-asset return+sigma sliders) + Income section
  FanChart.tsx           — Recharts ComposedChart, "erase" technique for percentile bands
  ResultsSummary.tsx     — stat grid, colour-coded success, auto-suggestions
  ScenarioComparison.tsx — pessimistic/base/optimistic deltas applied to all assets uniformly
  YearlyTable.tsx        — scrollable table, per-asset median columns, retirement age highlighted
  AnnualReview.tsx       — save yearly snapshots, enter actuals per asset, projected vs actual variance
lib/
  types.ts               — AssetDefinition, IncomeStream, SimulationInputs, YearBand
  montecarlo.ts          — runSimulation(), per-asset independent log-normal; findIncomeSuggestion()
  utils.ts               — cn(), formatCurrency() (NZD), formatPercent()
prisma/
  schema.prisma          — User, SimulationState, ProjectionSnapshot (all JSONB payloads)
```

## Monte Carlo Algorithm

All values in real (inflation-adjusted) terms. Log-normal return sampling via Box-Muller. Each asset simulates independently.

**Accumulation** (currentAge → retirementAge):
```
growthFactor = exp(mu_ln + sigma × z)   // z = Box-Muller standard normal
assetValue = assetValue × growthFactor + asset.annualContribution
```

**Decumulation** (retirementAge → lifeExpectancy):
```
assetValue = assetValue × growthFactor
totalPortfolio -= withdrawal            // from income streams config
income streams (incl. NZ Super) added by age range
portfolio = max(portfolio, 0)           // floor at ruin
```

Run 1000 sims, extract p5/p10/p25/p50/p75/p90/p95 per year. successProbability = % of sims where final portfolio > 0.

## Design

Fin's design system: muddy browns + neon orange/yellow, Space Grotesk/Mono, brutalist hard shadows. All tokens in `app/globals.css`.

- Background: `#17130E`, Surface: `#231C14`, Border: `#4A3828`
- Accent orange: `#FF4800` (primary), Accent yellow: `#F5DF00` (secondary)
- Text: `#EAE0D0`, Muted: `#A89880`
- Shadow: `4px 4px 0 #4A3828`, Accent shadow: `5px 5px 0 #FF4800`

## Fan Chart (Recharts "erase" technique)

Recharts has no native band support. Technique:
1. `<Area dataKey="p95">` — low-opacity fill
2. `<Area dataKey="p5">` — filled with background colour (erases below p5)
3. Repeat for p75/p25 (medium opacity)
4. `<Line dataKey="p50">` — orange, strokeWidth 2
5. `<ReferenceLine x={retirementAge}>` — dashed yellow

## Deployment (Railway)

1. Provision a PostgreSQL plugin — Railway auto-sets `DATABASE_URL`
2. Set `NEXTAUTH_SECRET` manually in the Railway env vars dashboard
3. Set `NEXTAUTH_URL` to the production domain
4. Run `npx prisma migrate deploy` after first deploy to apply schema
5. Railway auto-detects Next.js: runs `npm run build && npm start`

## Known Gotchas

- Prisma binary download requires `NODE_TLS_REJECT_UNAUTHORIZED=0` on this machine due to SSL cert issue
- Prisma locked to v6 — v7 introduced breaking URL config changes that caused incompatibility
- NZ Super is an income stream of type `nz_super`; no UK-style state pension concept

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/finboyes-rgb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
