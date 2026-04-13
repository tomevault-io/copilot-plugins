## bulgariangermanlearningapp

> **Tech Stack**: SvelteKit 2 + Svelte 5 (Runes) + TypeScript strict + Tailwind v4

# AI Agent Instructions - Bulgarian-German Learning App

**Tech Stack**: SvelteKit 2 + Svelte 5 (Runes) + TypeScript strict + Tailwind v4  
**Deployment**: Static site to GitHub Pages (no backend)  
**Package Manager**: pnpm ONLY (never npm/yarn)

---

## 🚀 Essential Commands

### Development
```bash
pnpm install              # Install dependencies
pnpm run dev              # Start dev server (HMR enabled, port 5173)
pnpm run check            # TypeScript + Svelte validation
pnpm run lint             # ESLint
pnpm run simulate-ci      # Pre-commit: TS + lint + unit + E2E
```

### Testing
```bash
pnpm run test:unit        # Vitest (single file: pnpm run test:unit tests/unit/foo.test.ts)
pnpm run test:components  # Playwright component tests
pnpm run test:e2e         # Playwright E2E tests
pnpm run test:accessibility # WCAG 2.1 AA compliance
pnpm run test:all         # All test suites
```

### Build & Deploy
```bash
pnpm run build            # Production build
pnpm run build:gh-pages   # With base path for GitHub Pages
pnpm run preview          # Preview production build
git push origin main      # Auto-deploys via GitHub Actions
```

---

## 💻 Code Style Guidelines

### Svelte 5 Runes (REQUIRED)
```svelte
<script lang="ts">
  let count = $state(0);
  let doubled = $derived(count * 2);
  let { title = 'Default', items = [] } = $props();
  $effect(() => {
    const handler = () => console.log('resize');
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  });
  // ❌ Never: export let prop, $: reactive
</script>
```

### File Extensions
- `.svelte` → Components with markup (use runes)
- `.svelte.ts` → Reactive logic/state (use runes)
- `.ts` → Pure utilities (no reactivity)

### TypeScript Strictness
```typescript
// ✅ Strict mode - no 'any'
function processItem(item: VocabularyItem): string { ... }
// Use type guards for narrowing
if (typeof value === 'string') { ... }
```

### State Management
```typescript
// ✅ Always use singleton appState
import { appState } from '$lib/state/app-state';
appState.languageMode;  // 'DE_BG' | 'BG_DE'
// ❌ Never instantiate state classes directly
```

### Naming Conventions
- **Components**: PascalCase (`VocabularyCard.svelte`)
- **Files**: kebab-case (`app-state.ts`)
- **Functions**: camelCase (`handleClick`)
- **Constants**: UPPER_SNAKE_CASE
- **Types**: PascalCase (`VocabularyItem`)

### Imports
```typescript
import { foo } from '$lib/utils/foo';      // SvelteKit alias
import { Bar } from '$lib/components/Bar.svelte';
```

### Error Handling & Validation
```typescript
import { ErrorHandler } from '$lib/services/errors';
ErrorHandler.error(new Error('message'));
import { VocabularyItemSchema } from '$lib/schemas/vocabulary';
const item = VocabularyItemSchema.parse(rawData);  // Always validate external data
```

---

## 🧪 Testing Guidelines

### Unit Tests (Vitest)
```typescript
import { describe, it, expect } from 'vitest';
describe('Feature', () => {
  it('should do something', () => { expect(true).toBe(true); });
});
```

### Component Tests (Playwright CT)
```typescript
import { test, expect } from '@playwright/experimental-ct-svelte';
import MyComponent from '$lib/components/MyComponent.svelte';
test('renders correctly', async ({ mount }) => {
  const component = await mount(MyComponent, { props: { title: 'Test' } });
  await expect(component).toContainText('Test');
});
```

### E2E Tests (Playwright)
```typescript
import { test, expect } from '@playwright/test';
test('can search vocabulary', async ({ page }) => {
  await page.goto('/vocabulary');
  await page.fill('input[type="search"]', 'Hallo');
  await expect(page.locator('.vocabulary-item')).toContainText('Здравей');
});
```

---

## 🛠️ MCP Tools (Critical for Svelte Work)

### Svelte MCP Server Workflow
1. `list-sections` → Get available Svelte 5/SvelteKit docs sections
2. `get-documentation` → Fetch relevant sections (fetch ALL at once)
3. Write component using official patterns
4. `svelte-autofixer` → Validate before finalizing (**REQUIRED**)
5. Fix all issues found
6. `pnpm run check` → Verify TypeScript

---

## ⚠️ Critical Rules

### NEVER
1. Don't restart dev server (HMR enabled) - check: `lsof -iTCP:5173 | grep LISTEN`
2. Don't use legacy Svelte syntax (`export let`, `$:`)
3. Don't use `any` type - use proper types or type guards
4. Don't skip Zod validation for external data
5. Don't create new state instances - use singleton `appState`
6. Don't use npm/yarn - pnpm ONLY

### ALWAYS
1. Run `pnpm run simulate-ci` before committing
2. Validate data with Zod schemas
3. Use singleton `appState` for state management
4. Add tests for new features (unit required, component recommended)
5. Use Svelte MCP for all Svelte development
6. Validate components with `svelte-autofixer` before finalizing

---

## 📁 Key Files & Structure

```
src/
├── lib/
│   ├── components/       # UI components (Svelte 5 runes)
│   ├── data/             # Data loading (loader.ts, db.svelte.ts)
│   ├── schemas/          # Zod validation schemas
│   ├── services/         # Business logic (event-bus.ts, errors.ts)
│   ├── state/            # Global state (app-state.ts singleton)
│   ├── types/            # TypeScript types
│   └── utils/            # Utility functions
├── routes/               # SvelteKit file-based routing
└── paraglide/            # i18n translations
data/
└── unified-vocabulary.json   # 746 vocabulary items
```

### Critical Files
- **State**: `src/lib/state/app-state.ts` (singleton facade)
- **Data**: `src/lib/data/loader.ts`, `db.svelte.ts`
- **Schema**: `src/lib/schemas/vocabulary.ts`
- **Services**: `src/lib/services/event-bus.ts`, `errors.ts`
- **Example**: `src/lib/components/ui/VocabularyCard.svelte`

---

## 📝 Commit Messages

Format: `<type>(<scope>): <subject>`  
**Types**: feat, fix, docs, style, refactor, perf, test, chore  
**Example**: `feat(vocabulary): add enriched definitions`

---

## 📚 Documentation

- **Single Source of Truth**: `docs/PROJECT_STATUS.md`
- **Grammar Reference**: `docs/GERMAN_BULGARIAN_GRAMMAR_GUIDE.md`
- **Architecture**: `docs/architecture/ARCHITECTURE.md`
- **Testing**: `docs/development/TESTING.md`
- **Quick Start**: `docs/GETTING_STARTED.md`

---

## 🐛 Common Issues

- **Port 5173 in use**: `pkill -f "pnpm dev"`
- **TypeScript errors**: `rm -rf node_modules .svelte-kit && pnpm install`
- **Tests failing in CI**: `CI=true pnpm run test:all`
- **Vocabulary not loading**: Verify `data/unified-vocabulary.json` exists

---

**Version**: 3.0.0 | **Last Updated**: March 2026 | **Maintained By**: YungSeepferd

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YungSeepferd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YungSeepferd)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
