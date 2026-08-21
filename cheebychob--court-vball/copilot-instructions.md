## court-vball

> - Every code change that affects app behavior, UI, data handling, scheduling, ratings, exports, backups, or tests must update the app version.

# Court project instructions

## Versioning Policy

- Every code change that affects app behavior, UI, data handling, scheduling, ratings, exports, backups, or tests must update the app version.
- Bug fixes increment the patch version.
- New backward-compatible features increment the minor version.
- Breaking data/schema or incompatible changes increment the major version.
- Documentation-only changes do not require an app version bump unless they alter in-app help text.
- `APP_INFO` in `index.html` is the single source of truth.
- Update both `APP_INFO.version` and `APP_INFO.build`.
- Build format is `YYYYMMDD.N`, using the current local date and a sequence number for multiple builds on the same day.
- The final implementation summary must state the old version and new version.
- Do not commit automatically.

## Court project principles

- Preserve historical rating integrity.
- Keep mobile usability strong.
- Avoid mysterious rating changes.
- Add tests with behavior changes.
- Prefer small, reviewable changes.
- Keep the single-file app unless intentionally split.
- Do not add unnecessary dependencies.

# Court Agent Instructions

Read `CLAUDE.md` for the project's architecture, safety rules, testing
expectations, and application conventions.

For event UI work, also read `docs/EVENT_UI_ROADMAP.md`.

Implement only the roadmap item assigned in the current prompt. Do not combine
later roadmap items or perform unrelated cleanup. Update the roadmap entry in
the same branch as the implementation.

---
> Source: [cheebychob/court-vball](https://github.com/cheebychob/court-vball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
