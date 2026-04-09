## article-mind-ui

> SvelteKit-based frontend for the Article Mind knowledge management system. This application provides a modern, reactive user interface for managing and exploring article collections with AI-powered insights.

# Article Mind UI - Frontend Development Guide

## Project Overview

SvelteKit-based frontend for the Article Mind knowledge management system. This application provides a modern, reactive user interface for managing and exploring article collections with AI-powered insights.

## Technology Stack

- **Framework:** SvelteKit 2.x with Svelte 5 (Runes API)
- **Language:** TypeScript 5.x (strict mode)
- **Build Tool:** Vite 6.x
- **Package Manager:** npm
- **Node Version:** 22.13.1 (managed by ASDF)

## Development Commands

### Standard Workflow

```bash
make install    # Install dependencies
make dev        # Start dev server (http://localhost:13000)
make build      # Build for production
make preview    # Preview production build
make test       # Run unit tests
make lint       # Check code quality
make format     # Format code
make check      # Run all quality checks (lint + format + test)
```

### API Type Generation

When the backend API changes:

```bash
make gen-api
# or
npm run gen:api
```

This fetches the OpenAPI spec from http://localhost:13010/openapi.json and generates TypeScript types in `src/lib/api/generated.ts`.

## Project Structure

```
src/
├── lib/
│   ├── api/              # API client and generated types
│   │   ├── client.ts     # Base API client
│   │   └── generated.ts  # Generated from OpenAPI (DO NOT EDIT MANUALLY)
│   └── components/       # Reusable Svelte components
├── routes/
│   ├── +layout.svelte    # Root layout with header
│   └── +page.svelte      # Home page
└── app.html              # HTML template
```

## API Contract Integration

### Overview

This project consumes the article-mind-service REST API following a **frozen contract specification**. The contract ensures type-safe communication between frontend and backend through automatic TypeScript type generation from the backend's OpenAPI specification.

**Philosophy**: Backend defines API contracts via Pydantic schemas → FastAPI auto-generates OpenAPI spec → Frontend auto-generates TypeScript types → Type-safe integration guaranteed.

### Contract Location

- **Frontend contract**: `docs/api-contract.md` (this project)
- **Backend contract**: `article-mind-service/docs/api-contract.md`

**CRITICAL**: Both contracts MUST stay synchronized. Any API changes require:

1. Backend updates contract and implementation
2. Backend deploys to development
3. Frontend runs `make gen-api` to regenerate types
4. Frontend updates code using new types
5. Both teams test integration

### Type Generation Workflow

#### 1. Backend Schema Change

Backend developer updates Pydantic schema in `article-mind-service/schemas/`:

```python
# article-mind-service/schemas/health.py
from pydantic import BaseModel
from typing import Literal

class HealthResponse(BaseModel):
    status: Literal["ok", "degraded", "error"]
    version: str
    database: Literal["connected", "disconnected"]
```

#### 2. Generate TypeScript Types

```bash
cd article-mind-ui
make gen-api  # Fetches OpenAPI and generates types
```

**What this does:**

- Fetches `http://localhost:13010/openapi.json` from backend
- Runs: `openapi-typescript http://localhost:13010/openapi.json -o src/lib/api/generated.ts`
- Overwrites `src/lib/api/generated.ts` with new types

**Requirements:**

- Backend must be running on http://localhost:13010
- Backend `/openapi.json` endpoint must be accessible
- `openapi-typescript` package must be installed

#### 3. Verify Generated Types

```typescript
// src/lib/api/generated.ts (auto-generated - DO NOT EDIT MANUALLY)
export interface HealthResponse {
	status: 'ok' | 'degraded' | 'error'; // Exact literal types from backend
	version: string;
	database: 'connected' | 'disconnected';
}
```

**Key benefits:**

- Literal types (`"ok" | "degraded" | "error"`) match backend exactly
- TypeScript catches invalid values at compile time
- IDE autocomplete shows exact allowed values
- Breaking changes fail TypeScript compilation

#### 4. Use in Components

```typescript
import type { HealthResponse } from '$lib/api/generated';
import { apiClient } from '$lib/api/client';

// Fully type-safe API call
const health = await apiClient.get<HealthResponse>('/health');

// TypeScript knows exact type
console.log(health.status); // "ok" | "degraded" | "error"
console.log(health.version); // string
console.log(health.database); // "connected" | "disconnected"

// TypeScript catches errors
// console.log(health.nonexistent); // ❌ Compile error!
```

### Guard Rails

#### Type Safety (CRITICAL)

- ✅ **ALWAYS** use generated types from `src/lib/api/generated.ts`
- ❌ **NEVER** manually type API responses
- ❌ **NEVER** use `any` or `@ts-ignore` on API calls
- ✅ **ALWAYS** run `make gen-api` after backend schema changes
- ❌ **NEVER** edit `src/lib/api/generated.ts` manually

**Why?**

- Compile-time safety prevents runtime errors
- IDE autocomplete shows exact API shape
- Breaking changes caught during development, not production
- Guaranteed alignment with backend implementation

**Example of violation:**

```typescript
// ❌ WRONG - Manual type definition
interface HealthResponse {
	status: string; // Too loose! Should be literal union
	version: string;
	database: string; // Missing constraints
}

// ✅ CORRECT - Use generated types
import type { HealthResponse } from '$lib/api/generated';
```

#### API Client Usage

- ✅ **ALWAYS** use `apiClient` from `src/lib/api/client.ts`
- ❌ **NEVER** use raw `fetch()` for API calls
- ✅ **ALWAYS** handle errors using client's error handling
- ❌ **NEVER** suppress API errors silently (no empty catch blocks)

**Why?**

- Consistent error handling across entire app
- Centralized request/response logging
- Easy to add authentication headers globally
- Request/response interceptors in single location
- Environment-specific base URL handling

**Example of violation:**

```typescript
// ❌ WRONG - Raw fetch
const response = await fetch('http://localhost:8000/health');
const health = await response.json();

// ❌ WRONG - Silent error suppression
try {
	const data = await apiClient.get('/api/v1/articles');
} catch (error) {
	// Silent failure - user has no idea what happened
}

// ✅ CORRECT - Use API client with error handling
import { apiClient } from '$lib/api/client';
import type { HealthResponse } from '$lib/api/generated';

try {
	const health = await apiClient.get<HealthResponse>('/health');
	// Handle success
} catch (error) {
	console.error('API error:', error);
	showErrorToast('Could not connect to server');
}
```

#### Environment Configuration

- ✅ **ALWAYS** use `VITE_API_BASE_URL` environment variable
- ❌ **NEVER** hardcode API URLs (http://localhost:8000, production URLs, etc.)
- ✅ Configure in `.env` for development
- ✅ Configure in `.env.production` for production
- ✅ Default to http://localhost:13010 if not set

**Why?**

- Easy environment switching (dev/staging/production)
- No accidental production API calls in development
- Deployment flexibility
- Single source of truth for API endpoint

**Example of violation:**

```typescript
// ❌ WRONG - Hardcoded URL
const response = await fetch('http://localhost:13010/health');

// ❌ WRONG - Hardcoded production URL
const API_URL = 'https://api.article-mind.com';

// ✅ CORRECT - Use environment variable
// In .env
VITE_API_BASE_URL=http://localhost:13010

// In code (apiClient handles this automatically)
const health = await apiClient.get('/health');
```

### Testing API Integration

#### Mock with Generated Types

**Always use generated types for test mocks to ensure type safety:**

```typescript
import type { HealthResponse } from '$lib/api/generated';
import { describe, it, expect } from 'vitest';

describe('Health Check', () => {
	it('should match API contract', () => {
		// Mock data must match generated type
		const mockHealth: HealthResponse = {
			status: 'ok',
			version: '1.0.0',
			database: 'connected'
		};

		// TypeScript enforces contract at compile time
		expect(mockHealth.status).toBe('ok');

		// This would fail TypeScript compilation:
		// const badMock: HealthResponse = {
		//   status: "invalid", // ❌ Not in "ok" | "degraded" | "error"
		//   version: "1.0.0",
		//   database: "connected"
		// };
	});
});
```

#### Test Contract Compliance

**Verify actual API responses match the contract:**

```typescript
import { apiClient } from '$lib/api/client';
import type { HealthResponse } from '$lib/api/generated';
import { describe, it, expect } from 'vitest';

describe('Health API', () => {
	it('returns valid health response', async () => {
		const health = await apiClient.get<HealthResponse>('/health');

		// Status must be one of allowed values
		expect(['ok', 'degraded', 'error']).toContain(health.status);

		// Database must be one of allowed values
		expect(['connected', 'disconnected']).toContain(health.database);

		// Version must be semantic versioning
		expect(health.version).toMatch(/^\d+\.\d+\.\d+$/);
	});

	it('handles degraded state correctly', async () => {
		const health = await apiClient.get<HealthResponse>('/health');

		// Contract: degraded status implies disconnected database
		if (health.status === 'degraded') {
			expect(health.database).toBe('disconnected');
		}
	});
});
```

#### Integration Test Pattern

```typescript
import { apiClient } from '$lib/api/client';
import { describe, it, expect, beforeAll } from 'vitest';

describe('API Integration Tests', () => {
	beforeAll(async () => {
		// Ensure backend is running and reachable
		const health = await apiClient.get('/health');
		expect(health.status).toBeDefined();
	});

	it('all endpoints follow generated types', async () => {
		// Test that real responses match generated TypeScript types
	});
});
```

### Common Pitfalls

#### ❌ Pitfall 1: Manual Type Definitions

See `docs/api-contract.md` section "Common Pitfalls" for detailed examples.

**Quick reference:**

```typescript
// ❌ WRONG
interface HealthResponse {
	status: string; // Too loose
}

// ✅ CORRECT
import type { HealthResponse } from '$lib/api/generated';
```

#### ❌ Pitfall 2: Not Regenerating Types

Backend adds new field → Frontend doesn't run `make gen-api` → Field missing in types → Runtime errors.

**Solution**: Always run `make gen-api` after backend changes.

#### ❌ Pitfall 3: Hardcoded URLs

**Solution**: Use `apiClient` which reads `VITE_API_BASE_URL`.

#### ❌ Pitfall 4: Silent Error Handling

**Solution**: Always log errors and show user feedback.

#### ❌ Pitfall 5: Using `any` or `@ts-ignore`

**Solution**: Fix type issues instead of suppressing them.

**Full pitfall documentation**: `docs/api-contract.md#common-pitfalls`

### Version Compatibility

#### Semantic Versioning

Backend uses semantic versioning (e.g., 1.0.0):

- **Major (1.x.x)**: Breaking changes - frontend MUST update
- **Minor (x.1.x)**: New features (backwards compatible)
- **Patch (x.x.1)**: Bug fixes (no API changes)

#### Version Checking

Frontend should check backend version compatibility:

```typescript
import { apiClient } from '$lib/api/client';
import type { HealthResponse } from '$lib/api/generated';

async function checkApiVersion() {
	const health = await apiClient.get<HealthResponse>('/health');
	const [major] = health.version.split('.').map(Number);

	const EXPECTED_MAJOR = 1;

	if (major !== EXPECTED_MAJOR) {
		throw new Error(`API version mismatch! Expected ${EXPECTED_MAJOR}.x.x, got ${health.version}`);
	}
}
```

### Pre-Commit Checklist

Before committing API integration code:

- [ ] Generated types are up to date (`make gen-api` run recently)
- [ ] No manual type definitions for API responses
- [ ] No `any` or `@ts-ignore` on API calls
- [ ] Using `apiClient` for all API requests (no raw `fetch()`)
- [ ] No hardcoded API URLs
- [ ] Error handling implemented (no silent failures)
- [ ] Tests include contract validation
- [ ] TypeScript compilation passes with no errors

### Workflow Summary

```
┌─────────────────────────────────────────────────────────────┐
│ Backend Change Flow                                         │
├─────────────────────────────────────────────────────────────┤
│ 1. Backend updates Pydantic schema in schemas/              │
│ 2. Backend implements endpoint with response_model          │
│ 3. Backend tests pass                                       │
│ 4. Backend deploys to development                           │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│ Frontend Update Flow                                        │
├─────────────────────────────────────────────────────────────┤
│ 1. Frontend runs: make gen-api                              │
│ 2. TypeScript types regenerated in lib/api/generated.ts    │
│ 3. TypeScript compilation shows any breaking changes       │
│ 4. Frontend updates code using new types                   │
│ 5. Frontend tests updated and passing                      │
│ 6. Integration tested against development backend          │
└─────────────────────────────────────────────────────────────┘
```

### Reference

- **Full API Contract**: `docs/api-contract.md`
- **Backend Contract**: `article-mind-service/docs/api-contract.md`
- **Backend OpenAPI**: http://localhost:13010/openapi.json
- **Backend Swagger UI**: http://localhost:13010/docs

### Quick Commands

```bash
# Generate TypeScript types from backend
make gen-api

# Start backend (for type generation)
cd article-mind-service && make dev

# Start frontend dev server
make dev

# Run API integration tests
make test
```

## Svelte 5 Runes API

This project uses Svelte 5 with the Runes API for state management.

### Reactive State

Use `$state` to create reactive variables:

```svelte
<script lang="ts">
	let count = $state(0); // Reactive state

	function increment() {
		count++; // Automatically triggers re-render
	}
</script>

<button onclick={increment}>
	Count: {count}
</button>
```

### Derived State

Use `$derived` for computed values:

```svelte
<script lang="ts">
	let count = $state(0);
	let doubled = $derived(count * 2); // Auto-updates when count changes
</script>

<p>Count: {count}, Doubled: {doubled}</p>
```

### Effects

Use `$effect` for side effects:

```svelte
<script lang="ts">
	let search = $state('');

	$effect(() => {
		console.log('Search changed:', search);
		// Fetch results, etc.
	});
</script>
```

### Props

Use `$props()` to receive component props:

```svelte
<script lang="ts">
	interface Props {
		title: string;
		count?: number;
	}

	let { title, count = 0 }: Props = $props();
</script>

<h1>{title}</h1><p>Count: {count}</p>
```

## Code Quality Standards

### ESLint Rules

- Enforce TypeScript strict mode
- Svelte-specific linting (accessibility, best practices)
- No unused variables or imports
- Consistent code style

### Prettier Configuration

- Tab width: 1 tab (not spaces)
- Single quotes
- Trailing commas: none
- Print width: 100 characters
- Svelte formatting via prettier-plugin-svelte

### Running Quality Checks

```bash
make lint          # ESLint
make format-check  # Prettier dry-run
make format        # Auto-format all files
make check         # Run all checks (lint + format + test)
```

## Testing Strategy

### Unit Tests (Vitest)

- Test business logic in `$lib` modules
- Test component behavior with @testing-library/svelte
- Location: `tests/unit/`

```bash
make test         # Run once (CI mode)
make test-watch   # Watch mode
```

### Writing Tests

```typescript
import { describe, it, expect } from 'vitest';
import { render } from '@testing-library/svelte';
import MyComponent from '$lib/components/MyComponent.svelte';

describe('MyComponent', () => {
	it('renders correctly', () => {
		const { getByText } = render(MyComponent, { props: { title: 'Test' } });
		expect(getByText('Test')).toBeTruthy();
	});
});
```

## Environment Variables

Configure via `.env` file (never commit `.env`, use `.env.example`):

```env
VITE_API_BASE_URL=http://localhost:13010
```

Access in code:

```typescript
const apiUrl = import.meta.env.VITE_API_BASE_URL;
```

## Common Pitfalls

### 1. Editing Generated Types

❌ **NEVER DO THIS:**

```typescript
// In src/lib/api/generated.ts
export interface HealthResponse {
	status: string;
	custom_field: string; // WRONG - will be overwritten
}
```

✅ **DO THIS INSTEAD:**

- Update backend Pydantic models
- Run `make gen-api`

### 2. API Base URL Hardcoding

❌ **WRONG:**

```typescript
fetch('http://localhost:13010/api/v1/health');
```

✅ **CORRECT:**

```typescript
apiClient.get('/api/v1/health');
```

### 3. Forgetting to Regenerate Types

After backend API changes, ALWAYS run:

```bash
make gen-api
```

### 4. Not Using Type Guards

When working with API responses, use type guards:

```typescript
function isHealthResponse(data: unknown): data is HealthResponse {
	return typeof data === 'object' && data !== null && 'status' in data;
}
```

### 5. Forgetting Svelte 5 Runes Syntax

❌ **OLD (Svelte 4):**

```svelte
<script lang="ts">
	export let count = 0; // Svelte 4 prop syntax
</script>
```

✅ **NEW (Svelte 5):**

```svelte
<script lang="ts">
	let { count = 0 } = $props(); // Svelte 5 Runes syntax
</script>
```

### 6. Watch Mode in CI Environments

❌ **WRONG:**

```bash
npm test  # May trigger watch mode
```

✅ **CORRECT:**

```bash
make test  # Uses vitest run (no watch)
```

## Development Workflow

1. **Start dev server:** `make dev`
2. **Make changes** to Svelte components or TypeScript
3. **Hot reload** happens automatically
4. **Run tests:** `make test` (in watch mode: `make test-watch`)
5. **Check quality:** `make lint && make format-check`
6. **Format code:** `make format`
7. **Build:** `make build`

## ASDF Version Management

Node.js version is pinned in `.tool-versions`:

```
nodejs 22.13.1
```

To install:

```bash
asdf install
```

## Component Development Guidelines

### File Naming

- Components: PascalCase (e.g., `ArticleCard.svelte`)
- Routes: kebab-case with + prefix (e.g., `+page.svelte`, `+layout.svelte`)
- Utilities: camelCase (e.g., `formatDate.ts`)

### Component Structure

```svelte
<script lang="ts">
	// 1. Imports
	import { apiClient } from '$lib/api/client';

	// 2. Props
	interface Props {
		title: string;
	}
	let { title }: Props = $props();

	// 3. State
	let count = $state(0);

	// 4. Derived state
	let doubled = $derived(count * 2);

	// 5. Functions
	function increment() {
		count++;
	}

	// 6. Effects
	$effect(() => {
		console.log('Count changed:', count);
	});
</script>

<!-- 7. Template -->
<div>
	<h1>{title}</h1>
	<button onclick={increment}>Count: {count}</button>
</div>

<!-- 8. Styles -->
<style>
	div {
		padding: 1rem;
	}
</style>
```

### Accessibility

- Use semantic HTML elements
- Add ARIA labels where appropriate
- Ensure keyboard navigation works
- ESLint will warn about common accessibility issues

## Guard Rails

- ✅ **DO:** Use the API client for all backend requests
- ✅ **DO:** Regenerate types after backend changes
- ✅ **DO:** Write tests for new components
- ✅ **DO:** Run `make check` before committing
- ✅ **DO:** Use Svelte 5 Runes API for all state management
- ❌ **DON'T:** Edit generated type files manually
- ❌ **DON'T:** Hardcode API URLs
- ❌ **DON'T:** Commit `.env` files
- ❌ **DON'T:** Skip type checking (`// @ts-ignore` should be rare)
- ❌ **DON'T:** Use Svelte 4 syntax (no `export let` for props)

## Troubleshooting

### Dev Server Won't Start

**Issue:** Port 13000 already in use

**Solution:**

```bash
lsof -ti:13000 | xargs kill -9
make dev
```

### TypeScript Errors

**Issue:** `.svelte-kit/tsconfig.json` missing

**Solution:**

```bash
make dev  # Generates .svelte-kit directory
```

### API Type Generation Fails

**Issue:** `make gen-api` errors

**Solution:**

- Ensure backend is running on http://localhost:13010
- Verify `/openapi.json` endpoint is accessible
- Check `openapi-typescript` is installed

### Hot Reload Not Working

**Issue:** Changes don't reflect in browser

**Solution:**

```bash
# Restart dev server
make dev
# Clear browser cache
# Check Vite config has proper HMR settings
```

### ESLint and Prettier Conflicts

**Issue:** Prettier formats code, ESLint complains

**Solution:**

- Ensure all dependencies are installed
- Run `make format` then `make lint`

## Resources

- [SvelteKit Docs](https://kit.svelte.dev/docs)
- [Svelte 5 Runes](https://svelte.dev/docs/svelte/what-are-runes)
- [Vite Documentation](https://vitejs.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Vitest Documentation](https://vitest.dev/)
- [Testing Library - Svelte](https://testing-library.com/docs/svelte-testing-library/intro)

## Next Steps

After scaffolding is complete:

1. Implement health check display (Plan 05)
2. Generate API types from backend: `make gen-api`
3. Create reusable components
4. Add routing for article management
5. Implement state management for complex features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MacPhobos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MacPhobos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
