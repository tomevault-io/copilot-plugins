## pre-ship-checks

> Required checks before opening or shipping a PR


# Pre-ship checks (PRs)

Before opening a PR or declaring work ready to ship, **always** run these commands and fix any failures:

```bash
npm run format
npm run lint
npm run lint:styles
```

Also run the usual quality gates when the change touches app code:

```bash
npm run typecheck
npm run lint
npm test
npm run build
```

If `npm run format` modifies files, include those changes in the PR branch before pushing.

Do not skip `format` or `lint:styles` even when the diff looks TypeScript-only — SCSS and other formatted files may still need updates.

Once all of the checks and formatting passes, please update implementation documents, customer facing documents and developer facing documents with all updated, completed work.

---
> Source: [chris-712interactive/cleanScheduler](https://github.com/chris-712interactive/cleanScheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
