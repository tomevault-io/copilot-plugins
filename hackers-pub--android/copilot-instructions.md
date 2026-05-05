## android

> This repository keeps its project guidance in two documents:

# AGENTS

This repository keeps its project guidance in two documents:

- Read [README.md](./README.md) first for product scope, build steps, and required checks.
- Read [CONVENTION.md](./CONVENTION.md) for reviewer-enforced coding rules under `app/src/main/`.

Minimum expectations for any change:

- Keep changes consistent with `README.md` and `CONVENTION.md`.
- Run `./gradlew :app:lintDebug` before pushing.
- Do not introduce new `!!` in production code.
- For paged post feeds, keep using the existing Paging overlay and deduplication patterns.
- When resolving merge conflicts, fast-forward local `main` from `origin/main` first (otherwise the merge silently uses a stale base), then after the merge inspect files touched by both branches against `main` directly — git's 3-way auto-merge can drop one side's hunks without surfacing a conflict marker.

---
> Source: [hackers-pub/android](https://github.com/hackers-pub/android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
