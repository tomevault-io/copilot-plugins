## la-cueva-de-tatoh

> <!-- # La Cueva de Tatoh - Project Context

<!-- # La Cueva de Tatoh - Project Context

## Project Overview

"La Cueva de Tatoh" is an Angular 21 monorepo workspace managed with `pnpm`. It consolidates multiple applications and a shared library into a single repository.

### Key Projects

The workspace (configured in `angular.json`) contains the following projects located in the `projects/` directory:

1. **`perfil-personal`** (Application): A personal portfolio/blog application.
   - **Features**: Blog posts (Markdown rendering via `ngx-markdown` & `prismjs`), QR generator, Service Worker (PWA).
   - **Tech**: Angular, SCSS.

2. **`comidas`** (Application): A meal planning or food tracking application.
   - **Features**: Service Worker (PWA).
   - **Tech**: Angular, SCSS.

3. **`componentes`** (Library): A shared UI component library used by the applications.
   - **Purpose**: Encapsulates reusable UI elements to ensure consistency across apps.

## Building and Running

### Prerequisites

- **Node.js**: Compatible with Angular 21.
- **Package Manager**: `pnpm` (enforced via `preinstall` script).

### Key Commands

| Command        | Description                                                               |
| :------------- | :------------------------------------------------------------------------ |
| `pnpm install` | Install dependencies.                                                     |
| `pnpm start`   | Runs `ng serve`. (Default serving command).                               |
| `pnpm build`   | Builds **all** projects: `componentes`, `perfil-personal`, and `comidas`. |
| `pnpm test`    | Runs unit tests (`ng test`).                                              |
| `pnpm lint`    | Runs linting (`ng lint`) for all projects.                                |
| `pnpm watch`   | Runs build in watch mode for development.                                 |

### Targeted Commands

To target specific projects, use the Angular CLI directly:

- **Serve specific app**: `ng serve <project-name>` (e.g., `ng serve comidas`)
- **Build specific app**: `ng build <project-name>`

## Development Conventions

- **Architecture**: Follows standard Angular workspace patterns. Code is modularized into `projects/`.
- **Styling**: SCSS is the standard for component styling.
- **Linting & Formatting**:
  - Uses **ESLint** (flat config) and **Prettier**.
  - Strict adherence to linting rules is expected.
  - Run `pnpm lint` to verify code quality.
- **Shared Code**: Any UI component or logic reusable between `perfil-personal` and `comidas` must be implemented in the `componentes` library.
- **Assets**: Public assets are located in the `public/` directory of each specific project.

## Technology Stack

- **Framework**: Angular 21
- **Language**: TypeScript 5.9
- **Build Tool**: Angular CLI (esbuild-based)
- **Testing**: Karma/Jasmine (primary), Vitest (installed)
- **Context7**: use context7 to check documentation. -->

# La Cueva de Tatoh – Gemini Agent Context

## Project Overview

"La Cueva de Tatoh" is an Angular 21 monorepo workspace managed with `pnpm`.

The repository contains multiple applications and a shared UI library, all located under `projects/`.

### Projects

1. **perfil-personal** (Application)
   - Personal portfolio / blog
   - Markdown rendering with `ngx-markdown` and `prismjs`
   - PWA enabled
   - Angular + SCSS

2. **comidas** (Application)
   - Meal planning / food tracking
   - PWA enabled
   - Angular + SCSS

3. **componentes** (Library)
   - Shared UI components
   - Any reusable UI or logic MUST live here

---

## Tooling & Environment

- Framework: **Angular 21**
- Language: **TypeScript 5.9**
- Package manager: **pnpm** (mandatory)
- Build system: Angular CLI (esbuild)
- Linting: **ESLint (flat config)**
- Formatting: **Prettier**
- Testing: Karma/Jasmine (primary), Vitest (installed)
- Documentation checks: **Context7**

---

## 🚨 ESLint Is Authoritative (MANDATORY)

**ESLint is the single source of truth.**

Any code produced:

- MUST comply with the ESLint flat configuration
- MUST pass `pnpm lint` without modifications
- MUST be rewritten if any rule is violated

Lint errors are **bugs**, not suggestions.

If there is a conflict between:

- best practices
- framework conventions
- personal preference

➡ **ESLint always wins.**

---

## TypeScript Rules (Strict)

- All functions and methods MUST have explicit return types
- All class members MUST declare accessibility
  - `public` is forbidden (use `private`, `protected`, or implicit for constructors only)
- Prefer `type` over `interface`
- Avoid `any` whenever possible
- Prefer `const` over `let`
- No unused variables
- No empty interfaces
- No shadowed variables
- One class per file only
- Indexed object styles are flexible unless specified

---

## JavaScript Rules (Strict)

- Always use `===` and `!==`
- Curly braces are mandatory
- Never use `var`
- Prefer arrow callbacks
- Prefer `const`
- Imports MUST be sorted:
  - Case-insensitive
  - Declaration order is preserved
  - Separated groups are allowed
- Maximum cyclomatic complexity: 20
- One variable declaration per statement
- No bitwise operators
- No `eval` or implied eval

---

## Angular Rules (Strict)

- Standalone components are preferred
- Prefer signals over RxJS when possible
- Prefer readonly outputs
- Avoid empty lifecycle hooks
- Change detection strategy is flexible unless specified

### Selectors (MANDATORY)

#### Components

- prefix: `app`
- style: `kebab-case`
- type: `element` or `attribute`

#### Directives

- prefix: `app`
- style: `camelCase`
- type: `attribute`

Violating selector rules is a **hard error**.

---

## Angular Templates (HTML – Strict)

- Use strict equality only
- Prefer new control flow syntax:
  - `@if`
  - `@for`
- Attribute order MUST follow ESLint configuration:
  1. Structural directives
  2. Template references
  3. Attribute bindings
  4. Input bindings
  5. Two-way bindings
  6. Output bindings
- Prefer self-closing tags when possible
- Use `trackBy` functions in loops
- Avoid unnecessary complexity in templates

---

## Shared Code Rules

- Any logic or UI reusable across apps MUST be implemented in `componentes`
- Do not duplicate shared components
- Do not place app-specific logic in the shared library

---

## Mandatory Validation Workflow

Before presenting any code:

1. Generate the solution
2. Validate it mentally against ESLint rules
3. Rewrite until no rule is violated
4. Assume `pnpm lint` will run immediately

If the code would fail linting, it must be fixed before answering.

---

## Output Expectations

- Do NOT invent conventions
- Do NOT guess APIs
- Do NOT relax rules for convenience
- Prefer explicit, strict, and verbose code over clever shortcuts

Correct, lint-compliant code is more important than brevity.

---

## Documentation

- Use **Context7** to validate Angular and TypeScript APIs when needed
- If documentation is unclear, prefer the most restrictive ESLint-compliant approach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/T4toh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/T4toh)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
