## vibe-coding

> "Mon Budget" is a personal budgeting PWA for teenagers. Frontend-only application (local-first):

# AGENTS.md - AI Agent Guidelines

## Project Overview

"Mon Budget" is a personal budgeting PWA for teenagers. Frontend-only application (local-first):
- **Frontend**: SvelteKit 5 (JavaScript, not TypeScript) with static adapter, IndexedDB storage
- **Deployment**: GitHub Pages (branch `trunk`) with `BASE_PATH` support
- **Locale**: French (fr-FR)

## Build & Run Commands

### Frontend (from `/frontend`)
```bash
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Production build (runs prebuild/postbuild scripts)
npm run preview      # Preview production build
```

### Environment Variables
```bash
BASE_PATH=           # Empty for dev; set to /repo-name for GitHub Pages (e.g. /vibe-coding)
```

### Build Scripts (auto-run)
- `scripts/generate-manifest.js` — injects `BASE_PATH` into `static/manifest.json` from template
- `scripts/copy-404.js` — copies `index.html` → `404.html` for GitHub Pages SPA routing

## Testing

### Framework: Playwright (E2E only) + axe-core (accessibility)

Test files are in `/frontend/e2e/`.

```bash
npm run test:e2e                              # Run all tests
npx playwright test e2e/transactions.spec.ts  # Single file
npx playwright test -g "create-expense"       # By name pattern
npm run test:e2e:headed                       # With visible browser
npm run test:e2e:ui                           # Interactive UI mode
```

### Test Files

| File | Coverage |
|------|----------|
| `transactions.spec.ts` | CRUD transactions, type/category switching, edit prefill, delete |
| `dashboard.spec.ts` | Stats, recent transactions (limit 5), balance styling, monthly isolation |
| `budget.spec.ts` | Set/update budget, progress bar states, month navigation, category exclusion |
| `goals.spec.ts` | CRUD goals, +1/+10/-1/-10 increments, floor at 0, achieve badge, validation |
| `navigation.spec.ts` | Desktop navbar, mobile bottom nav (375×667), active link, responsive behavior |
| `accessibility.spec.ts` | axe-core scans on all pages (empty + with-data states), zero violations expected |
| `charts.spec.ts` | BarChart/PieChart/LineChart visibility (hidden when no data), period toggle |
| `integration.spec.ts` | Cross-page: transaction affects budget/dashboard, edit/delete propagation |

### Writing Tests
- Tests use French UI labels (e.g., "Ajouter", "Supprimer", "Aucune transaction")
- Use `data-testid` attributes when adding new testable elements
- Chart wrappers use `data-testid="bar-chart"`, `"pie-chart"`, `"line-chart"`
- Follow existing patterns in `e2e/*.spec.ts`

## Application Features

### Pages & Routes

| Route | Description |
|-------|-------------|
| `/` | Dashboard — solde, stats mensuelles, transactions récentes (5), BarChart |
| `/transactions` | CRUD transactions, switch type income/expense, categories par type |
| `/budget` | Budget mensuel, navigation ← →, PieChart dépenses, LineChart historique |
| `/goals` | Objectifs d'épargne, +1/+10/-1/-10, badge "Atteint", suppression |

### Transaction Categories

**Dépenses** : Alimentation, Transport, Loisirs, Shopping, Abonnements, Education, Autre

**Revenus** : Argent de poche, Job etudiant, Cadeaux, Autre

### Chart Components (SVG — no external library)

| Component | Usage | Props |
|-----------|-------|-------|
| `BarChart.svelte` | Dashboard — revenus vs dépenses (6 derniers mois) | `data: [{ month: 'YYYY-MM', income, expenses }]` |
| `PieChart.svelte` | Budget — dépenses par catégorie (mois courant) | `data: [{ category, amount }]` |
| `LineChart.svelte` | Budget — budget vs réel (toggle 6/12 mois) | `data: [{ month, budget, spent }]`, `period: 6 \| 12` |

Charts are hidden when there is no data to display.

### Navigation (dual system)
- **Desktop** (≥ 768px): top navbar (`.navbar`) with text links
- **Mobile** (< 768px): fixed bottom bar (`.bottom-nav`, `data-testid="bottom-nav"`) with emoji icons + labels + `jiggle` animation on active item

## Code Style Guidelines

### Frontend (JavaScript/Svelte)

**Imports**
```javascript
import { onMount } from 'svelte';    // Svelte imports first
import { db } from '$lib/db.js';     // Then lib imports with $lib alias
```

**Svelte Component Structure**
```svelte
<script>
  // 1. Imports  2. Props/state  3. Reactive ($:)  4. Functions
</script>
<svelte:head><title>Page - Mon Budget</title></svelte:head>
<!-- Template -->
<style>/* Scoped styles */</style>
```

**Formatting**
- Tabs for indentation
- Single quotes, no semicolons
- camelCase for variables/functions, kebab-case for CSS classes

**Event syntax — use Svelte 5**
```svelte
<!-- Correct (Svelte 5) -->
<button onclick={handler}>...</button>

<!-- Avoid (Svelte 4, deprecated) -->
<button on:click={handler}>...</button>
```

**Error Handling**
```javascript
try {
  result = await db.someOperation();
} catch (e) {
  error = e.message;
} finally {
  loading = false;
}
```

**Locale Formatting** (always fr-FR)
```javascript
new Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'EUR' }).format(amount)
new Date(dateStr).toLocaleDateString('fr-FR')
```

## CSS Guidelines

**Variables** (in `app.css`):

| Variable | Value | Usage |
|----------|-------|-------|
| `--primary` | `#a5b4fc` | Primary color |
| `--primary-dark` | `#818cf8` | Hover/active states |
| `--secondary` | `#34d399` | Success/income |
| `--danger` | `#fb9494` | Errors/delete |
| `--warning` | `#f59e0b` | Warning states (budget >75%) |
| `--bg` | `#0f172a` | Page background |
| `--bg-card` | `#1e293b` | Card background |
| `--bg-input` | `#334155` | Input background |
| `--text` | `#f8fafc` | Primary text |
| `--text-muted` | `#a8b8cc` | Secondary text |
| `--border` | `#475569` | Borders |
| `--radius` | `12px` | Border radius |
| `--bottom-nav-height` | `80px` | Mobile bottom nav height |

**Utility Classes**: `.card`, `.text-muted`, `.text-success`, `.text-danger`, `.mb-1`/`.mb-2`/`.mb-3`, `.mt-2`, `.flex`, `.flex-between`, `.grid`, `.grid-2`, `.form-group`, `.container`, `.euro`

**Mobile padding**: `main` gets `padding-bottom: calc(80px + env(safe-area-inset-bottom))` on `< 768px` to account for the bottom nav.

## Database (IndexedDB only)

DB name: `mon-budget`, version: `1`

**Stores**

| Store | keyPath | Indexes | Fields |
|-------|---------|---------|--------|
| `transactions` | `id` (autoIncrement) | `date`, `type`, `category` | `type` ('income'\|'expense'), `amount`, `category`, `description`, `date` (YYYY-MM-DD), `createdAt` |
| `budgets` | `month` | — | `month` (YYYY-MM), `amount` |
| `goals` | `id` (autoIncrement) | — | `name`, `target_amount`, `current_amount` (default 0), `achieved` (default false), `createdAt` |

`getDashboard()` in `lib/db.js` computes all stats client-side: balance, monthly income/expenses, total income/expenses, last 5 recent transactions.

## Deployment (GitHub Pages)

- Branch: `trunk`
- CI: `.github/workflows/deploy.yml`
- `BASE_PATH` is set to `/${{ github.event.repository.name }}` in CI
- `svelte.config.js` injects `BASE_PATH` into `paths.base` at build time
- `static/.nojekyll` disables Jekyll processing
- `404.html` (copy of `index.html`) handles SPA client-side routing

Always use `$app/paths` `base` when building internal links to ensure compatibility with GitHub Pages subdirectory deployment.

## Code Review Principles

- **Challenge** architecture and implementation choices actively
- **Flag** any compromise on strong typing or functional purity
- **Never validate** code without appropriate tests
- Provide **concrete alternatives** when identifying problems
- Be **token-efficient** — alert before expensive operations

## File Structure

```
frontend/src/
  lib/
    db.js               # IndexedDB wrapper + getDashboard()
    BarChart.svelte     # SVG bar chart (income vs expenses)
    PieChart.svelte     # SVG donut chart (expenses by category)
    LineChart.svelte    # SVG line chart (budget vs actual)
  routes/
    +layout.svelte      # Dual navigation (desktop navbar + mobile bottom nav)
    +page.svelte        # Dashboard
    transactions/+page.svelte
    budget/+page.svelte
    goals/+page.svelte
  app.css               # Global styles + CSS variables
  app.html              # HTML shell (lang="fr", PWA meta)
  service-worker.js     # Cache-first PWA offline support

frontend/static/
  manifest.json         # Generated at build (do not edit directly)
  manifest.json.template # Source for manifest (edit this)
  .nojekyll             # GitHub Pages: disable Jekyll
  icons/                # PWA icons (192, 512, svg)

frontend/scripts/
  generate-manifest.js  # Prebuild: injects BASE_PATH into manifest
  copy-404.js           # Postbuild: copies index.html → 404.html

frontend/e2e/           # Playwright tests (8 spec files)
```

## Known Issues

| Issue | Location | Severity |
|-------|----------|----------|
| `LineChart.svelte` uses Svelte 4 `on:click` syntax instead of Svelte 5 `onclick` | `src/lib/LineChart.svelte` | Low — deprecation warning |

@RTK.md

---
> Source: [StephaneTrebel/vibe-coding](https://github.com/StephaneTrebel/vibe-coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
