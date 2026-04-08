## tickytock

> > **TL;DR**: This is a fully reactive Svelte 5 app with encrypted storage. Read the skill guides below based on what you're working on.

# AI Agent Guide for TickyTock

> **TL;DR**: This is a fully reactive Svelte 5 app with encrypted storage. Read the skill guides below based on what you're working on.

## Quick Start

1. ✅ Read [Architecture](./docs/ai/architecture.md) - **ESSENTIAL** - Read this first!
2. 🎯 Pick relevant skill guides based on your task
3. ⚠️ Check [Common Pitfalls](./docs/ai/common-pitfalls.md) before committing

## Available Skills

### 📐 [Architecture](./docs/ai/architecture.md) ⭐ **READ FIRST**

Data flow, reactive patterns, and core principles.

**Read this if:**

- New to the project
- Working on stores or business logic
- Debugging reactive state issues
- Adding new features

**Key topics:**

- Fire-and-forget mutations (return `Promise<void>`)
- Event-driven state updates via EncryptedPouch
- 3-layer architecture (Component → ActivityStore → DataStore)
- Reactive patterns with `$state` and `$derived`
- Multi-account support and sync

---

### 🎨 [UI Framework](./docs/ai/ui-framework.md)

Tabler UI components, utilities, and styling guidelines.

**Read this if:**

- Creating or updating UI components
- Working on forms, buttons, layouts
- Adding styles or CSS
- Improving accessibility

**Key topics:**

- Use Tabler classes, avoid custom CSS
- Semantic color classes (primary, success, danger, etc.)
- Responsive design utilities
- Accessibility best practices
- When custom CSS is acceptable

---

### 🧪 [E2E Testing](./docs/ai/e2e-testing.md)

End-to-end testing with Cucumber and Playwright.

**Read this if:**

- Writing or updating E2E tests
- Fixing failing E2E tests
- Adding ARIA attributes for accessibility
- Creating new feature files

**Key topics:**

- Locator priority: Role-based → Labels → Placeholders
- Never use CSS selectors or generic selectors
- Always inspect component code before writing tests
- Add missing ARIA attributes to components first
- Test user behavior, not implementation

---

### 📁 [Code Organization](./docs/ai/code-organization.md)

File structure, naming conventions, and where things go.

**Read this if:**

- Creating new files
- Unsure where to put code
- Adding new features
- Understanding the project structure

**Key topics:**

- Directory structure and file naming
- Import patterns and dependency rules
- Where to add types, stores, components, utilities
- Package.json scripts reference

---

### ⚠️ [Common Pitfalls](./docs/ai/common-pitfalls.md)

Anti-patterns to avoid and lessons learned.

**Read this if:**

- Something feels overly complex
- Tests are failing mysteriously
- Getting type errors or lint errors
- Wondering why a pattern exists

**Key topics:**

- Don't return values from mutations
- Don't manually update `$state`
- Don't use `onMount` for data loading
- Don't import deleted files (e.g., `activities.svelte.ts`)
- Don't write custom CSS without checking Tabler first

---

## Project Info

- **Framework**: Svelte 5 (with runes: `$state`, `$derived`, `$effect`)
- **Language**: TypeScript (strict mode)
- **UI**: Tabler UI (Bootstrap-based)
- **Storage**: @mrbelloc/encrypted-pouch (custom library)
- **Build**: Vite
- **Tests**: Vitest (unit) + Cucumber/Playwright (E2E)
- **Routing**: svelte-spa-router

## Core Principles

1. **Fully Reactive** - All data flows through events, UI updates automatically
2. **Fire-and-Forget** - Mutations return `Promise<void>`, events update state
3. **Tabler Native** - Use Tabler components, avoid custom CSS
4. **Accessibility First** - Proper ARIA attributes enable better tests
5. **3-Layer Architecture** - Component → ActivityStore → DataStore

## Quick Commands

```bash
# Development
npm run dev              # Start dev server
npm run dev:e2e          # Start dev server for E2E tests

# Testing
npm test                 # Run unit tests (watch)
npm run test:run         # Run unit tests (CI)
npm run test:e2e         # Run E2E tests (headless)
npm run test:e2e:headed  # Run E2E tests (visible browser)

# Quality
npm run check            # Type check (src + e2e)
npm run lint             # Lint all files
npm run lint:fix         # Fix lint issues
npm run format           # Format all files

# Build
npm run build            # Production build
npm run preview          # Preview production build
```

## Common Workflows

### Adding a New Feature

1. Define types in `src/lib/types.ts`
2. Add storage logic in `src/lib/dataStore.svelte.ts` (if needed)
3. Add business logic in `src/lib/activityStore.svelte.ts`
4. Create component in `src/components/`
5. Add E2E test in `e2e/features/`

### Fixing a Bug

1. Read [Architecture](./docs/ai/architecture.md) if data flow related
2. Check [Common Pitfalls](./docs/ai/common-pitfalls.md) for known issues
3. Add test to reproduce bug
4. Fix the issue
5. Verify tests pass

### Updating UI

1. Check [UI Framework](./docs/ai/ui-framework.md) for Tabler patterns
2. Search Tabler docs for existing components
3. Update component with Tabler classes
4. Add ARIA attributes for accessibility
5. Update E2E tests if needed

### Writing E2E Tests

1. Read [E2E Testing](./docs/ai/e2e-testing.md)
2. Inspect the component code first
3. Add missing ARIA attributes to component
4. Write feature file in Gherkin
5. Write step definitions with role-based locators
6. Run tests: `npm run test:e2e:headed`

## When Stuck

1. 📖 Re-read [Architecture](./docs/ai/architecture.md) - most issues relate to data flow
2. 🔍 Search existing components for similar patterns
3. 📚 Check Tabler UI docs for UI patterns
4. 🐛 Run tests to isolate the issue
5. 💬 Ask for help if complexity seems unusual

## Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Deep technical dive (500+ lines)
- **[README.md](./README.md)** - User-facing overview
- **[docs/ai/](./docs/ai/)** - Skill-specific guides (this index)

## Version Info

Last major refactor: **January 2025**

**What changed:**

- Hybrid reactive/imperative → Fully reactive
- Mutations return docs → Mutations return `Promise<void>`
- Manual state updates → Event-driven state updates
- Deleted `activities.svelte.ts` redundant layer
- 4+ layers → 3-layer architecture

If you see old patterns in code, update them to the new reactive pattern.

---

**Remember**: Simple, reactive, Tabler-native code is the goal. When in doubt, trust the event system and read the docs!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pablolb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pablolb)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
