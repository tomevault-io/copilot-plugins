## is-fast-internet

> Every push to `main` is released automatically by `.github/workflows/publish.yml`. The workflow increments the patch version in `package.json` and `package-lock.json`, validates the package, commits the release version, creates the matching `v*` tag, and publishes to npm through Trusted Publishing (OIDC). A concurrency group serializes releases, and GitHub's built-in token prevents the generated release commit/tag from recursively starting another workflow.

# Repository instructions

Every push to `main` is released automatically by `.github/workflows/publish.yml`. The workflow increments the patch version in `package.json` and `package-lock.json`, validates the package, commits the release version, creates the matching `v*` tag, and publishes to npm through Trusted Publishing (OIDC). A concurrency group serializes releases, and GitHub's built-in token prevents the generated release commit/tag from recursively starting another workflow.

Before pushing a package change to `main`:

1. Run `npm run typecheck`, `npm test`, and `npm pack --dry-run`.
2. Commit and push the change without manually bumping the version for an ordinary main-branch release.
3. Let the workflow create the patch version commit and tag and publish only after validation succeeds.
4. Verify the release with `npm view is-fast-internet@<version> --registry=https://registry.npmjs.org/`; do not consider it complete until npm serves the new version.

Do not publish commits from branches other than `main`.

---
> Source: [okasi/is-fast-internet](https://github.com/okasi/is-fast-internet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
