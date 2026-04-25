## svelte-sandbox

> **NEVER IMPLEMENT CODE WITHOUT IMMEDIATELY TESTING IN BROWSER**

# Project Memory

# 🚨 CRITICAL DEVELOPMENT RULE 🚨
**NEVER IMPLEMENT CODE WITHOUT IMMEDIATELY TESTING IN BROWSER**
- Test EVERY single change in the browser before proceeding
- If you write ANY code, you MUST verify it works in the browser
- If you cannot test (server down, etc.), STOP and ask user to start server
- NO EXCEPTIONS - this is mandatory for ALL code changes

## Implementation Workflow

⚠️ **CRITICAL REQUIREMENT: ALWAYS TEST IN BROWSER DURING DEVELOPMENT** ⚠️

1. **Before starting work**: 
   - Ensure you understand the requirements
   - Confirm backend server is running on port 8101, if not request user to start it

2. **During development** (MUST DO AFTER EACH CHANGE):
   - Follow the established patterns and conventions
   - Use existing components and utilities
   - Write clean, maintainable code
   - **🚨 MANDATORY: Test EVERY change immediately in the browser**
   - **🚨 MANDATORY: Navigate to the relevant page and verify functionality works**
   - **🚨 MANDATORY: Test user interactions, form submissions, and UI behavior**
   - **🚨 MANDATORY: Check browser console for any errors**
   - **🚨 MANDATORY: Do NOT proceed to next change until current change is verified working**

3. **After implementing ALL changes**:
   - **FINAL VERIFICATION: Re-test all modified pages in the browser**
   - Verify mobile responsiveness if applicable
   - Confirm no console errors exist

4. **Before committing**:
   - Run `pnpm check` to ensure code quality, fix bugs
   - Run `pnpm format` to ensure code formatting, fix formatting issues
   - Run tests if applicable
   - Review changes for quality

5. **Git workflow**:
   - Commit changes with descriptive messages
   - Rebase if needed to keep history clean
   - Push changes to remote repository

## Core Architecture Principles

This application follows a **vertical architecture** with **minimal abstractions** for **maximum developer experience**:

### 🎯 Minimal Abstractions
- **Use OpenAPI-generated types directly** - no custom wrappers or API client classes
- **Direct openapi-fetch calls** in actions - no intermediate API layers  
- **Minimal custom interfaces** 

### 🏗️ Vertical Feature Organization  
- Each feature is **self-contained** in `src/lib/app/{feature}/`
- **High cohesion** within features, **loose coupling** between features
- Feature exports are **barreled** through `index.ts` for clean imports

### ⚡ Developer Experience Focus
- **Straightforward patterns** - easy to learn, easy to follow
- **Consistent error handling** with simple `{data, error}` structure  
- **Type-safe** throughout with TypeScript and Zod validation
- **Minimal cognitive overhead** - few concepts to learn
- **Vertical architecture** - business logic and UI clearly separated

### 🚀 Easy Feature Addition
Adding a new feature follows a **predictable, mechanical pattern**:
1. **Business Logic**: Create in `lib/app/{feature}/`
   - Define types in `{feature}-api.ts` (import from OpenAPI)
   - Create actions in `actions/` (use openapi-fetch directly)
   - Add configuration in `config.ts`
   - Export through `index.ts` (barrel exports)
2. **User Interface**: Create in `routes/{feature}/`
   - Build components in `components/` (route-specific)
   - Create pages with loaders/actions using feature business logic
   - Co-locate UI components with routes that use them

## Code style

- Use kebab-case for file naming
- Run `pnpm format` before you commit

## UI Components

- DO NOT change any files directly in the `lib/components/ui` folder - these are shadcn-svelte components
- Install new shadcn components using `pnpm dlx shadcn-svelte@latest add [component name] -y -o`.
- Component documentation is available via context7 mcp. Library: huntabyte/shadcn-svelte
- Use existing shadcn-svelte components when possible

## UI Conventions

- Always use `import type { PageProps } from './$types';` for +page.svelte files, when properties are needed;
- Always try to use the types from $types if available for +server.ts and +page.ts
- Always use a interface to declare svelte component props

```typescript [cashier-card.svelte]
interface Props {
	cashier: GetCashiersResult;
	ondeleted: (item: GetCashiersResult) => void;
}
```

## Error Handling Pattern

Actions use openapi-fetch directly and return a simplified Result type with `{data, error}` structure:

```typescript
import { ok, error, type ActionResult } from '$lib/utils/action-result';
import client from '$lib/api/billing';
import type { ZodError } from 'zod';

// Error result for all operations  
type ActionError = {
	error: string;
	code: number;
	validationErrors?: Record<string, string[]>;
};

// Query pattern - returns data
export const getItemQuery = async (id: string): Promise<ActionResult<Item>> => {
	if (!id?.trim()) {
		return error('Item ID is required', 400);
	}

	const { data, response } = await client.GET('/Items/{id}', {
		params: { path: { id } }
	});

	if (data) {
		return ok(data);
	}

	if (response?.status === 404) {
		return error('Item not found', 404);
	}
	return error('Failed to load item. Please try again later.', response?.status || 500);
};

// Command pattern - returns data 
export const createItemCommand = async (request: CreateItemRequest): Promise<ActionResult<Item>> => {
	const { data, response } = await client.POST('/Items', {
		body: request as any
	});

	if (data) {
		return ok(data);
	}

	if (response?.status === 400) {
		return error('Invalid request data', 400);
	}
	return error('Failed to create item. Please try again later.', response?.status || 500);
};

// Void operations pattern (DELETE, PUT operations that return no data)
export const deleteItemCommand = async (id: string): Promise<ActionResult<void>> => {
	if (!id?.trim()) {
		return error('Item ID is required', 400);
	}

	const { error: apiError, response } = await client.DELETE('/Items/{id}', {
		params: { path: { id } }
	});

	if (!apiError) {
		return ok();
	}

	if (response?.status === 404) {
		return error('Item not found', 404);
	}
	return error('Failed to delete item. Please try again later.', response?.status || 500);
};
```

In page loaders, check for `result.error`:

```typescript
import { error } from '@sveltejs/kit';

export const load: PageServerLoad = async ({ params }) => {
	const result = await getItemQuery(params.id);

	if (!result.error) {
		return {
			item: result.data
		};
	}

	return error(result.error.code, result.error.message);
};
```

# Feature Implementation Pattern

## File Structure

The application uses **vertical architecture** with **business logic** in features and **UI** in routes:

```
src/lib/app/{feature}/
├── {feature}-schema.ts      # Zod v4 validation schemas + OpenAPI types
├── {feature}-constants.ts   # Feature constants (statuses, enums, etc.)
├── config.ts                # Feature configuration (API endpoints, permissions, business rules)
├── index.ts                 # Public exports (barreled import)
├── components/              # Route-specific UI components
│   ├── {item}-form.svelte   # Form component (example)
│   ├── {item}-card.svelte   # Card component (example)
│   └── {item}-table.svelte  # Table component (example)
└── actions/                 # Business logic (commands & queries) with co-located parameter types
    ├── get-{items}.ts       # List query with GetItemsQuery interface (example)
    ├── get-{item}.ts        # Single item query with GetItemParams interface (example)
    ├── create-{item}.ts     # Create command with CreateItemParams interface (example)
    ... (other actions)

src/routes/{feature}/
├── +page.svelte            # List page
├── +page.server.ts         # List loader
├── create/ (example)
│   ├── +page.svelte        # Create page
│   └── +page.server.ts     # Create actions
└── [id]/
    ├── +page.svelte        # Details page (example)
    ├── +page.server.ts     # Details loader (example)
    ├── +server.ts          # API endpoint e.g. DELETE (example)
    └── edit/ (example)
        ├── +page.svelte    # Edit page
        └── +page.server.ts # Edit loader + actions
    ... (other routes)
 ... (other routes)

src/lib/infrastructure/      # Cross-cutting concerns (opt-in)
├── api/                     # API infrastructure
├── auth/                    # Authentication infrastructure
├── config/                  # App configuration
├── error/                   # Error handling infrastructure
├── observability/           # OpenTelemetry setup
├── performance/             # Performance monitoring
└── state/                   # Global state management
```

## API Types Pattern

Use OpenAPI-generated types directly in the schema file, with parameter interfaces co-located in action files:

```typescript
// {feature}-schema.ts
import type { components } from '$lib/api/billing/v1';

// Use OpenAPI generated types directly - maximum type safety, zero abstractions
export type Item = components['schemas']['Item'];
export type GetItemsResult = Item; // or components['schemas']['GetItemsResult'] if different
export type CreateUpdateItemRequest = components['schemas']['CreateItemCommand'];

// Custom types only when needed for client-side logic
export interface ItemSummary {
	totalItems: number;
	totalValue: number;
}

// Form validation schemas
export const createUpdateItemSchema = z.object({
	name: z.string().min(2, 'Name must be at least 2 characters'),
	amount: z.number().positive('Amount must be greater than 0')
});

export type CreateUpdateItemSchema = typeof createUpdateItemSchema;
```

```typescript
// actions/get-items.ts - Parameter interfaces co-located with their actions
import { ok, error, type ActionResult } from '$lib/utils/action-result';
import client from '$lib/api/billing';
import type { GetItemsResult } from '../{feature}-schema';

export interface GetItemsQuery {
	page?: number;
	pageSize?: number;
	search?: string;
	sortBy?: string;
	sortOrder?: 'asc' | 'desc';
	status?: string;
}

export const getItemsQuery = async (query?: GetItemsQuery): Promise<ActionResult<GetItemsResult[]>> => {
	// Implementation...
};
```

**Key Principles:**
- **OpenAPI types in schema**: All API-generated types in `{feature}-schema.ts`
- **Parameters co-located**: Action parameter interfaces defined in their respective action files  
- **Type safety**: Leverage generated types from your API spec
- **Minimal custom interfaces**: Only add when needed for UI logic
- **No API client classes**: Actions call openapi-fetch directly
- **Simplified parameters**: Actions with only a single `id` parameter should take `id: string` directly, not wrapped in an interface

## Action Patterns

### Query Pattern (Read Operations)

```typescript
// actions/get-{item}.ts
import { ok, error, type ActionResult } from '$lib/utils/action-result';
import client from '$lib/api/billing';
import type { Item } from '../{item}-schema';

export const getItemQuery = async (id: string): Promise<ActionResult<Item>> => {
	if (!id?.trim()) {
		return error('Item ID is required', 400);
	}

	const { data, response } = await client.GET('/Items/{id}', {
		params: { path: { id } }
	});

	if (data) {
		return ok(data);
	}

	if (response?.status === 404) {
		return error('Item not found', 404);
	}
	return error('Failed to load item. Please try again later.', response?.status || 500);
};
```

### List Query Pattern

```typescript
// actions/get-{items}.ts
import { ok, error, type ActionResult } from '$lib/utils/action-result';
import client from '$lib/api/billing';
import type { GetItemsResult } from '../{item}-schema';

export interface GetItemsQuery {
	page?: number;
	pageSize?: number;
	search?: string;
	sortBy?: string;
	sortOrder?: 'asc' | 'desc';
	status?: string;
}

export const getItemsQuery = async (query?: GetItemsQuery): Promise<ActionResult<GetItemsResult[]>> => {
	const { data, response } = await client.GET('/Items', {
		params: { query: query as any }
	});

	if (data) {
		const items = (data || []) as GetItemsResult[];
		// Optional: Sort or transform data
		items.sort((a, b) => a.name.localeCompare(b.name));
		return ok(items);
	}

	return error('Failed to load items. Please try again later.', response?.status || 500);
};
```

### Command Pattern (Write Operations)

```typescript
// actions/create-{item}.ts
import { ok, error, type ActionResult } from '$lib/utils/action-result';
import client from '$lib/api/billing';
import type { Item } from '../{item}-schema';

export interface CreateItemParams {
	name: string;
	amount: number;
	currency?: string;
	dueDate?: string;
	cashierId?: string;
}

export const createItemCommand = async (params: CreateItemParams): Promise<ActionResult<Item>> => {
	const { data, response } = await client.POST('/Items', {
		body: params as any
	});

	if (data) {
		return ok(data);
	}

	if (response?.status === 400) {
		return error('Invalid request data', 400);
	}
	if (response?.status === 409) {
		return error('Item already exists', 409);
	}
	return error('Failed to create item. Please try again later.', response?.status || 500);
};
```

### Update/Delete Patterns

```typescript
// actions/update-{item}.ts
export interface UpdateItemParams {
	id: string;
	name: string;
	amount: number;
	currency?: string;
	dueDate?: string;
	cashierId?: string;
}

export const updateItemCommand = async (params: UpdateItemParams): Promise<ActionResult<Item>> => {
	const { id, ...updateData } = params;
	if (!id?.trim()) {
		return error('Item ID is required', 400);
	}

	const { data, response } = await client.PUT('/Items/{id}', {
		params: { path: { id } },
		body: updateData as any
	});

	if (data) {
		return ok(data);
	}

	if (response?.status === 404) {
		return error('Item not found', 404);
	}
	if (response?.status === 400) {
		return error('Invalid request data', 400);
	}
	return error('Failed to update item. Please try again later.', response?.status || 500);
};

// actions/delete-{item}.ts - Uses error destructuring for void operations
export const deleteItemCommand = async (id: string): Promise<ActionResult<void>> => {
	if (!id?.trim()) {
		return error('Item ID is required', 400);
	}

	const { error: apiError, response } = await client.DELETE('/Items/{id}', {
		params: { path: { id } }
	});

	if (!apiError) {
		return ok();
	}

	if (response?.status === 404) {
		return error('Item not found', 404);
	}
	return error('Failed to delete item. Please try again later.', response?.status || 500);
};
```

## Validation Pattern

```typescript
// {item}-schema.ts
import { z } from 'zod';

export const createUpdate{Item}Schema = z.object({
	name: z
		.string()
		.min(2, 'Name must be at least 2 characters'),
	email: z
		.email('Please enter a valid email address')
});

export type CreateUpdate{Item}Schema = typeof createUpdate{Item}Schema;
```

## Component Patterns

### Form Component

```svelte
<!-- components/{item}-form.svelte -->
<script lang="ts">
	import { superForm, type Infer, type SuperValidated } from 'sveltekit-superforms';
	import { zod4Client as zodClient } from 'sveltekit-superforms/adapters';

	import * as Form from '$lib/components/ui/form/index.js';
	import { Input } from '$lib/components/ui/input';
	import { Card, CardHeader, CardTitle, CardContent } from '$lib/components/ui/card';
	import { Alert, AlertDescription, AlertTitle } from '$lib/components/ui/alert';
	import { Save, AlertCircle } from '@lucide/svelte';

	import { createUpdate{Item}Schema, type CreateUpdate{Item}Schema } from '../{item}-schema';

	interface Props {
		mode: 'create' | 'edit';
		data: {
			form: SuperValidated<Infer<CreateUpdate{Item}Schema>>;
			error?: string | null;
		};
	}

	let { mode, data }: Props = $props();

	const form = superForm(data.form, {
		validators: zodClient(createUpdate{Item}Schema)
	});

	const { form: formData, enhance, submitting, errors } = form;

	const formTitle = mode === 'create' ? 'Create {Item}' : 'Edit {Item}';
	const formDescription = mode === 'create' ? 'Add a new {item} to the system' : 'Update {item} information';

	const submitButtonText = $derived(
		$submitting
			? mode === 'create'
				? 'Creating...'
				: 'Updating...'
			: mode === 'create'
				? 'Create {Item}'
				: 'Update {Item}'
	);

	const mainError = $derived(data.error);
	const hasFormErrors = $derived($errors && Object.keys($errors).length > 0);
</script>

<div class="container mx-auto max-w-2xl p-6">
	<Card>
		<CardHeader>
			<CardTitle>{formTitle}</CardTitle>
			<p class="text-muted-foreground">{formDescription}</p>
		</CardHeader>
		<CardContent>
			{#if mainError}
				<Alert variant="destructive" class="mb-6">
					<AlertCircle class="h-4 w-4" />
					<AlertTitle>Error</AlertTitle>
					<AlertDescription>
						<p>{mainError}</p>
						{#if hasFormErrors}
							<ul class="mt-2 list-inside list-disc text-sm">
								{#each Object.entries($errors) as [field, fieldErrors]}
									{#if fieldErrors && fieldErrors?.length > 0}
										<li class="capitalize">{field}: {fieldErrors?.join(', ')}</li>
									{/if}
								{/each}
							</ul>
						{/if}
					</AlertDescription>
				</Alert>
			{/if}

			<form method="POST" use:enhance class="space-y-8">
				<Form.Field {form} name="name">
					<Form.Control>
						{#snippet children({ props })}
							<Form.Label>Name *</Form.Label>
							<Input {...props} bind:value={$formData.name} placeholder="Enter {item} name" disabled={$submitting} />
						{/snippet}
					</Form.Control>
					<Form.Description>Enter the {item}'s full name</Form.Description>
					<Form.FieldErrors />
				</Form.Field>

				<Form.Field {form} name="email">
					<Form.Control>
						{#snippet children({ props })}
							<Form.Label>Email *</Form.Label>
							<Input
								{...props}
								type="email"
								bind:value={$formData.email}
								placeholder="Enter email address"
								disabled={$submitting} />
						{/snippet}
					</Form.Control>
					<Form.Description>This will be used for login and notifications</Form.Description>
					<Form.FieldErrors />
				</Form.Field>

				<div class="flex gap-2">
					<Form.Button type="submit" disabled={$submitting}>
						<Save size={16} />
						{submitButtonText}
					</Form.Button>
					<Form.Button type="button" variant="outline" href="/{items}" disabled={$submitting}>Cancel</Form.Button>
				</div>
			</form>
		</CardContent>
	</Card>
</div>
```

### Card Component

```svelte
<!-- components/{item}-card.svelte -->
<script lang="ts">
	import { Button } from '$lib/components/ui/button';
	import { Card, CardContent } from '$lib/components/ui/card';
	import type { Get{Items}Result } from '../{items}-api';

	interface Props {
		{item}: Get{Items}Result;
		ondeleted: (item: Get{Items}Result) => void;
	}

	let { {item}, ondeleted }: Props = $props();
	let deleting = $state(false);

	async function delete{Item}() {
		deleting = true;
		try {
			await fetch(`/{items}/${{{item}.{item}Id}}`, { method: 'DELETE' });
			ondeleted({item});
		} catch (error) {
			console.error('Failed to delete {item}:', error);
		} finally {
			deleting = false;
		}
	}
</script>
```

## Page Patterns

### List Page

```svelte
<!-- routes/{items}/+page.svelte -->
<script lang="ts">
	import type { PageProps } from './$types';
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import { {Item}Card } from '$lib/app/{items}';

	let { data }: PageProps = $props();
	let {items} = $state(data.{items});
	let searchTerm = $state('');

	let filtered{Items} = $derived(
		{items}.filter(({item}) => {
			if (!searchTerm.trim()) return true;
			const searchLower = searchTerm.toLowerCase();
			return {item}.name.toLowerCase().includes(searchLower);
		})
	);

	const ondeleted = ({item}: Get{Items}Result) =>
		({items} = {items}.filter((i) => i.{item}Id !== {item}.{item}Id));
</script>
```

### Page Server Loaders

```typescript
// +page.server.ts (create)
import { superValidate } from 'sveltekit-superforms';
import { zod4 as zod } from 'sveltekit-superforms/adapters';;
import { createUpdate{Item}Schema } from '$lib/app/{items}/{item}-schema';

export const load: PageServerLoad = async () => {
	return {
		form: await superValidate(zod(createUpdate{Item}Schema))
	};
};

// +page.server.ts (edit)
import { error } from '@sveltejs/kit';

export const load: PageServerLoad = async ({ params }) => {
	const result = await get{Item}Query(params.id);

	if (result.error) {
		return error(result.error.code, result.error.message);
	}

	return {
		form: await superValidate(result.data, zod(createUpdate{Item}Schema))
	};
};

// +page.server.ts (list)
export const load: PageServerLoad = async () => {
	const {items} = await get{Items}Query();
	return { {items} };
};

// +page.server.ts (details)
import { error } from '@sveltejs/kit';

export const load: PageServerLoad = async ({ params }) => {
	const result = await get{Item}Query(params.id);

	if (!result.error) {
		return { {item}: result.data };
	}

	return error(result.error.code, result.error.message);
};
```

### Form Actions

```typescript
// +page.server.ts (create/edit)
import { superValidate } from 'sveltekit-superforms';
import { zod4 as zod } from 'sveltekit-superforms/adapters';
import { fail, redirect } from '@sveltejs/kit';
import { createUpdate{Item}Schema } from '$lib/app/{items}/{item}-schema';
import { create{Item}Command, update{Item}Command } from '$lib/app/{items}';
import { setFormErrorsFromZod } from '$lib/utils';

export const actions: Actions = {
	default: async ({ request, params }) => {
		const form = await superValidate(request, zod(createUpdate{Item}Schema));

		if (!form.valid) {
			return fail(400, { form, error: null });
		}

		const result = params?.id
			? await update{Item}Command(params.id, form.data)
			: await create{Item}Command(form.data);

		if (!result.error) {
			redirect(303, '/{items}');
		}

		setFormErrorsFromZod(form, result.error.validationErrors);

		return fail(result.error.code, { form, error: result.error.message });
	}
};
```

## Public Exports

```typescript
// lib/app/{feature}/index.ts
// API types from schema
export {
	type Get{Items}Result,
	type {Item},
	type CreateUpdate{Item}Request as {Item}DetailsRequest,
	type {Item}Summary
} from './{feature}-schema';

// Action parameter types exported from their respective action files
// Note: Only export parameter types for actions with multiple parameters
// Actions with single id parameter use id: string directly (no interface needed)
export { type Get{Items}Query } from './actions/get-{items}';
export { type Create{Item}Params } from './actions/create-{item}';
export { type Update{Item}Params } from './actions/update-{item}';

// Configuration
export { {item}Config, type {Item}Config } from './config';

// Constants (if applicable)
export { {Item}Statuses, type {Item}Status } from './{item}-constants';

// Commands and Queries
export { create{Item}Command } from './actions/create-{item}';
export { update{Item}Command } from './actions/update-{item}';
export { delete{Item}Command } from './actions/delete-{item}';
export { get{Item}Query } from './actions/get-{item}';
export { get{Items}Query } from './actions/get-{items}';

// Note: Components moved to route-specific locations
// Import from /routes/{feature}/components/ as needed
```

## Component Import Pattern

```typescript
// In route files, import business logic from features
import { type Get{Items}Result, get{Items}Query } from '$lib/app/{items}';

// Import UI components from local route components
import {Item}Card from './components/{item}-card.svelte';
import {Item}Form from './components/{item}-form.svelte';
```

## Form Validation Utilities

When working with forms and validation errors, use the `setFormErrorsFromZod` utility:

```typescript
import { setFormErrorsFromZod } from '$lib/utils';

// In your action handler after a failed command
if (result.error) {
	// Automatically set field-specific errors from validation
	setFormErrorsFromZod(form, result.error.validationErrors);
	
	return fail(result.error.code, { form, error: result.error.message });
}
```

## Additional Patterns

### DELETE Endpoint

```typescript
// routes/{items}/[id]/+server.ts
import { error } from '@sveltejs/kit';
import type { RequestHandler } from './$types';
import { deleteItemCommand } from '$lib/app/{items}';

export const DELETE: RequestHandler = async ({ params }) => {
	const result = await deleteItemCommand(params.id);

	if (!result.error) {
		return new Response(null, { status: 204 });
	}

	return error(result.error.code, result.error.message);
};
```

### Form Component Data Pattern

When passing data to form components, always include both the form data and error state:

```typescript
// In +page.svelte
interface PageProps {
	data: {
		form: SuperValidated<Infer<Schema>>;
		error?: string | null;
	};
}

// Usage
<{Item}Form mode="create" data={data} />
```

This pattern enables the form component to display both field-level validation errors (through SuperForms) and general operation errors (through the error property).

### State Management

- Use Svelte 5 runes: `$state`, `$derived`, `$props`
- Component-level state for UI interactions
- Form state with validation feedback
- Loading states during async operations

### Navigation Patterns

- After create: redirect to details page (e.g., `/cashiers/${result.data.cashierId}`)
- After update: redirect to details page  
- Cancel buttons: use `href` for navigation
- Delete: client-side fetch with optimistic UI update

### Key Conventions

- Use interface for component props
- Import types from `./$types` in routes
- Client and server-side validation
- Progressive enhancement with `use:enhance`
- Error display with touched state tracking

# Tech Stack

## Frontend

- **SvelteKit**: Full-stack web framework with TypeScript
- **Svelte 5**: Latest Svelte with runes (`$state`, `$derived`, `$props`)
- **Tailwind CSS**: Utility-first CSS framework
- **shadcn-svelte**: UI component library
- **Zod v4**: Schema validation and type safety
- **Formsnap**: Form handling with SvelteKit SuperForms
- **Vitest**: Unit testing framework
- **Playwright**: E2E testing framework

## Backend Integration

- **openapi-fetch**: Type-safe API client with OpenAPI schema integration
- **OpenAPI/Swagger**: API schema for automatic type generation
- **OpenTelemetry**: Observability and tracing
- **REST API**: Direct HTTP calls with structured error handling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vgmello) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
