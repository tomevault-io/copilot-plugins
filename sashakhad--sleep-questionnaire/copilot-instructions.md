## sleep-questionnaire

> All contributors must ensure the following before committing code:


# Pre-Commit Checks Rule

All contributors must ensure the following before committing code:

- Run code formatters (Prettier, ESLint with --fix) to auto-fix style and lint issues.
- Run type-checking (e.g., `pnpm run type-check`) to ensure type safety.
- Run all unit tests (e.g., `pnpm run test`) and ensure they pass.
- Run all E2E tests (e.g., `pnpm run test:e2e` or `pnpm cypress run`) and ensure they pass.
- Only commit code that passes all of the above checks.

## Example Commands

```bash
pnpm run lint --fix
pnpm run format
pnpm run type-check
pnpm run test
pnpm run test:e2e # or pnpm cypress run
```

## Automation (Recommended)

- Use Husky or a similar tool to automate these checks as pre-commit or pre-push hooks.
- Document any exceptions or overrides in the relevant place (e.g., README or commit message).

## Rationale

- Ensures code quality, consistency, and reliability.
- Prevents broken or unformatted code from entering the repository.
- Reduces review and CI friction by catching issues early.

---

- Ensures code quality, consistency, and reliability.
- Prevents broken or unformatted code from entering the repository.
- Reduces review and CI friction by catching issues early.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sashakhad) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
