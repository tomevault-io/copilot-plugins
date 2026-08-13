## testing-ci

> Vitest, CI pipeline, lint, and release packaging expectations


# Testing & CI Rules

## CI pipeline (`.github/workflows/ci.yml`)

Mirrored locally by:

```bash
npm run ci
```

Steps in order:

1. `npm ci` (root + extension)
2. `npm run lint`
3. `npm run format:check`
4. `npm run typecheck`
5. `npm run test`
6. `npm run build:extension`
7. `npm run build:web`

**All must pass** before merge.

## Test stack

- **Vitest 3** + **happy-dom**
- Config: `vitest.config.ts` (root)
- All tests live under `tests/` (see `tests/README.md`)
- Extension: `tests/extension/**/*.test.ts`
- Landing: `tests/landing/**/*.test.ts`

## Source ↔ test mapping (mandatory)

| Source | Test |
|--------|------|
| `extension/src/<path>/<file>.ts` | `tests/extension/<path>/<file>.test.ts` |
| `src/<path>/<file>.ts` | `tests/landing/<path>/<file>.test.ts` |

Same basename. Import SUT via `@/` - do not co-locate `*.test.ts` under `src/`.

| Change | Test path |
|--------|-----------|
| `extension/src/storage/codec.ts` | `tests/extension/storage/codec.test.ts` |
| `extension/src/storage/db.ts` | `tests/extension/storage/db.test.ts` |
| `extension/src/lib/workspaceTree.ts` | `tests/extension/lib/workspaceTree.test.ts` |
| `extension/src/store/useStore.ts` | `tests/extension/store/useStore.test.ts` (+ `moveWorkspaceItem.test.ts` for moves) |
| `extension/src/editor/markdownPaste.ts` | `tests/extension/editor/markdownPaste.test.ts` |
| `extension/src/lib/attachments/*` | `tests/extension/lib/attachments/*.test.ts` |
| `extension/src/lib/text.ts` | `tests/extension/lib/text.test.ts` |
| `src/lib/seo.ts` | `tests/landing/lib/seo.test.ts` |

## Running tests

```bash
npm run test                                          # all
npm run test:watch                                    # watch mode
npm run test -- tests/extension/editor/               # scoped
npm run test -- tests/landing/lib/seo.test.ts         # landing SEO
```

## Landing SEO verification

After touching `src/lib/seo.ts`, `ai-content.json`, `generate-sitemap.mjs`, or `llms[.]txt.ts`:

```bash
npm run test -- tests/landing/lib/seo.test.ts
npm run generate:seo
npm run dev:web
curl -s http://localhost:8080/robots.txt
curl -s http://localhost:8080/sitemap.xml
curl -s http://localhost:8080/llms.txt | head
```

Production check (after deploy): `curl https://<VITE_SITE_URL>/llms.txt`

Set `VITE_SITE_URL` on your hosting provider so generated files and JSON-LD use the canonical domain.

## Lint & format

```bash
npm run lint
npm run format        # write
npm run format:check  # CI
```

ESLint 9 flat config. Unused vars → prefix `_`.

## Typecheck

```bash
npm run typecheck
# root tsconfig + extension tsconfig
```

## Extension packaging

```bash
npm run package:extension
# → public/mymemos-extension.zip
```

Required for landing download button. Warn in UI if missing. Run manually when shipping extension changes that should refresh the download ZIP (not hooked on every commit).

## Local quality gates

| Gate | Command |
|------|---------|
| Pre-push | `npm run check` |
| Full / CI | `npm run ci` (`check` + builds) |

Skip pre-push with `SKIP_PRE_PUSH_CI=1` only when necessary.

## Build guards

- Do not run `build:extension` during `npm run dev` - kills HMR
- `npm run build:web` builds demo + landing - required for deploy

## PR expectations

- Summary of why (not just what)
- Test plan checklist
- Screenshots for UI changes
- Note storage/schema impact if applicable

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
