## opsnerva

> - Never place build caches, test caches, package-manager caches, compiler work directories, or other temporary files inside this repository.

# Repository Agent Instructions

## Temporary files and caches

- Never place build caches, test caches, package-manager caches, compiler work directories, or other temporary files inside this repository.
- Use each tool's default cache directory when it is outside this repository; do not override it without a concrete isolation or reproducibility need.
- Put task-owned temporary artifacts that need an explicit location in a task-specific directory under the system temporary directory, and clean them after the task.
- Repository-managed build outputs such as `web/dist` are product artifacts, not caches.

## Change preparation

- Before modifying code, collect enough information to identify the root cause, affected paths, existing behavior, and relevant constraints.
- Inspect the surrounding implementation and current changes before editing; do not implement a fix from the visible symptom alone.

## Implementation quality

- Do not add defensive compatibility paths, fallback branches, adapters, or duplicate implementations without a demonstrated current requirement. Prefer removing obsolete behavior over preserving it speculatively.
- Before accumulating localized patches, evaluate whether a focused simplification or refactor produces a smaller and clearer implementation. Update or remove superseded code, tests, documentation, locale keys, and styles as part of the same change.
- Optimize selectively from collected evidence. Keep changes cohesive and maintainable instead of pursuing the requested outcome through layered special cases.
- Organize code into cohesive files by module or feature responsibility. Do not accumulate unrelated components, handlers, state management, and protocol logic in one oversized, tightly coupled file.
- When refactoring a touched area, split existing oversized or coupled files along clear responsibility boundaries when that improves cohesion; avoid arbitrary fragmentation into trivial files.

## Frontend performance

- Treat frontend runtime performance as an implementation-quality requirement. Inspect update frequency, render scope, lifecycle, and idle resource use before adding timers, subscriptions, animations, or streaming UI updates.
- Do not use aggressive polling when an event-driven or server-push path is available. When polling is required, use a justified interval and bounded lifecycle, and stop it when the view is inactive, hidden, unmounted, or the operation reaches a terminal state.
- Do not leave loading indicators or decorative animations running indefinitely. Tie animation lifetime to real activity, stop it when hidden or complete, and avoid continuously repainting expensive properties.
- Coalesce or throttle high-frequency progress and streaming updates, keep rapidly changing state close to its consumers, and avoid rerendering entire pages or long lists for localized changes.

## Frontend copy

- Keep frontend text concise and limited to labels, values, statuses, actionable errors, and instructions required to complete an operation.
- Do not add explanatory, promotional, repetitive, or self-evident helper text.
- Confirmation dialogs contain only a title, required fields, action buttons, and actionable errors. Do not add eyebrow labels, explanatory body copy, repeated consequences, or self-evident warnings.
- When removing UI text, also delete its unused locale keys, markup, and styles.

## Desktop builds

- Never compile, test, or build Rust/Tauri locally.
- Validate desktop changes through source review and GitHub Actions. Local verification is limited to Go tests and the Web build.

---
> Source: [Enterpr1se0/OpsNerva](https://github.com/Enterpr1se0/OpsNerva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
