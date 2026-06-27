## buka-samples

> > **Language policy**: All customization files are written in English.

# Project Guidelines

> **Language policy**: All customization files are written in English.
> Always respond in the language the user is using in the chat conversation.
> If the user writes in Japanese, respond in Japanese. If in English, respond in English.

## Overview

This workspace is a **Vibe Coding experience environment** where participants build a Todo app
that extracts events from Gmail and registers them in Google Calendar.
The app has not been created yet — participants will build it from scratch with AI agent assistance.

## Code Style

- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js (ES Modules)
- **Formatter**: Prettier (default config)
- **Linter**: ESLint with `eslint-plugin-security` for vulnerability detection
- Always use `const` / `let`, never `var`
- Prefer `async`/`await` over raw Promises
- Use explicit return types on exported functions

## Architecture

- **Gmail integration**: `googleapis` npm package (Gmail API v1)
- **Google Calendar integration**: `googleapis` npm package (Calendar API v3)
- **Authentication**: OAuth 2.0 via Google Cloud Console credentials
- **Credential storage**: `~/.todo-app/credentials.json` (outside the workspace — never sent to LLM)
- Keep API interaction logic in a dedicated `src/services/` directory
- Keep business logic (event extraction, parsing) in `src/domain/`

## Build and Test

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript
npm test             # Run tests (Vitest)
npm run lint         # ESLint (includes security rules)
npm run lint:fix     # ESLint with auto-fix
npm run format       # Prettier
```

## Development Workflow

### TDD-First

All development MUST follow TDD (Test-Driven Development):

1. Write a failing test first (Red)
2. Write the minimum code to pass (Green)
3. Refactor while keeping tests green (Refactor)

See the `/dev-tdd` prompt and [tdd-workflow instructions](.github/instructions/tdd-workflow.instructions.md) for details.

### Lint on Every Change

After every code edit, run the linter:

```bash
npx eslint --fix <changed-file>
```

This is a non-negotiable practice. Every file you create or modify must pass lint before moving on.

### Pre-PR Checklist (Vulnerability Gate)

Before pushing a PR, complete ALL of the following:

1. **All tests pass**: `npm test`
2. **Lint clean**: `npm run lint` (zero errors)
3. **Dependency vulnerability scan**: `npm audit`
   - If `npm audit` reports vulnerabilities at level `moderate` or higher, they MUST be resolved before the PR.
   - Use `npm audit fix` to auto-fix where possible.
   - If manual resolution is needed, document the issue and the plan.
4. **Security lint review**: Confirm no `eslint-plugin-security` warnings remain.
5. **No secrets exposed**: Verify no API keys, tokens, or credentials appear in code, logs, or anywhere inside the workspace.

## Conventions

- Branch naming: `feature/<short-description>`, `fix/<short-description>`
- Commit messages: Conventional Commits (`feat:`, `fix:`, `test:`, `chore:`)
- One logical change per commit
- Use the `/plan-checklist` prompt before starting any task to create a trackable checklist

## Security

### Secret Protection

Credentials are stored **outside the workspace** in `~/.todo-app/` to prevent them from being sent to the LLM via editor context.

- **NEVER** display, print, or include the contents of `credentials.json` or `token.json` in chat responses, logs, or code output.
- **NEVER** store secrets inside the workspace. No `.env` files with real values, no hardcoded keys.
- When the user needs to configure credentials, guide them through the `/setup` skill which handles placement in `~/.todo-app/`.
- If a secret value accidentally appears in context, immediately warn the user and advise them to rotate the credential.
- See [security instructions](.github/instructions/security.instructions.md) for full policy.

### Dependency Security

- Use `npm audit` to check for known vulnerabilities in dependencies.
- Use `eslint-plugin-security` to detect common security anti-patterns in code (e.g., `eval()`, non-literal `require()`, unsafe regex).
- Before any PR, the vulnerability gate described above must pass.

---
> Source: [TsuyoshiUshio/buka-samples](https://github.com/TsuyoshiUshio/buka-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
