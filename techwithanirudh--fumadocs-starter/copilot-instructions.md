## fumadocs-starter

> - Next.js App Router site with Fumadocs content and MDX.

# Agent Guide

## Project Overview
- Next.js App Router site with Fumadocs content and MDX.
- TypeScript in strict mode with path aliases (`@/*`, `@/public/*`).
- Tailwind CSS (v4) with PostCSS, plus Radix UI and shadcn/ui components.
- Content lives in `content/` and Fumadocs-generated `.source/`.
- Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask.

## Repo Layout
- `src/app`: App Router routes, layouts, and metadata.
- `src/components`: UI and layout components.
- `content/` and `.source/`: MDX collections and generated content.
- `emails/`, `scripts/`, `public/`: email templates, tooling, and static assets.

## Commands
- Install: `bun install`
- Dev/build: `bun dev`, `bun run build`
- Types: `bun run typegen`, `bun run typecheck`
- Lint: `bun run check` (fix: `check:write`, unsafe: `check:unsafe`)
- QA: `bun run check:links`, `bun run check:spelling`

## Testing
- No automated test runner is configured (no `test` script or test config files found).
- Single-test command: not available until a test runner is added.

## Philosophy

This codebase will outlive you. Every shortcut becomes someone else's burden. Every hack compounds into technical debt that slows the whole team down.

You are not just writing code. You are shaping the future of this project. The patterns you establish will be copied. The corners you cut will be cut again.

Fight entropy. Leave the codebase better than you found it.

## Environment & Config
- Environment variables are validated in `src/env.ts` using `@t3-oss/env-nextjs` and `zod`.
- `next.config.ts` loads `src/env` at startup; keep env validation passing locally.
- Set `SKIP_ENV_VALIDATION=true` only for CI/build environments that cannot supply secrets.
- Server env includes database, Resend, auth providers, BotID, and GitHub/Google OAuth tokens.
- Client env uses `NEXT_PUBLIC_*` for analytics and base URL settings.

## Content & Docs
- Author content in `content/`; generated content lives in `.source/` (never edit by hand).
- `bun install` runs `fumadocs-mdx` via `postinstall` to sync MDX types.
- Prefer MDX frontmatter for metadata (title, description, dates).

## UI & Styling
- Tailwind CSS v4 with PostCSS; global tokens live in `src/styles/globals.css`.
- Prefer `cn` from `src/lib/utils.ts` to merge class names.
- shadcn/ui components live in `src/components/ui`; extend them rather than copy/paste.
- Radix UI primitives are used for accessible overlays, menus, and form controls.

## Coding Standards
Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

- **Format code**: `bun x ultracite fix`
- **Check for issues**: `bun x ultracite check`
- **Diagnose setup**: `bun x ultracite doctor`

Biome (the underlying engine) provides robust linting and formatting. Most issues are automatically fixable.

### Type Safety & Explicitness
- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

### Modern JavaScript/TypeScript
- Use arrow functions for callbacks and short functions
- Prefer `for...of` loops over `.forEach()` and indexed `for` loops
- Use optional chaining (`?.`) and nullish coalescing (`??`) for safer property access
- Prefer template literals over string concatenation
- Use destructuring for object and array assignments
- Use `const` by default, `let` only when reassignment is needed, never `var`

### Async & Promises
- Always `await` promises in async functions - don't forget to use the return value
- Use `async/await` syntax instead of promise chains for better readability
- Handle errors appropriately in async code with try-catch blocks
- Don't use async functions as Promise executors

### Imports & Modules
- Prefer path aliases (`@/*`, `@/public/*`) over deep relative imports.
- `fumadocs-mdx:collections/*` resolves to `.source/*` for generated content.
- Avoid barrel files; import from the source module directly.
- Keep import groups ordered: built-in, external, internal.

### Files & Naming
- Use PascalCase for React components and camelCase for functions/variables.
- Favor kebab-case for file and folder names in `src/`.
- Keep route segments aligned with App Router conventions in `src/app`.

### React & JSX
- Use function components over class components
- Call hooks at the top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use the `key` prop for elements in iterables (prefer unique IDs over array indices)
- Nest children between opening and closing tags instead of passing as props
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility:
  - Provide meaningful alt text for images
  - Use proper heading hierarchy
  - Add labels for form inputs
  - Include keyboard event handlers alongside mouse events
  - Use semantic elements (`<button>`, `<nav>`, etc.) instead of divs with roles

### Error Handling & Debugging
- Remove `console.log`, `debugger`, and `alert` statements from production code
- Throw `Error` objects with descriptive messages, not strings or other values
- Use `try-catch` blocks meaningfully - don't catch errors just to rethrow them
- Prefer early returns over nested conditionals for error cases

### Code Organization
- Keep functions focused and under reasonable cognitive complexity limits
- Extract complex conditions into well-named boolean variables
- Use early returns to reduce nesting
- Prefer simple conditionals over nested ternary operators
- Group related code together and separate concerns

### Security
- Add `rel="noopener"` when using `target="_blank"` on links
- Avoid `dangerouslySetInnerHTML` unless absolutely necessary
- Don't use `eval()` or assign directly to `document.cookie`
- Validate and sanitize user input

### Performance
- Avoid spread syntax in accumulators within loops
- Use top-level regex literals instead of creating them in loops
- Prefer specific imports over namespace imports
- Avoid barrel files (index files that re-export everything)
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

### Framework-Specific Guidance
**Next.js:**
- Use Next.js `<Image>` component for images
- Use `next/head` or App Router metadata API for head elements
- Use Server Components for async data fetching instead of async Client Components
**React 19+:**
- Use ref as a prop instead of `React.forwardRef`
**Solid/Svelte/Vue/Qwik:**
- Use `class` and `for` attributes (not `className` or `htmlFor`)

### Testing Conventions
- Write assertions inside `it()` or `test()` blocks
- Avoid done callbacks in async tests - use async/await instead
- Don't use `.only` or `.skip` in committed code
- Keep test suites reasonably flat - avoid excessive `describe` nesting

### When Biome Can't Help
Biome's linter will catch most issues automatically. Focus your attention on:
1. **Business logic correctness** - Biome can't validate your algorithms
2. **Meaningful naming** - Use descriptive names for functions, variables, and types
3. **Architecture decisions** - Component structure, data flow, and API design
4. **Edge cases** - Handle boundary conditions and error states
5. **User experience** - Accessibility, performance, and usability considerations
6. **Documentation** - Add comments for complex logic, but prefer self-documenting code

Most formatting and common issues are automatically fixed by Biome. Run `bun x ultracite fix` before committing to ensure compliance.

---
> Source: [techwithanirudh/fumadocs-starter](https://github.com/techwithanirudh/fumadocs-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
