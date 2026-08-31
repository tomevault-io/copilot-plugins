## pi-conf-manager

> Welcome to **pi-config-manager** — a Pi extension that manages presets, tools, skills, context files, and extensions from a single package. This guide covers conventions and workflows for contributors.

# Repository Guidelines

Welcome to **pi-config-manager** — a Pi extension that manages presets, tools, skills, context files, and extensions from a single package. This guide covers conventions and workflows for contributors.

## Project Structure

```
src/                  # Source code (TypeScript modules)
  index.ts            # Main extension entry point
  types.ts            # Shared type definitions
  storage.ts          # Persistent storage handling
  policy-manager.ts   # Runtime policy management
  presets.ts          # Preset model and helpers
  preset-editor.ts    # Interactive preset editing
  extensions.ts       # Extension registry
  prompt-highlighting.ts  # Skill-block syntax highlighting
tests/                # Test suites (Bun test runner)
skills/               # SKILL.md definitions shipped with the package
docs/agents/          # Agent-specific documentation
scripts/              # Build and CI helper scripts
.github/              # GitHub Actions workflows
```

## Build, Test, and Development Commands

| Command              | Description                                      |
| -------------------- | ------------------------------------------------ |
| `pnpm install`       | Install dependencies                             |
| `pnpm test`          | Run the full test suite via `bun test`           |
| `pnpm run typecheck` | Run `tsc --noEmit` for type checking             |
| `pnpm run check`     | Full check: typecheck + test + package integrity |

> **Note:** This project uses **pnpm** for dependency management and **Bun** as the test runner. Node >= 22.19.0 is required.

## Coding Style & Naming Conventions

- **Language:** TypeScript 5.x, ESNext modules (`"module": "ESNext"`).
- **Strict mode is on** — use explicit types; avoid `any`.
- **Indentation:** 2 spaces.
- **Naming:** Use `camelCase` for functions and variables, `PascalCase` for types, classes, and exported components, and `kebab-case` for file names.
- **Exports:** Prefer named exports over default exports for module entry points.
- **No hard-coded secrets, tokens, or credentials in source.**

## Testing Guidelines

- **Framework:** Bun's built-in test runner (`bun test`).
- Tests live in `tests/` and follow the naming convention `<module>.test.ts`.
- All new or modified logic must include corresponding tests.
- Run tests with `pnpm test` and full checks with `pnpm run check` before opening a PR.

## Commit & Pull Request Guidelines

- **Commits:** Follow [Conventional Commits](https://www.conventionalcommits.org/):
    - `feat:` — new feature
    - `fix:` — bug fix
    - `refactor:` — code restructuring
    - `docs:` — documentation updates
    - `ci:` — CI/CD changes
    - `release:` — version preparation
- **Pull Requests:** Include a clear description, reference related issues, and confirm `pnpm run check` passes. Add screenshots or logs when behavior changes are involved.

## Security & Configuration Tips

- Never commit `.env` files or sensitive configuration; `.pi/` and `node_modules/` are already in `.gitignore`.
- Peer dependencies (`@earendil-works/pi-coding-agent`, `@earendil-works/pi-tui`) must match the installed Pi runtime version.

## Agent skills

### Issue tracker

Issues live as local markdown files under `.scratch/<feature>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Default five-role vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout: `CONTEXT.md` at repo root + `docs/adr/`. See `docs/agents/domain.md`.

---
> Source: [0717wuwang/pi-conf-manager](https://github.com/0717wuwang/pi-conf-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
