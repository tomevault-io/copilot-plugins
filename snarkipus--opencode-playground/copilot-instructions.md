## opencode-playground

> Generates a Svelte Playground link with the provided code.

# Agent Operational Guide (AGENTS.md)

This document provides context, commands, and coding standards for agents operating in this repository.
This project is a modern web application built with **SvelteKit**, **Svelte 5**, **Tailwind CSS v4**, and **Shadcn-svelte**.

## 1. Environment & Commands

### Build & Verification

Always run verification commands after making changes.

- **Type Check**: `npm run check`
  - Runs `svelte-check` to verify TypeScript types and Svelte component usage.
  - **Critical**: Must pass before finishing any task.
- **Build**: `npm run build`
  - Compiles the application for production using Vite.
  - Ensures no build-time errors exist.
- **Lint**: `npm run lint`
  - Runs Prettier check and ESLint.
- **Format**: `npm run format`
  - Runs Prettier write to fix formatting issues.
- **Dev Server**: `npm run dev`
  - Starts the local development server.

### Testing

- **Unit Tests**: `npm run test`
  - Runs Vitest for unit testing.
  - Use for testing utilities and complex component logic.
- **Type Check**: `npm run check`
  - Runs `svelte-check`.

## 2. Tech Stack & Conventions

### Core Technologies

- **Framework**: SvelteKit (File-based routing in `src/routes`)
- **UI Library**: Svelte 5 (Runes mode)
- **Styling**: Tailwind CSS v4 (CSS-first configuration)
- **Components**: Shadcn-svelte (Headless UI + Tailwind)
- **Language**: TypeScript

### Svelte 5 Guidelines (Runes)

**DO NOT** use legacy Svelte 4 syntax (e.g., `export let prop`, `$: derived`).
Use Svelte 5 Runes for all reactivity.

**Props:**

```svelte
<script lang="ts">
	interface Props {
		title: string;
		count?: number;
		children?: import('svelte').Snippet;
	}
	let { title, count = 0, children }: Props = $props();
</script>
```

**State & Derived:**

```svelte
<script lang="ts">
	let count = $state(0);
	let double = $derived(count * 2);

	function increment() {
		count += 1;
	}
</script>
```

**Effects:**
Use `$effect` sparingly, primarily for synchronization with external systems (DOM, API).

```svelte
$effect(() => {
  console.log(count);
});
```

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

##### Available MCP Tools:

###### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

###### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

###### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

###### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

###### Tailwind CSS v4 Guidelines

This project uses **Tailwind CSS v4**.

- **Configuration**: Managed in `src/app.css` using CSS variables and `@theme` blocks.
- **No `tailwind.config.js`**: Do not look for or create this file.
- **Dynamic Classes**: Use the `cn()` utility from `$lib/utils` to merge classes and resolve conflicts.

**Example:**

```svelte
<script lang="ts">
	import { cn } from '$lib/utils';
	let { class: className } = $props();
</script>

<div class={cn('bg-primary text-primary-foreground p-4', className)}>Content</div>
```

**CSS Variables (Shadcn):**
The theme is defined in `src/app.css`. Use HSL variables (e.g., `--primary`, `--foreground`).
Use the `dark` class variant for dark mode overrides.

### Shadcn-svelte Components

- **Guides & References**: See `docs/vendor/shadcn-svelte/llms.txt` for shadcn-svelte guides, references, and best practices.
- **Location**: `src/lib/components/ui`
- **Installation**: Use the CLI for new components: `npx shadcn-svelte@latest add [component]`
- **Structure**: Components are "owned" code. You can modify them directly if needed, but prefer wrapping or extending.

### TypeScript Rules

- **Strict Mode**: Enabled.
- **Explicit Types**: Avoid `any`. Define interfaces for props and API responses.
- **Imports**: Always use the `$lib` alias for internal imports.
  - Bad: `../../lib/utils`
  - Good: `$lib/utils`

## 3. Project Structure

```
src/
├── app.css          # Global styles, Tailwind setup, Theme variables
├── app.html         # HTML shell
├── lib/
│   ├── components/  # Reusable UI components
│   │   └── ui/      # Shadcn components (button, card, etc.)
│   ├── utils.ts     # Global utilities (cn helper)
│   └── assets/      # Static assets imported in code
└── routes/          # SvelteKit File-based routing
    ├── +layout.svelte # Root layout (imports app.css)
    └── +page.svelte   # Home page
```

## 4. Development Workflow for Agents

1.  **Analysis**:
    - Read `src/routes` to understand the current page structure.
    - Check `src/lib/components` for available UI blocks.

2.  **Implementation**:
    - Create new components in `src/lib/components`.
    - Use `npx shadcn-svelte@latest add` if a standard UI pattern is needed (tabs, dialog, etc.).
    - Update `src/app.css` if new global theme variables are strictly necessary (prefer utility classes).

3.  **Verification**:
    - Run `npm run check` to ensure type safety.
    - Run `npm run build` to confirm the application compiles.

## 5. Common Patterns & Snippets

**Class Merging:**

```typescript
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
	return twMerge(clsx(inputs));
}
```

**Page Layout:**
Pages usually do not need a `<main>` wrapper as `+layout.svelte` handles the root structure.
Focus on the content of the specific route.

**Forms:**
Use standard HTML forms enhanced with Svelte Actions or Form Actions (`+page.server.ts`) if backend logic is added.
For client-side state, bind using `$state`.

```svelte
<script>
	let email = $state('');

	function handleSubmit(e: Event) {
		e.preventDefault();
		// handle submit
	}
</script>

<form onsubmit={handleSubmit}>
	<input bind:value={email} />
</form>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/snarkipus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/snarkipus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
