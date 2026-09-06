## slim-select

> Slim Select project conventions (releases, docs, and shared agent guidance)


# Slim Select

Project-wide conventions for agent work in this repo. Add new sections here as needed.

## Changelog

When shipping a version bump or finishing user-facing work that will go into a release, update `CHANGELOG.md` in the same change set (do not leave it for later).

Follow Keep a Changelog style already used in the file:

- Add or extend the section for the version being released (e.g. `## [4.2.0]`)
- Use `### Added`, `### Changed`, `### Fixed`, or `### Removed` as appropriate
- Prefer concise bullets; link related GitHub issues when known (`[#707](...)`)
- Skip pure chore/noise (dependency bumps, dist rebuilds) unless they affect consumers

Before considering a release or version commit done, confirm `CHANGELOG.md` mentions the notable changes.

---
> Source: [brianvoe/slim-select](https://github.com/brianvoe/slim-select) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
