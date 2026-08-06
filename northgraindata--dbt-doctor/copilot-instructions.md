## dbt-doctor

> Clone of [react-doctor](https://github.com/millionco/react-doctor) adapted for dbt projects. **Static analysis and health checks for dbt projects** (SQL, YAML, Jinja).

# dbt-doctor

Clone of [react-doctor](https://github.com/millionco/react-doctor) adapted for dbt projects. **Static analysis and health checks for dbt projects** (SQL, YAML, Jinja).

## Stack

- pnpm monorepo: `packages/dbt-doctor` (CLI), `@dbt-doctor/core`, `@dbt-doctor/project-info`, `@dbt-doctor/types`, `dbt-doctor-rules`, `packages/website`
- Lint: custom TypeScript rules + optional **sqlfluff** subprocess
- Scoring: local formula in `@dbt-doctor/core`; optional API at `dbt-doctor.northgraindata.com`

## Commands

```bash
pnpm install
pnpm build
pnpm test
pnpm lint
pnpm typecheck
```

## Conventions

- Use `@antfu/ni`: `ni`, `nr`, `nun`
- kebab-case files; arrow functions; interfaces over types
- Magic numbers in `constants.ts` with `SCREAMING_SNAKE_CASE` suffixes
- `// HACK:` prefix for non-obvious workarounds

---
> Source: [northgraindata/dbt-doctor](https://github.com/northgraindata/dbt-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
