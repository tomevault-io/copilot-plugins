## maakdown

> This file defines repo-level operating rules for any agent working in Maakdown.

# AGENTS

This file defines repo-level operating rules for any agent working in Maakdown.

## Core Rules

1. Follow the approved docs in `docs/`; if implementation must deviate, update `DEV_CONTEXT.md` and the relevant docs.
2. Keep `docs/task-tracker.md` current when tasks are added, reprioritized, started, completed, or blocked.
3. Keep `DEV_CONTEXT.md` current whenever project state meaningfully changes.
4. Do not push to a remote repository, create remote branches, or open pull requests unless the user explicitly asks.
5. Never commit signing certificates, private keys, notarization credentials, store credentials, provisioning profiles, or generated release binaries.

## Verification And Commit Policy

- Every meaningful implementation change should end with verification that matches the scope of the change.
- Check `git status` before editing. The worktree may contain user changes.
- Prefer narrow changes and avoid bundling unrelated cleanup into product work.
- Use the cheapest useful verification first:
  - docs/tree-only changes: `find`, `rg`, and `git status`
  - frontend changes: `npm install` if needed, then `npm run check` and `npm run build`
  - backend changes: `go test ./...` once Go is installed
  - full app changes: `wails build` once Wails and Go are installed
- After verification succeeds, commit the completed coherent unit of work unless the user asks not to.
- Do not batch unrelated work into one commit when they can be split cleanly.
- If verification is blocked by missing local tools or functional constraints, record the blocker in `DEV_CONTEXT.md`.

## GitHub Actions Verification

- Verify changes to GitHub Actions workflows or scripts used directly by those
  workflows on the remote `ci/sandbox` branch before pushing them to `main`.
- Keep `workflow_dispatch` available on the workflow under test.
- Use this flow without creating a local sandbox branch:
  1. Force-push local `HEAD` to `origin/ci/sandbox`.
  2. Trigger the workflow with
     `gh workflow run <workflow-file> --ref ci/sandbox`.
  3. Watch the run for up to 10 minutes and inspect failures before touching
     `main`.
- Retry at most three remote attempts while correcting workflow definitions,
  workflow scripts, runner setup, permissions, artifact handling, or other CI
  infrastructure owned by the change.
- If the run is still active after 10 minutes, stop watching and ask the user
  to report its final result before continuing.
- If the first sandbox attempt succeeds, the verified change may be pushed to
  `main` after local verification.
- If verification requires multiple sandbox attempts, pause for user
  confirmation before pushing to `main`.
- Cross-platform workflow verification establishes that the workflow,
  supporting scripts, tests, and builds execute on the declared runners. Do
  not fix unrelated product behavior merely to turn a workflow run green;
  report product failures separately unless the user expands the task.

## DEV_CONTEXT.md Policy

`DEV_CONTEXT.md` is required project memory. Update it whenever one or more of the following changes:

- the purpose of files or directories becomes clearer
- a new file or subsystem is added
- a planned task is added, removed, reprioritized, or redefined
- a task is completed
- an implementation detail materially changes
- an architectural or product decision is made or revised
- verification commands or blockers change

At minimum, `DEV_CONTEXT.md` should contain:

- project summary
- tree-level summary
- current phase and active focus
- planned tasks and features
- completed tasks with timestamps
- implementation notes and decision log
- current verification commands and blockers

## Task Tracker Policy

`docs/task-tracker.md` is the project/progress tracker. It should be updated as work lands:

- `Todo`: not started
- `In Progress`: actively being worked
- `Blocked`: cannot proceed without a dependency or decision
- `Done`: implemented and verified
- `Deferred`: intentionally out of current scope

Each task should have an owner field, dependencies, exit criteria, and verification notes.

## Remote Policy

- Never push.
- Never create or update a remote branch.
- Never open a pull request.
- Exception: only do any of the above if the user explicitly instructs you to.

## Signing And Release Policy

The user plans to sign macOS and Windows builds with their own certificates. Agents must preserve that structure without storing secrets:

- macOS signing assets belong in the user's keychain or external secret storage, not git.
- Windows code-signing certificates (`.pfx`, `.p12`, `.cer`, private keys) must not be committed.
- Notarization credentials, Apple API keys, timestamp server credentials, and CI secrets must be referenced by environment variable names or secret-manager keys only.
- Keep signing-safe templates, entitlements, manifests, and documentation in git.
- Generated installers, disk images, signed binaries, symbol archives, and notarization logs are build artifacts and should remain ignored unless the user explicitly asks otherwise.

## Release Guide Policy

- When asked to prepare, cut, sign, publish, verify, document, or complete a
  release, read and follow `docs/RELEASING.md` as the canonical release
  runbook.
- Use `docs/release-checklist.md` as the release gate. Do not tag or publish
  while required checks are failing or release blockers remain unaccepted.
- Final GitHub release notes must follow the format in `docs/RELEASING.md`:
  opening summary, homepage, exact downloads table, highlights, notable changes,
  notes, and full changelog.
- For every public feature release, execute
  `docs/release-site-refresh.md` after the artifacts and release notes are
  final, then verify the GitHub Pages deployment and live landing page.
- A documentation-only release or emergency artifact rebuild may skip new
  screenshots only when the user-facing app surface is unchanged. Record the
  exception in `DEV_CONTEXT.md`.
- Keep `DEV_CONTEXT.md`, `docs/task-tracker.md`, and any active release tracker
  aligned with the release tag, verification evidence, published assets,
  release notes, and site-refresh status.
- Remote pushes, tags, GitHub Release mutations, and publication still require
  explicit user authorization under the Remote Policy.

## Beast Mode

If the user says to "go beast mode" on a goal:

- Work autonomously toward the requested goal without pausing for ordinary decisions.
- If a decision is needed and there is a reasonable recommended path, take that path and continue.
- Break work into coherent milestones and create verified commits along the way.
- Keep `DEV_CONTEXT.md` and `docs/task-tracker.md` updated as milestones land.
- Stop only if a real blocker, destructive external action, missing credential, or user-only decision prevents progress.

Beast mode increases autonomy for implementation choices, not for remote pushes or secret-handling.

## Practical Workflow

1. Read `DEV_CONTEXT.md` and `docs/task-tracker.md`.
2. Inspect relevant docs/code.
3. Implement the next coherent task.
4. Verify the result.
5. Update `DEV_CONTEXT.md` and `docs/task-tracker.md`.
6. Commit verified work.

## Practical Defaults

- For product bugs, identify the root cause, add or adjust a regression test,
  then implement the fix.
- For workflow failures, distinguish workflow/setup/script defects from product
  test failures before editing code.
- Keep workflow and release documentation aligned with the actual scripts and
  triggers.

## UI And Design System Policy

- UI changes must first use existing primitives from
  `frontend/src/design-system`.
- If a needed reusable control does not exist, add it to the design system
  before using it in feature code: create the Svelte primitive, export it from
  `frontend/src/design-system/index.ts`, render its important states in
  `DesignSystemGallery.svelte`, add component CSS under `.ds-*` selectors, and
  update `docs/design-system/README.md`.
- Keep feature CSS for layout, placement, and domain-specific presentation.
  Do not recreate reusable button, icon button, popover, dialog, chip, toggle,
  checkbox, menu, field, or settings-row styling inside feature components.
- Exceptions are allowed when extracting a primitive would make the code less
  clear or fight the platform. Good examples are OS-native window chrome and
  drag regions, browser-rendered Markdown content owned by the sanitizer,
  one-off layout containers, measurement/virtualizer wrappers, and highly
  specialized surfaces whose behavior is tightly coupled to a single feature.
  Document the exception in the component or relevant docs when it is not
  obvious from the code.

## App Icon Update Procedure

The app has three icon surfaces that must be updated together:

### Source Files

| File | Purpose |
|---|---|
| `docs/design-system/maakdown.icon` | Apple Icon Composer bundle (layered, supports macOS light/dark/tinted theming) |
| `docs/design-system/maakdown.icon/Assets/maakdown_light.png` | Light-theme master image. Single source — also derives `build/appicon.png` and the light title-bar mark |
| `docs/design-system/maakdown.icon/Assets/maakdown_dark.png` | Dark-theme master image. Single source — also derives the dark title-bar mark |

The masters live **only** inside the `.icon` bundle's `Assets/` directory (the same
files `icon.json` references), so there is one copy of each. All other icon
surfaces below are *derived* from these and regenerated, never hand-edited.

### 1. macOS App Icon (Finder / Dock)

Wails auto-generates a legacy `iconfile.icns` from `build/appicon.png`, but macOS
theming requires the `.icon` bundle compiled by Apple's `actool` into `Assets.car`
plus a proper `maakdown.icns` fallback. The postbuild script handles this:

```bash
wails build -platform darwin/arm64
scripts/postbuild-darwin.sh            # compiles .icon → Assets.car + maakdown.icns
```

`scripts/postbuild-darwin.sh` does the following:
1. Runs `xcrun actool` to compile `docs/design-system/maakdown.icon` into
   `Assets.car` (themed asset catalog) and `maakdown.icns` (legacy fallback).
2. Copies both into `Maakdown.app/Contents/Resources/`.
3. Removes the Wails-generated `iconfile.icns` so it cannot compete.
4. Touches the app bundle so Finder re-reads icon metadata.

`build/darwin/Info.plist` uses both `CFBundleIconFile` and `CFBundleIconName`
pointing to `maakdown` (matching the `actool --app-icon` name).

Requires: Xcode command-line tools (`xcrun actool`).

### 2. Windows / Linux App Icon

Wails generates the Windows `.ico` and Linux icon from `build/appicon.png`.
Update it by copying the light-theme PNG:

```bash
cp docs/design-system/maakdown.icon/Assets/maakdown_light.png build/appicon.png
```

This is done before `wails build`; Wails handles the rest automatically.

### 3. Title Bar Icon (Frontend)

The toolbar brand mark uses theme-aware icons imported as Vite assets:

- `frontend/src/assets/app-icon-light.png` — shown when theme is light
- `frontend/src/assets/app-icon-dark.png` — shown when theme is dark

Regenerate from the bundle masters (scaled to 64×64):

```bash
sips -z 64 64 docs/design-system/maakdown.icon/Assets/maakdown_light.png --out frontend/src/assets/app-icon-light.png
sips -z 64 64 docs/design-system/maakdown.icon/Assets/maakdown_dark.png  --out frontend/src/assets/app-icon-dark.png
```

The icon switch is wired in `WorkspaceToolbar.svelte` via
`config.theme === 'dark' ? appIconDark : appIconLight`.

### 4. Windows Markdown File Icon

Windows file associations do not synthesize an app-icon overlay for ProgId
document icons. Maakdown therefore ships a standalone derived document icon
without an app badge:

- `docs/design-system/markdown-file-icon.png` — source artwork for the
  Markdown document icon
- `build/windows/markdown.ico` — committed ICO embedded by the Windows build

At startup, the Windows association code writes the embedded ICO to
`%APPDATA%\Maakdown\markdown.ico` and registers `Maakdown.md\DefaultIcon` to
that per-user file. The main application icon remains `build/windows/icon.ico`.

### Full Update Checklist

1. Update the masters in `docs/design-system/maakdown.icon/Assets/` (and `icon.json` if layers change).
2. Regenerate title bar icons: `sips -z 64 64 ...` (see above).
3. Copy the light master to `build/appicon.png` (see above).
4. If the visual language changes, regenerate `docs/design-system/markdown-file-icon.png` and `build/windows/markdown.ico`.
5. Run `wails build -platform darwin/arm64`.
6. Run `scripts/postbuild-darwin.sh`.
7. Optionally `killall Dock; killall Finder` to clear macOS icon cache.

## Current Repo Expectations

- `docs/markdown-viewer-design-spec.md` and `docs/markdown-viewer-implementation-plan.md` are the current source of product and architecture truth.
- Wails v2.12.x is pinned for v1; do not migrate to Wails v3 unless the user explicitly reopens that decision.
- Svelte 5.x and Vite 8.x are the frontend baseline.
- Local Markdown images use the tokenized loopback asset server, not Wails v2 dynamic `AssetsHandler`.
- Generated Wails bindings under `frontend/wailsjs/` must be treated as generated. Application code should import through `frontend/src/ipc/`.

---
> Source: [cybermaak/maakdown](https://github.com/cybermaak/maakdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
