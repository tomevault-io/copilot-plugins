## listo

> - **Dev**: `npm run dev` (local development server)

# Agent Guidelines for Listo

## Build/Test/Check Commands
- **Dev**: `npm run dev` (local development server)
- **Build**: `npm run build` (production build)
- **Type Check**: `npm run check` (svelte-check with TypeScript)
- **Deploy**: `npm run deploy` (build + Cloudflare Pages deploy)
- No test runner configured - add tests if needed

## Code Style & Conventions

### TypeScript
- Strict mode enabled - all code must be fully typed
- Use interfaces for component props (e.g., `interface Props { ... }`)
- Prefer explicit types over `any`
- Import types: `import type { TypeName } from '...'`

### Svelte 5 Runes
- Use `$state()` for reactive state (not `let x = $state(value)`)
- Use `$derived()` for computed values
- Use `$effect()` for side effects
- Use `$props()` for component props with TypeScript interfaces
- Avoid Svelte 4 patterns (stores in components, reactive statements with `$:`)

### Imports & Organization
- Group imports: external → SvelteKit → $lib → types
- Use path aliases: `$lib`, `$app`, `$env`
- Named exports from `$lib/components/ui` (e.g., `import { Button, Input, Card } from '$lib/components/ui'`)

### Naming
- Components: PascalCase files and names (e.g., `Button.svelte`)
- Files: kebab-case for non-components (e.g., `toast.svelte.ts`)
- Functions: camelCase
- Types/Interfaces: PascalCase
- CSS: Tailwind classes (no custom CSS unless necessary)

### Error Handling
- Use try/catch with console.error for logging
- Return result objects: `{ success: boolean, data?: T, error?: string }`
- Show user-friendly errors via toast: `toastStore.error('message')`
- Validate all user input before processing

### API Routes
- Use SvelteKit RequestHandler type
- Platform env vars: `platform?.env?.KEY || FALLBACK`
- Return `json()` responses with proper status codes
- Always handle errors and return appropriate responses

## Architecture Notes
- SvelteKit + Cloudflare Pages (edge runtime)
- IndexedDB for local storage (Dexie)
- WebAuthn for authentication
- Mobile-first, PWA-enabled
- Follow the "Lazy Days" design system from @llm/design.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonnyparris)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jonnyparris)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
