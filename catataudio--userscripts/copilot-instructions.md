## userscripts

> Use when we need to discover and evaluate additional skills for browser automation, publishing, testing, or repo workflows.

# AGENTS.md

This repository is the source workspace for refining, consolidating, and maintaining browser userscripts.

## Primary Goal

Optimize for:

- Tampermonkey compatibility
- maintainability
- safe refactors
- easy reuse across multiple browsers and machines

## Repository Conventions

- Keep installable scripts in `scripts/` as `*.user.js` whenever possible.
- Keep templates in `templates/`.
- Keep migration, sync, and operating notes in `docs/`.
- Move retired or superseded scripts into `archive/`.

## Script Standards

- Every installable script should have a complete Tampermonkey metadata block.
- Default to narrow `@match` rules rather than broad host permissions.
- Use `@grant none` unless Tampermonkey APIs are actually needed.
- Prefer `document-idle` unless the site requires earlier execution.
- Bump `@version` for every meaningful behavior change.
- Add comments only where they help future maintenance.

## Refactoring Rules

- Prefer small, behavior-preserving cleanups first.
- Consolidate duplicate utilities only when the result stays Tampermonkey-friendly.
- Use defensive DOM checks and idempotent init patterns.
- Avoid overengineering build steps unless the script count or complexity justifies it.

## Multi-Browser / Multi-Machine Bias

When choosing between approaches, prefer the one that makes distribution easier across:

- multiple Chromium browsers
- Firefox where practical
- multiple local browser profiles
- multiple machines

Preferred long-term distribution model:

1. Source lives in this Git repo.
2. Installable `*.user.js` files are the durable output.
3. Scripts are eventually hosted behind stable raw URLs for Tampermonkey auto-update.

## Expected Workflow

1. Ingest raw `.js` files from the user.
2. Normalize them into Tampermonkey-ready `*.user.js` files where needed.
3. Refine naming, metadata, and initialization timing.
4. Consolidate overlap across scripts when justified.
5. Keep docs current when the operating model changes.

## Audit Standard

Do not stop at file cleanup alone when the user wants help deciding what to keep.

Use the `userscripts-erudite` workflow as the default audit and consolidation model for this repository.

Required audit sequence:

1. Parse the metadata block.
   Record `@match`, `@include`, `@exclude`, `@grant`, `@require`, `@connect`, `@run-at`, `@noframes`, and update URLs.
2. Build a behavior map.
   For each script, capture `Trigger`, `Target`, `Action`, `Persistence`, and `External calls`.
3. Audit risk.
   Classify each script as `SAFE`, `CAUTION`, or `HIGH RISK`.
4. Compare scripts across the set.
   Call out overlap in domains, selectors, observers, timers, keyboard shortcuts, CSS injection, and storage keys.
5. Refactor only after the analysis is stable.
   Tighten scope, remove dead code, remove stale metadata, and keep outputs as plain `.user.js` files.

For each userscript under review, explicitly answer:

1. Where does it run?
   Record its `@match`, `@include`, `@exclude`, and any broad host scope.
2. What does it touch?
   Note DOM changes, storage, network requests, cookies, clipboard, downloads, redirects, auto-clicking, external libraries, and injected UI.
3. What permission footprint does it have?
   Record `@grant`, `@require`, `@connect`, `unsafeWindow`, `GM_xmlhttpRequest`, and any broad or risky metadata.
4. Is it still earning its keep?
   Classify it as `keep`, `review`, or `cut`.

## Review Heuristics

### Usually low risk

- CSS injection
- small DOM cleanup
- simple keyboard shortcuts
- modest site-specific UI polish

### Medium risk

- broad `@match` patterns
- heavy mutation observers
- many intervals or timers
- storage writes
- large multi-feature site overlays

### Higher risk

- `GM_xmlhttpRequest`
- external `fetch` or XHR to third-party hosts
- `@connect`
- third-party `@require`
- `unsafeWindow`
- cookie manipulation
- form interception
- automatic posting, clicking, redirects, or downloads

## Output Expectations For Reviews

When auditing userscripts, produce a compact table or equivalent summary with:

- script name
- runs on
- main behavior
- risk level
- keep / review / cut recommendation

If overlap exists, call it out explicitly instead of treating scripts independently.

Preferred review structure:

- `Summary`
- `Per-Script Analysis`
- `Security Audit`
- `Overlap and Conflict Map`
- `Consolidation Plan`
- `Refactored Script` when a merge or rewrite is actually performed

## Current Priority

For pruning work, prioritize:

1. enabled scripts
2. broadest-scope scripts
3. oldest scripts
4. scripts with unclear value or overlapping behavior

## Skills To Prefer

These local skills are the most relevant to this repository:

- `userscripts-erudite`
  Use as the primary skill for auditing, explaining, consolidating, and stabilizing userscripts.
- `find-skills`
  Use when we need to discover and evaluate additional skills for browser automation, publishing, testing, or repo workflows.
- `codex-iteration-sync`
  Use when this repo needs to be pushed, mirrored, or bootstrapped cleanly across other machines.
- `skill-creator`
  Use if this workspace grows into a repeatable userscript-maintenance workflow worth packaging as a custom skill.
- `frontend-design`
  Use when a userscript injects or redesigns visible UI and the interface quality matters.
- `web-design-guidelines`
  Use when reviewing injected UI for accessibility, clarity, and interaction quality.

## Current Bias

Do not introduce a bundler, build pipeline, or shared runtime until the repository actually needs one. Start with plain, inspectable userscripts.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/catataudio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
