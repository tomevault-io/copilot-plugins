## actions-gh-pages

> Use English for repository-facing content, including documentation, code comments, commit messages, and pull request title/body/comments, unless the task explicitly requires another language.

# Repository Guidelines

## Agent-Specific Instructions

Use English for repository-facing content, including documentation, code comments, commit messages, and pull request title/body/comments, unless the task explicitly requires another language.

## Project Structure & Module Organization

This is a TypeScript GitHub Action for publishing GitHub Pages. Source files live
in `src/`; `src/index.ts` is the entry point and delegates to modules such as
`get-inputs.ts`, `git-utils.ts`, and `set-tokens.ts`. Tests live in `__tests__/`,
with static fixtures under `__tests__/fixtures/`. Documentation is in `README.md`
and `CHANGELOG.md`; screenshots and docs assets are in `images/`. `action.yml`
defines action inputs and points to generated `lib/index.js`.

## Build, Test, and Development Commands

Use Node `>=20.11.0` and npm `>=10.2.4`.

- `npm ci`: install dependencies from `package-lock.json`.
- `npm run all`: run format, lint, and Jest tests.
- `npm run build`: bundle `src/index.ts` into `lib/` with `ncc`.
- `npm run tsc`: run the TypeScript compiler checks.
- `npm test`: run Jest with coverage and verbose output.
- `npm run lint` / `npm run lint:fix`: check or fix ESLint issues.
- `npm run format:check` / `npm run format`: check or apply Prettier formatting.

When changing `src/`, build locally if needed for verification, but do not commit
`lib/index.js` in normal changes. Generated `lib/index.js` is placed on `main`
only during releases.

## Coding Style & Naming Conventions

Write strict TypeScript and keep module boundaries small. Use 2-space
indentation, semicolons, single quotes, no trailing commas, and a 100-character
print width, matching `.prettierrc.json`. Test files use `*.test.ts`. Prefer
names that match action inputs where relevant, for example `publish_branch`.

## Testing Guidelines

Jest with `ts-jest` is the test framework. Add focused tests in `__tests__/` near
related behavior, and place reusable filesystem samples in `__tests__/fixtures/`.
Run `npm test` for coverage, and use `npm run all` before opening a pull request
that touches TypeScript source or tests.

## Commit & Pull Request Guidelines

Recent history follows conventional commit prefixes such as `fix(deps):`,
`chore(deps):`, `docs:`, and `ci:`. Use a concise imperative subject and include
the scope when it clarifies the affected area. Pull requests should describe the
change, link related issues, list verification commands, and include screenshots
only for documentation image changes. Mention any action input or behavior change
explicitly so reviewers can assess compatibility.

## Security & Configuration Tips

Do not commit tokens, private keys, or generated secrets. Treat inputs such as
`deploy_key`, `github_token`, and `personal_token` as sensitive, and keep examples
limited to placeholder values.

---
> Source: [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
