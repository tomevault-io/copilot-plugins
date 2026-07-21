## homebridge-wiz-lan

> This file documents the consistent release ritual for this repo, derived from past tags and merge history. Follow it whenever a PR is merged or a release is cut.

# homebridge-wiz-lan — maintainer guide

This file documents the consistent release ritual for this repo, derived from past tags and merge history. Follow it whenever a PR is merged or a release is cut.

## Release ritual

Releases are always cut as a **separate direct commit to `master`**, never bundled into a PR merge.

### 1. Merge the PR
- Use a true merge commit (no squash).
- Dependabot PRs and human-contributor PRs are both merged the same way.

### 2. Decide whether to release now or batch
- **Release immediately** for user-visible features or fixes.
- **Batch** for dependabot bumps and small follow-ups — fold them into the next human-driven release.

### 3. Cut the release (single direct commit on `master`)

In one commit, update all of:

- **`package.json`** — bump `version` (semver: patch for fix, minor for feature, major for breaking).
- **`CHANGELOG.md`** — add a new section at the top in the existing format:
  ```
  ## X.Y.Z
  - [FEAT] / [FIX] short description
  - Thank you [@handle](https://github.com/handle) for [#NN](https://github.com/kpsuperplane/homebridge-wiz-lan/pull/NN)
  ```
  - Use `[FEAT]` and `[FIX]` prefixes.
  - Credit human contributors by handle + PR link. Omit credit for dependabot.
- **`README.md` contributors block** — if this release includes a first-time external contributor, add them under the credits section in the same format as existing entries (`#### [@handle]` then `[#NN title](url)`).
- **`package-lock.json`** — regenerate via `npm install` so it matches the new version.

The commit message is just the version string, e.g. `3.2.6`.

### 4. Tag and push
- Tag the bump commit with **`vX.Y.Z`** (always include the `v` prefix — historical tags were inconsistent; new tags should standardize on `v`).
- Push the commit and the tag to `origin/master`.
- npm publish runs from `.github/workflows/npm-publish.yml` on tag push.

## What NOT to do
- Don't bump `package.json` inside a feature PR — the version bump is a separate maintainer commit on master.
- Don't squash-merge — preserve the merge-commit history.
- Don't tag the merge commit; tag the version-bump commit that follows it.
- Don't credit dependabot in CHANGELOG or README.
- Don't create a release per dependabot PR — batch them.

## Quick reference: file touch list per release

| File | Why |
| --- | --- |
| `package.json` | version bump |
| `package-lock.json` | sync with new version |
| `CHANGELOG.md` | new section with FEAT/FIX bullets and contributor credit |
| `README.md` | add new external contributor to credits block (only if applicable) |

## Testing

- Test runner: **Bun** (`bun test`). Production runtime is still Node — Bun is used only for the test suite. Tests live in `test/`, mirroring the `src/` layout.
- Run locally:
  - `bun test` — run the suite.
  - `bun test --coverage` — run with coverage (text + lcov reports).
  - `bun test --watch` — watch mode for iterative work.
- Required before cutting a release: **`bun test` must pass**. Run it before the version-bump commit in the ritual above.
- Coverage target: **≥80% lines** on `src/` (excluding `src/index.ts`, `src/constants.ts`, `src/types.ts`). Don't lower this — instead, add tests when adding code.
- CI runs `.github/workflows/test.yml` on every PR and push to master. Two jobs:
  - `bun-test` runs the suite + coverage upload.
  - `node-smoke` runs `npm run build` across Node 18/20/22 to ensure the published JS still compiles on every supported Node line.
- When fixing a regression, add a test that would have caught it. The cache/state and network layers have history — see `test/accessories/WizLight/pilot.test.ts` for examples tied to issues #96/#101/#143/#145/#159.
- Mocking conventions:
  - `test/__mocks__/homebridge.ts` provides the Homebridge HAP surface (Service, Characteristic, PlatformAccessory, AdaptiveLightingController).
  - `test/__helpers__/factories.ts` provides `makeFakeWiz()`, `makeDevice()`, `makeLightPilot()`, `makeSocketPilot()`, `FakeSocket`.
  - For `pilot.ts` tests, stub `src/util/network`'s `getPilot`/`setPilot` via `mock.module` and synthesize replies. For `wiz.ts` tests, mock only `dgram` so the real network functions still operate on the FakeSocket.

---
> Source: [kpsuperplane/homebridge-wiz-lan](https://github.com/kpsuperplane/homebridge-wiz-lan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
