## upgrade-helper

> Upgrade Helper is a static web tool that helps developers upgrade React Native applications by showing the full diff between two versions and layering in curated guidance.

# Upgrade Helper — Agent Guide

## Overview

Upgrade Helper is a static web tool that helps developers upgrade React Native applications by showing the full diff between two versions and layering in curated guidance.

This repository is valuable because it combines:

- upstream template diffs
- release-specific upgrade notes
- inline comments on important files
- binary file download support
- a simple UI that helps users track upgrade progress

Upgrade Helper is both:

- a trust-sensitive user-facing tool for React Native upgrades
- an open-source repository that requires steady, low-friction maintenance

Treat both responsibilities seriously. Correctness, stability, and clarity matter more than novelty.

---

## Primary counterpart

Your main human counterpart will often be one of the maintainers of this project.

Optimize for reducing maintainer burden while preserving project quality, trust, and stability.

That means you should be useful not only for product and code changes, but also for the ongoing work of maintaining an open-source repository, including triage, reproduction, documentation, test hygiene, CI/workflow fixes, and contributor-facing improvements.

When context is incomplete, do as much of the legwork as possible before asking for help:

- inspect the relevant code and docs
- reproduce the issue if possible
- identify affected files and workflows
- summarize likely causes
- propose the smallest safe next step

Prefer giving maintainers ready-to-use output:

- concise issue summaries
- reproduction notes
- draft PR descriptions
- documentation updates
- focused fix plans
- validation results

---

## Maintainer support scope

This agent supports maintainers with both code work and repository maintenance work.

Treat the following as first-class tasks:

- issue triage and bug reproduction
- narrowing down regressions
- reviewing contributor changes for risk, correctness, and missing validation
- improving README, contributing docs, and contributor guidance
- updating release-specific upgrade content under `src/releases/`
- maintaining tests, fixtures, mocks, and CI/workflow health
- identifying small, safe modernization opportunities in an aging codebase
- preparing clear summaries for maintainers to use in issues, PRs, or discussions

When helping with repository maintenance:

- prefer reducing ambiguity for maintainers
- prefer small, reviewable changes
- surface risks, follow-ups, and missing information clearly
- do not assume maintainers want a broad refactor when a targeted fix will do
- separate immediate fixes from longer-term cleanup ideas

---

## Product priorities

1. Preserve trust in the upgrade guidance
2. Keep user-facing behavior stable and predictable
3. Prefer small, reversible maintenance changes
4. Keep the app static and client-side
5. Keep contributor and maintainer workflows lightweight and clear

---

## Technical reality

This is an older but functioning frontend codebase.

Current stack:

- React 18
- Create React App via `react-app-rewired`
- TypeScript with `strict: true` and `allowJs: true`
- Emotion for styling
- Ant Design for UI components
- Framer Motion for transitions
- Yarn 1
- GitHub Pages deployment

Compatibility notes:

- `.node-version` and GitHub Actions currently target Node 16
- `mise.toml` declares Node 22 locally
- do not assume the repo is ready for Node-22-only changes unless you are explicitly modernizing the toolchain
- use `yarn`, not npm or bun, unless explicitly asked to change package management

This codebase should be treated as legacy-but-live: stable, useful, and worth improving carefully.

---

## Read first

Before making significant changes, read:

- `README.md`
- `CONTRIBUTING.md`
- `package.json`
- `src/components/pages/Home.tsx`
- `src/utils.ts`
- `src/releases/index.js`
- `src/releases/types.d.ts`

If touching UI and diff rendering, also read:

- `src/components/common/DiffViewer.tsx`
- `src/components/common/Diff/DiffSection.tsx`
- `src/components/common/Diff/DiffHeader.tsx`
- `src/__tests__/Home.e2e.spec.ts`

If touching release guidance, study an existing release file first, for example:

- `src/releases/react-native/0.60.tsx`
- `src/releases/react-native/0.77.tsx`

If touching maintainer or contributor workflows, also read:

- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/workflows/push.yml`
- `.github/workflows/deploy.yml`

Then read the files directly relevant to your task.

---

## Architecture map

- `src/components/pages/Home.tsx`
  - top-level page orchestration
  - theme toggle
  - URL state
  - settings and version selectors

- `src/hooks/fetch-release-versions.ts`
  - fetches available release versions from upstream repos

- `src/hooks/fetch-diff.ts`
  - fetches raw diff files and parses them for rendering

- `src/utils.ts`
  - central URL builders
  - changelog URL generation
  - version filtering
  - app name and package replacement
  - other shared logic

- `src/releases/`
  - curated release notes and inline upgrade comments
  - this is part of the product, not incidental content

- `src/theme/`
  - light/dark theme tokens

- `src/utils/test-utils.ts`
  - Puppeteer helpers and mocked network responses for screenshot e2e tests

- `.github/workflows/`
  - CI and deployment workflows

---

## Working principles

### 1. Preserve trust

Do not invent release guidance, upgrade behavior, or troubleshooting advice.

When adding or updating release content, prefer authoritative sources such as:

- official React Native blog posts
- official changelogs
- relevant upgrade-support issues
- upstream release notes for macOS and Windows variants

If a claim is uncertain, omit it or leave a clear follow-up note instead of guessing.

### 2. Treat URL state as part of the product API

The app relies on URL parameters and anchor links for deep-linking and restoring state.

Be careful when changing behavior related to:

- `from`
- `to`
- `package`
- `language`
- `name`
- hash anchors

Preserve backward compatibility whenever possible.

### 3. Prefer incremental modernization

This repo is old, but it works.

Do not casually combine product work with major ecosystem migrations such as:

- CRA to Vite, Next, or another build system
- Yarn to pnpm or bun
- Emotion to another styling system
- Ant Design replacement

If modernization is requested, plan it as a focused effort with clear validation and rollback points.

### 4. Respect upstream data contracts

Release lists and diff files come from upstream repositories.

If you change fetching or parsing behavior:

- verify the upstream URL structure
- update mocks and fixtures
- validate the loading flow and rendered output

Do not introduce server-side assumptions into this static app unless explicitly asked.

### 5. Treat release metadata as product code

Files under `src/releases/` are a core part of the user experience.

When editing them:

- keep copy concise and actionable
- link to real sources
- keep inline comments tightly targeted
- verify `fileName`, `lineNumber`, and `lineChangeType` against the real diff
- use existing patterns and the `ReleaseT` structure from `src/releases/types.d.ts`

### 6. Preserve the core upgrade flows

Be careful around these user-facing behaviors:

- selecting current and target versions
- filtering release candidates
- switching package or platform (`react-native`, `react-native-macos`, `react-native-windows`)
- switching RNW language (`cpp`, `cs`)
- replacing default app name and package in displayed paths and file contents
- rendering diffs and anchor links
- binary file download, view, and copy actions
- done-state tracking
- dark mode

### 7. Keep styling consistent

Use the existing Emotion + Ant Design approach.

When changing UI:

- preserve light and dark mode parity
- avoid introducing a second styling system
- avoid unnecessary visual churn in a stable tool
- prefer clarity and legibility over decorative redesigns

### 8. Optimize for maintainer efficiency

When a task can be completed by producing a high-quality summary, reproduction, draft, or targeted fix plan, do that work proactively.

Good maintainer support often looks like:

- narrowing a bug from “something is broken” to a specific file and cause
- preparing a minimal patch instead of a broad rewrite
- updating docs while fixing the underlying issue
- identifying whether a problem belongs in code, content, tests, CI, or upstream dependencies

---

## Change-specific guidance

### Adding or updating release guidance

If a task is about release notes, comments, or upgrade-specific guidance:

1. read `src/releases/types.d.ts`
2. read one or two existing release files first
3. add or edit the appropriate file under `src/releases/<package>/`
4. update `src/releases/index.js` if needed
5. verify links and comment targets carefully

### Touching shared utilities

`src/utils.ts` is a hotspot.

If you change it:

- add or update tests in `src/__tests__/utils.spec.ts`
- check version filtering behavior
- check changelog URL behavior
- check app name and package replacement behavior

### Touching remote fetch logic

If fetch URLs or parsing rules change:

- update mocks in `src/utils/test-utils.ts`
- update fixtures if needed
- validate both happy-path loading and user-visible output

### Touching visible UI

If the UI changes materially:

- run the app locally with `yarn start`
- validate the key flows manually
- run e2e screenshots if practical
- update snapshots intentionally, not incidentally

### Touching contributor or maintainer workflows

If changing docs, templates, workflows, or maintenance ergonomics:

- preserve contributor clarity
- keep CI predictable
- avoid adding unnecessary process overhead
- explain the maintainer benefit clearly

---

## Validation

Minimum required before finishing work:

- `yarn lint`
- `yarn typecheck`
- `yarn test --runInBand`

For UI or behavior changes, also do:

- `yarn start`
- manual validation at `http://localhost:3000`

For higher-confidence UI changes, use:

- `yarn test-e2e`
- or `yarn docker-test-e2e`

Notes:

- CI currently covers lint, typecheck, and non-e2e tests
- screenshot e2e coverage is heavier and local-oriented
- if snapshots change, explain why

---

## Agent operating loop

For non-trivial tasks, follow this sequence:

1. Scout
   - identify the affected user flow
   - identify the relevant upstream data source
   - identify the tests that should protect the change

2. Plan
   - choose the smallest safe change
   - call out risk areas and compatibility concerns

3. Implement
   - change the narrowest layer possible
   - avoid opportunistic rewrites

4. Review
   - challenge the change for correctness
   - check edge cases
   - verify it fits the existing architecture

5. Recover
   - if old tooling blocks progress, apply the smallest unblock
   - document broader tech debt separately instead of smuggling in a large migration

---

## Reporting back

When summarizing work:

- be concise
- say what changed
- say what user flow or maintainer workflow it affects
- say what risks remain
- list the exact validation commands run

---

## PR and change hygiene

- keep changes small and focused
- preserve readable diffs
- for UI changes, include screenshots or a short visual summary when useful
- explain motivation, impact, and test plan clearly
- do not bury risky refactors inside unrelated fixes

---
> Source: [react-native-community/upgrade-helper](https://github.com/react-native-community/upgrade-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
