## template

> You are an AI coding assistant working in a T3 Turbo monorepo. Follow these conventions strictly.

# AI Agent Context for T3 Turbo Monorepo

You are an AI coding assistant working in a T3 Turbo monorepo. Follow these conventions strictly.

## Project Structure

This is a Turborepo monorepo using:
- **Package Manager**: pnpm (workspace protocol)
- **Apps**: Next.js 15, Expo (React Native)
- **Packages**: Shared packages under `packages/`
- **Tooling**: Shared configs under `tooling/`

### Monorepo Layout
```
apps/
  ├─ nextjs/          # Next.js 15 app (web)
  └─ expo/            # Expo app (React Native)
packages/
  ├─ api/             # tRPC router (server-only)
  ├─ auth/            # Better Auth configuration
  ├─ db/              # Drizzle ORM schema & client
  ├─ store/           # Zustand state management
  ├─ ui/              # shadcn/ui components
  └─ validators/      # Zod schemas
tooling/
  ├─ eslint/          # Shared ESLint configs
  ├─ prettier/        # Shared Prettier config
  ├─ tailwind/        # Shared Tailwind configs
  └─ typescript/      # Shared tsconfig
```

## Code Style Rules

### TypeScript
- Use strict mode (already configured)
- Prefer type imports: `import type { X } from '...'`
- Use `satisfies` for type-safe object literals
- No `any` types - use `unknown` if needed
- Use `zod/v4` for validation: `import { z } from 'zod/v4'`

### Import Order (enforced by Prettier)
1. Type-only imports
2. React/Next.js/Expo
3. Third-party packages
4. `@acme/*` packages (types first, then values)
5. Local imports (`~/`, `../`, `./`)

### Naming Conventions
- **Files**: kebab-case for files, PascalCase for React components
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Types/Interfaces**: PascalCase
- **Functions**: camelCase, descriptive verbs

### React/Next.js Patterns
- Use Server Components by default
- Client Components only when needed (`'use client'`)
- Use `~/` alias for app-level imports
- Prefer `async` Server Components
- Use tRPC for API calls (type-safe)

### Error Handling
- Use Zod schemas for validation
- Throw errors with descriptive messages
- Use tRPC error codes appropriately
- Never expose sensitive data in errors

## Security Rules

### Environment Variables
- **NEVER** use `process.env` directly
- Always use validated env: `import { env } from '~/env'` or `@acme/env`
- Never commit `.env` files
- Never log secrets or API keys
- Use `.env.example` as template

### Secrets Handling
- No secrets in code
- No secrets in logs
- No PII in logs
- Use environment variables for all secrets

## File Organization

### Package Structure
Each package should have:
- `package.json` with proper workspace deps
- `tsconfig.json` extending `@acme/typescript-config/base`
- `eslint.config.js` using shared config
- `src/index.ts` as main entry point

### App Structure (Next.js)
- `src/app/` - App Router pages
- `src/components/` - React components
- `src/trpc/` - tRPC client/server setup
- `src/auth/` - Auth client/server utilities

### App Structure (Expo)
- `src/app/` - Expo Router pages
- `src/components/` - React Native components
- `src/utils/` - Utilities (api, auth, etc.)

## Dependencies

### Package Dependencies
- `@acme/api` - **dev dependency** in client apps (for types only)
- `@acme/api` - **production dependency** only in Next.js server
- Shared runtime code → create `@acme/shared` package

### Import Rules
- Use workspace protocol: `"@acme/ui": "workspace:*"`
- Prefer shared packages over duplication
- Keep dependencies minimal

## Testing & Quality

### Linting
- ESLint runs on all files
- TypeScript strict mode enabled
- Import order enforced
- No unused variables (prefix with `_` to ignore)

### Formatting
- Prettier with Tailwind plugin
- Auto-sort imports
- Run `pnpm format:fix` before committing

### Type Checking
- `pnpm typecheck` must pass
- No `@ts-ignore` or `@ts-expect-error` without comment

## Commit Messages

**MANDATORY**: All commit messages MUST follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. The agent MUST enforce this format for every commit.

### Commit Message Format

**Required Format**: `<type>(<scope>): <description>`

Where:
- `<type>`: One of the commit types below (REQUIRED)
- `<scope>`: Optional package/app name (e.g., `api`, `ui`, `nextjs`, `expo`, `cursor`)
- `<description>`: Clear, imperative-mood description (REQUIRED)

### Supported Commit Types

The following commit types are supported per the Conventional Commits specification:

- **`feat`**: A new feature (triggers minor version bump)
- **`fix`**: A bug fix (triggers patch version bump)
- **`docs`**: Documentation only changes
- **`style`**: Code style changes (formatting, missing semicolons, etc.) that do not affect code meaning
- **`refactor`**: Code restructuring that neither fixes a bug nor adds a feature
- **`perf`**: Performance improvements
- **`test`**: Adding or updating tests
- **`build`**: Changes to build system or external dependencies
- **`ci`**: Changes to CI configuration files and scripts
- **`chore`**: Other changes that don't modify src or test files (maintenance tasks)
- **`revert`**: Reverts a previous commit

### Enforcement Rules

1. **ALWAYS** format commit messages using the Conventional Commits format
2. **ALWAYS** use one of the supported commit types
3. **ALWAYS** use imperative mood in the description (e.g., "add" not "added" or "adds")
4. **ALWAYS** keep the description concise but descriptive
5. **NEVER** use periods at the end of the description
6. **NEVER** commit without a properly formatted message

### Examples

✅ **Valid Commit Messages:**
- `feat(api): add user profile endpoint`
- `fix(ui): resolve button hover state`
- `docs: update README with setup instructions`
- `chore(cursor): enforce Conventional Commit rules for Cursor Agent`
- `refactor(store): simplify counter store implementation`
- `test(api): add unit tests for user router`
- `perf(nextjs): optimize image loading`
- `build: update dependencies to latest versions`
- `ci: add GitHub Actions workflow for linting`
- `style: format code with Prettier`
- `revert: revert "feat(api): add user profile endpoint"`

❌ **Invalid Commit Messages:**
- `Added login button` (missing type)
- `fix: Fixed the bug` (past tense, period at end)
- `update readme` (missing type, lowercase)
- `feat: add feature` (too vague)
- `chore: updates` (not imperative mood)

### Agent Behavior

When committing changes:
1. **Analyze** the changes to determine the appropriate commit type
2. **Identify** the scope (package/app) if applicable
3. **Generate** a commit message following the format: `<type>(<scope>): <description>`
4. **Validate** the message before committing
5. **Suggest** corrections if the user provides an invalid format

## Observability

### Logging
- Use structured logging when available
- Log at appropriate levels (error, warn, info, debug)
- Never log secrets or PII
- Include context in error logs

### Performance
- Use React Server Components for performance
- Lazy load heavy components
- Optimize images (Next.js Image)
- Monitor bundle size

## Documentation

### Code Comments
- Document complex logic
- Explain "why" not "what"
- Use JSDoc for public APIs
- Keep comments up-to-date

### README Files
- Each package should have a README if non-trivial
- Document usage, examples, and architecture
- Keep examples current

## Common Tasks

### Adding a New Package
```bash
pnpm turbo gen init
```
Follows monorepo conventions automatically.

### Adding a UI Component
```bash
pnpm ui-add
```
Uses shadcn/ui CLI.

### Adding a tRPC Endpoint
1. Create router in `packages/api/src/router/`
2. Export from `packages/api/src/root.ts`
3. Use Zod schemas for validation
4. Use `protectedProcedure` for auth-required routes

### Adding a Database Schema
1. Add to `packages/db/src/schema.ts`
2. Run `pnpm db:push` to sync
3. Export types from `packages/db/src/index.ts`

## Forbidden Patterns

❌ **DO NOT:**
- Use `process.env` directly (use validated env)
- Import from `@acme/api` in client apps (use dev dependency)
- Commit `.env` files
- Use `any` type
- Skip type checking
- Ignore linting errors without fixing
- Create circular dependencies
- Hardcode secrets or API keys
- Use `@ts-ignore` without explanation

✅ **DO:**
- Use type imports: `import type { X }`
- Use `satisfies` for type safety
- Validate inputs with Zod
- Follow import order
- Write descriptive commit messages
- Keep dependencies minimal
- Document complex logic
- Test your changes

## Quick Reference

- **Lint**: `pnpm lint` / `pnpm lint:fix`
- **Format**: `pnpm format` / `pnpm format:fix`
- **Typecheck**: `pnpm typecheck`
- **Dev**: `pnpm dev`
- **Build**: `pnpm build`

See `docs/AI_CONTEXT.md` for detailed documentation.
docs/DEVELOPER_GUIDE.md for developer-facing docs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vukovicpavle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
