## skiftesgatan-se

> | Command       | Description                      |

## Commands

| Command       | Description                      |
| ------------- | -------------------------------- |
| `pnpm format` | Auto-format code                 |
| `pnpm lint`   | Lint code                        |
| `pnpm knip`   | Detect unused files/dependencies |
| `pnpm check`  | SvelteKit sync + svelte-check    |
| `pnpm test`   | Run all tests (unit + E2E)       |

## Available MCP Tools:

### Svelte

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

#### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

#### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

#### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### Better Auth

Documentation server for Better Auth. Use when working on authentication, sessions, or the auth configuration (`src/lib/server/auth.ts`, `src/lib/server/auth.config.ts`).

#### 1. search_docs

Search for relevant Better Auth documentation pages. Call this first to find pages related to your task.

#### 2. get_doc

Retrieve full content of a specific documentation page. Use after search_docs to read the details you need.

### Resend

Documentation server for the Resend email API. Use when working on email delivery (`src/lib/server/email.ts`).

#### 1. search_resend

Search Resend documentation for topics like sending emails, templates, domains, and API usage.

### Knip

Runs Knip to detect unused files, dependencies, and exports. Use when cleaning up code or checking for dead code.

#### 1. knip-run

Run Knip analysis on the project. Returns unused files, dependencies, and exports.

#### 2. knip-docs

Search Knip documentation for configuration and usage guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ollema) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
