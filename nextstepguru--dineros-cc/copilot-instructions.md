## dineros-cc

> Global safety and automation constraints for the Dineros project


# Global Safety Rules

## Build and Server

- **Never** automatically build the project. Do not run `pnpm run build`, `npm run build`, or `nuxt build` unless the user explicitly requests it.
- **Never** automatically start the development server. Do not run `pnpm dev`, `npm run dev`, or `nuxt dev` unless the user explicitly requests it.
- **Typecheck**: Do not run `pnpm typecheck`, `nuxt typecheck`, `npm run typecheck`, or equivalent unless the user explicitly asks.

## Destructive and Side-Effect Operations

- Do not run destructive or mutating operations (e.g. database reset, migration apply, deploy, secret rotation) without explicit user request.
- Do not commit, push, or open PRs unless the user explicitly asks.

## Documentation and Response Style

- Do not generate markdown files (README, CONTRIBUTING, docs) unless the user explicitly directs it.
- Prefer direct, minimal answers. Do not give lengthy summaries unless asked. When reporting work, emphasize what is next or what is left rather than narrating every completed step.

## Rule Updates

- Update Cursor rules (`.cursor/rules/*.mdc`) when project conventions or tooling change and the rules no longer match.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NextStepGuru) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
