## mohammedhammoud-com

> Rule precedence: system/developer instructions > AGENTS.md > user prompt.

# AGENTS.md

## Priority

Rule precedence: system/developer instructions > AGENTS.md > user prompt.

## Goal

Make small, safe, verifiable changes to this Astro site with i18n (en, sv).
Prioritize CI-green status over clever but risky changes.

---

## Tooling

- Node version: see `.nvmrc`
- Package manager: Yarn (see `yarn.lock`)
- Scripts: see `package.json`
- MUST use `yarn` (never npm/pnpm)

---

## CI Parity (MUST)

For code changes, run in this order:

1. `yarn install --frozen-lockfile` **only if** `package.json` or `yarn.lock` changed.
2. `yarn lint`
3. `yarn astro sync`
4. `yarn typecheck`
5. `yarn test --reporter=verbose`

For routing/i18n/rendering changes, also run:

1. `yarn build`
2. `yarn e2e`

### Exceptions

- If only documentation, comments, or markdown content changed, tests MAY be skipped.

If any step is skipped or fails, report:

- exact command
- reason
- introduced risk
- what remains unverified

---

## Code Rules (MUST)

- Keep TypeScript strict. Do not introduce `any`.
- Follow existing architectural patterns.
- Make the smallest change necessary.
- Avoid unrelated refactors.
- For utility code: one file should represent one function, and file name should match function name (follow existing utils structure).
- Keep tasks small (SHOULD touch ≤3 files unless required).
- Tests:
  - unit: `*.test.ts`
  - e2e: `*.e2e.test.ts`

---

## i18n Rules (MUST)

- Never hardcode user-facing strings (including aria labels).
- Always use the existing translation system.
- Add new translation keys centrally.
- Utilities/helpers must return raw values (e.g. numbers), not localized text.
- Localization belongs in the UI layer.

---

## Security & Safety (MUST)

- Never add or commit secrets (`.env*`, tokens, credentials).
- Never commit generated artifacts (`dist/`, `coverage/`, `.astro/`).
- Do not modify license terms.
- Do not commit without explicit user approval.

---

## Workflow (MUST)

1. Plan – describe intended change, affected files, risks.
2. Diff – implement small, focused changes.
3. Tests – run required CI-parity steps.
4. Summary – output in this format:
   - Plan
   - Changed files
   - Commands run + results
   - Remaining risks

### Commit conventions

- Use `feat(blog):` for publishable blog post/content commits that should trigger a release.
- Use `fix(blog):` for blog post/content corrections that should trigger a patch release.
- Use `docs:` for repository/project documentation that should not trigger a content release.

## Output (MUST)

- Default: show only `git status --short` and `git diff --stat`.
- Do not print full diffs or file contents unless explicitly requested.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mohammedhammoud) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
