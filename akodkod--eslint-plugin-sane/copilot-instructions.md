## eslint-plugin-sane

> Custom ESLint plugin with opinionated rules for sane code formatting.

# eslint-plugin-sane

Custom ESLint plugin with opinionated rules for sane code formatting.

## Commands

- `bun test` — run all tests
- `bun test rules/<name>.test.ts` — run tests for a specific rule
- `bun run build` — build to `dist/` (targets Node)
- `bun run type-check` — typecheck without emitting

## Project Structure

- `index.ts` — plugin entry point, exports all rules
- `rules/<name>.ts` — rule implementation
- `rules/<name>.test.ts` — tests for the rule (colocated)

## Adding a New Rule

1. Create `rules/<name>.ts` implementing `Rule.RuleModule` from `eslint`
2. Create `rules/<name>.test.ts` with tests using `bun:test` and ESLint's `RuleTester`
3. Register the rule in `index.ts`

## Conventions

- Use Bun for all tooling (test, build, install, run)
- TypeScript with strict mode enabled
- ESLint flat config (v10) — no legacy `.eslintrc` patterns
- Rules use the `Rule.RuleModule` type from `eslint`
- Tests use `RuleTester` with `languageOptions.parserOptions.ecmaFeatures.jsx: true` for JSX rules
- Keep rules self-contained in single files — no shared utilities unless truly needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/akodkod)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/akodkod)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
