## palworld-server-dashboard

> - Keep changes small and boring; reuse existing components/utilities before adding new ones.

# Project agent instructions

- Keep changes small and boring; reuse existing components/utilities before adding new ones.
- Run `npm run check` before handing off changes that touch app code.
- This project uses release-please for changelogs. Write commits in Conventional Commits format:
  - `feat(scope): add new capability`
  - `fix(scope): correct broken behavior`
  - `perf(scope): improve performance`
  - `docs(scope): update documentation`
  - `refactor(scope): simplify without behavior change`
  - `chore(scope): maintenance`
- Use the changelog-visible types when possible: `feat`, `fix`, `perf`, `revert`, `docs`, `chore`, `refactor`.
- Hidden-but-valid release types: `test`, `build`, `ci`, `deps`.
- Mark breaking changes with `!` after the type/scope, and include a `BREAKING CHANGE:` footer.

---
> Source: [RNZ01/palworld-server-dashboard](https://github.com/RNZ01/palworld-server-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
