## context-window-exceeded

> - MUST make the smallest viable diff to satisfy the request.

# Change Policy

## Minimal Change Requirements

- MUST make the smallest viable diff to satisfy the request.
- MUST avoid drive-by changes (cleanup, reformatting, renames) not required by the task.
- MUST preserve existing file structure and Nuxt conventions unless explicitly asked.

## Refactors and Renames

- MUST ask before:
  - renaming files, components, or exported symbols
  - moving files or changing routes
  - reorganizing CSS or splitting/merging components
  - introducing new abstractions or shared utilities

## Dependencies and Lockfiles

- MUST NOT add, remove, or upgrade dependencies unless explicitly requested.
- MUST NOT hand-edit `package-lock.json`.
- If dependency changes are explicitly requested:
  - MUST update `package.json`.
  - SHOULD prefer regenerating `package-lock.json` via the package manager rather than manual edits.

## Conventions to Preserve

- MUST preserve Vue SFC structure and patterns already used:
  - `<script setup lang="ts">` where applicable
  - Nuxt auto-imports via `#imports` where already used
- MUST preserve existing accessibility patterns (e.g., skip link, ARIA labels) unless explicitly asked to change them.
- SHOULD prefer content edits in `content/` for copy changes that belong in Markdown-driven pages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/djmoore711) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
