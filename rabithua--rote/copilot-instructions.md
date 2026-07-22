## rote

> - Always use Bun for JavaScript/TypeScript workflows in this repository.

# Agent Instructions

## Package Manager Policy

- Always use Bun for JavaScript/TypeScript workflows in this repository.
- Do not use npm, pnpm, or yarn unless the user explicitly asks for it.

## Command Conventions

- Install dependencies: `bun install`
- Run scripts: `bun run <script>`
- Run package binaries: `bun x <command>`
- Add dependencies: `bun add <pkg>`
- Add dev dependencies: `bun add -d <pkg>`

## Project Paths

- Frontend commands should run in `web/`.
- Backend commands should run in `server/`.

## Pull Request Policy

- Always create work branches from `develop`, not `main`.
- Always open pull requests against `develop`, not `main`.
- Before pushing or opening a PR, verify the branch base and PR base are `develop`.
- When correcting an existing PR with the wrong base, change the PR base to `develop` before
  force-pushing or otherwise synchronizing the branch, so main-only source guards are not triggered.

## Validation Policy

- After every code change, run both lint and build before finishing.
- If frontend is changed, run in `web/`:
  - `bun run lint`
  - `bun run build`
- If backend is changed, run in `server/`:
  - `bun run lint`
  - `bun run build`

## Commit Quality Policy

- Before committing, Codex must let the project Codex hook run and must not bypass a failed quality gate.
- Keep module boundaries explicit. New code should belong to a clear feature, domain, service, route, state, or UI component boundary.
- Avoid large single-file implementations. Split files by responsibility before they become hard to scan, test, or review.
- Do not place unrelated behavior in generic `utils`, `helpers`, `common`, `manager`, or catch-all files.
- Do not introduce glue code that only forwards data between mismatched APIs without clarifying ownership or domain boundaries.
- Prefer direct typed interfaces and project-local conventions over pass-through wrappers, broad adapters, or duplicated helper layers.
- Do not add hardcoded user-facing text in source files. This project has localization files, so route UI labels, messages, empty states, validation text, and accessibility text through the localization layer.
- Do not add `workaround`, `hack`, `temporary fix`, `quick fix`, `monkey patch`, or broad compatibility shims as normal implementation.
- If a temporary mitigation is unavoidable, keep it narrow, document the triggering issue, and include a removal condition.
- Do not hide failing behavior behind broad `try/catch`, silent fallbacks, sleeps, retries, or feature flags without a clear reason.

---
> Source: [Rabithua/Rote](https://github.com/Rabithua/Rote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
