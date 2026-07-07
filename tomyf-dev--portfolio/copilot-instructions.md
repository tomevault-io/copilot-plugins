## portfolio

> This file defines the **rules and guidelines** for AI coding agents working on this project.

# AGENTS.md – AI Coding Agent Instructions

This file defines the **rules and guidelines** for AI coding agents working on this project.
All AI agents must strictly follow these instructions.

---

# 1. Project Overview

Portfolio website for ToMY F.

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS 4
- **Output**: SSG (Static Site Generation) - `next build` outputs static files to `out/`
- **Deployment**: Static hosting (e.g., Cloudflare Pages)

---

# 2. Build & Test

```bash
# Setup (using pnpm)
pnpm install

# Development
pnpm dev                    # Start dev server at http://localhost:3000

# Build (SSG)
pnpm build                  # Generate static files to out/

# Preview (after build)
pnpm start                  # Run npx serve out

# Lint
pnpm lint                   # ESLint
```

**Note**: No test framework is currently configured.

---

# 3. Project Structure

```
/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root layout
│   ├── page.tsx            # Home page
│   ├── globals.css         # Global styles (Tailwind)
│   └── favicon.ico         # Favicon
├── public/                 # Static assets
├── next.config.ts          # Next.js config (SSG output settings)
├── tsconfig.json           # TypeScript config
├── eslint.config.mjs       # ESLint config
├── postcss.config.mjs      # PostCSS config (Tailwind)
└── package.json            # Dependencies and scripts
```

**Path alias**: `@/*` → Relative path from project root

---

# 4. Code Style

## General

- Use TypeScript (strict mode)
- Use functional components
- Use default exports (following Next.js App Router conventions)

## Naming Conventions

- **Components**: PascalCase (`ProfileCard.tsx`)
- **Utilities**: camelCase (`formatDate.ts`)
- **Constants**: SCREAMING_SNAKE_CASE (`MAX_ITEMS`)
- **Types/Interfaces**: PascalCase without prefix (`Props`, `User`)

## Styling

- Use Tailwind CSS utility classes
- Keep custom CSS minimal
- Support dark mode using `dark:` prefix

## Files

- One component per file
- Page components go under `app/`
- Shared components go under `components/` directory

---

# 5. Branch Rules (GitHub Flow)

## 5.1 Primary Branch

### `main`

* The **default and only** primary branch.
* All feature work MUST start from `main`.
* All pull requests MUST target `main`.

> AI Agents:
>
> * NEVER commit directly to `main`.
> * ALWAYS use a work branch and open a pull request into `main`.

---

## 5.2 Development Flow

1. Create a work branch from `main`.
2. Make commits on the work branch.
3. Open a pull request into `main`.

---

## 5.3 Branch Naming Rules

Use **lowercase letters and hyphens** only.

### Feature Branches

All work branches use the `feature/` prefix.

```
feature/<short-summary>
```

Examples:

* `feature/oauth-login`
* `feature/fix-auth-redirect`
* `feature/update-readme`

### AI-Generated Branches

AI tools generate branches with tool-specific prefixes.
These are permitted and follow the same workflow as `feature/*` branches.

| Tool | Prefix | Example |
|------|--------|---------|
| GitHub Copilot | `copilot/` | `copilot/add-theme-switcher` |
| OpenAI Codex | `codex/` | `codex/fix-login-bug` |
| Cursor | `cursor/` | `cursor/refactor-api` |
| Claude Code | `claude/` | `claude/add-tests` |
| Google Jules | `jules/` | `jules/update-deps` |

Other `<tool-name>/*` patterns are also permitted.

Rules for AI-generated branches:

* They MUST be created from `main`.
* They MUST be merged into `main` via pull request.

---

## 5.4 Optional: Issue Numbers

If an issue tracker is used, branches may include IDs:

```
feature/123-oauth-login
copilot/456-fix-redirect
```

---

# 6. Commit Message Rules (Conventional Commits)

This project uses **Conventional Commits**.
All commit messages MUST follow the format below.

---

## 6.1 Commit Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Required:

* `<type>`
* `<description>` (short, clear, lowercase, English)

### Optional:

* `scope`
* body text
* footer (breaking changes, issue references, etc.)

---

## 6.2 Allowed Types

* `feat` — New feature
* `fix` — Bug fix
* `docs` — Documentation only
* `style` — Formatting (no code behavior impact)
* `refactor` — Code restructuring
* `perf` — Performance improvements
* `test` — Tests only
* `build` — Build system or dependency changes
* `ci` — CI pipeline changes
* `chore` — Miscellaneous changes

---

## 6.3 Scopes

Use scopes for clarity when appropriate:

Examples:

* `feat(auth): add passwordless login`
* `fix(api): handle missing spaceId`

Do NOT overuse scope. Only add it when meaningful.

---

## 6.4 Examples

```
feat(auth): add passwordless login endpoint

fix(api): handle missing membership id in request

docs(readme): update branch strategy explanation

refactor(ui): simplify space selector component

chore(deps): update nextjs to 15.0.1
```

---

## 6.5 Prohibited Examples

```
update
fix bug
change something
wip
temp
test commit
```

Ambiguous commit messages are strictly prohibited.

---

# 7. Pull Request Rules

## 7.1 PR Title Format

Use Conventional Commits format for PR titles:

```
<type>[optional scope]: <description>
```

Examples:

* `feat(auth): add OAuth login support`
* `fix(api): resolve null pointer exception`
* `docs: update API documentation`

---

## 7.2 PR Description

Every PR MUST include:

* **Summary**: Brief description of changes
* **Changes**: List of specific modifications
* **Testing**: How the changes were tested
* **Related Issues**: Link to related issues (if applicable)

---

## 7.3 PR Requirements

* Keep PRs focused on a single concern
* Avoid mixing refactors with feature changes
* Ensure all tests pass before requesting review
* Include screenshots for UI changes

---

# 8. Permissions and Boundaries

## 8.1 Allowed Without Confirmation

AI agents MAY perform these actions without asking:

* Read files and list directories
* Search codebase for patterns and references
* Run lint, format, and typecheck on modified files
* Run unit tests related to changes
* Create new files in appropriate locations
* Write and update documentation

---

## 8.2 Requires Confirmation

AI agents MUST ask before performing these actions:

* Install new packages or dependencies
* Delete or rename existing files
* Run full build or E2E test suites
* Modify configuration files
* Make changes outside the current task scope
* Execute commands with side effects (e.g., database operations)

---

## 8.3 Prohibited Without Explicit Approval

AI agents MUST NOT perform the following actions without explicit human approval:

### Security

* Commit secrets, credentials, API keys, or tokens
* Modify authentication or authorization logic
* Change security-related configurations
* Expose sensitive data in logs or error messages

### Infrastructure & Configuration

* Modify CI/CD pipelines or workflows
* Change deployment configurations
* Alter database schemas or migrations
* Update environment variable definitions

### Dependencies

* Add new production dependencies
* Upgrade major versions of existing dependencies
* Remove existing dependencies
* Add dependencies with unknown or restrictive licenses

### Destructive Operations

* Delete or overwrite existing files without confirmation
* Remove or rename public APIs
* Drop database tables or columns
* Clear caches or storage in production

### Architecture

* Introduce new architectural patterns
* Create new top-level directories
* Change project structure significantly
* Add new external service integrations

---

# 9. AI Agent Rules

AI agents MUST follow these enforced rules:

1. **NEVER commit directly to `main`.**
   Always use a work branch and open a pull request into `main`.

2. **Use `feature/` prefix** for manually created branches, or use the default AI tool prefix.

3. **Respect all boundaries** defined in Section 8. Ask for approval when uncertain.

4. **Run verification before committing:**
   * Lint checks
   * Type checks
   * Unit tests (if applicable)

5. **Keep commits meaningful:**
   * Group related changes together
   * Avoid excessive tiny commits
   * Write clear commit messages

6. **When proposing large changes:**
   * Explain reasoning in the PR body
   * Summarize impact
   * Break into smaller PRs if possible

---

# 10. Additional Notes

> **TODO**: Add any project-specific notes.

**Example:**

```
## External Services
- Authentication: Auth0
- Database: PostgreSQL via Supabase
- File Storage: AWS S3

## Environment Variables
See .env.example for required variables.

## Documentation
- API docs: /docs/api.md
- Architecture: /docs/architecture.md
```

---

This AGENTS.md will expand as the project evolves.

---
> Source: [tomyf-dev/portfolio](https://github.com/tomyf-dev/portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
