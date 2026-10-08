## changelog

> Keep CHANGELOG.md Unreleased current; do not cut a versioned release unless asked


# Changelog policy

Follow [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Keep `## [Unreleased]` as the first version heading (immediately under the intro).

## Unreleased (default)

When completing **notable** work (user-facing features, fixes, behaviour changes, docs that users would notice, new examples/tools), **update `CHANGELOG.md` in the same change**:

- Add or extend bullets under `### Added`, `### Fixed`, `### Changed`, `### Removed`, or `### Notes for integrators` as they already appear in this file.
- Create a subsection only when it has at least one bullet; drop empty subsections.
- Merge into an existing bullet when it is the same change; do not duplicate.
- Skip trivial noise: formatting-only, ruff/lint, test-only internals, comment-only, or chore commits with no user-visible effect.
- **Do not** invent a versioned release (`## [x.y.z]`), dates, or version-bump prose while working under Unreleased.
- **Do not** bump `src/pyopticfilm/_version.py` unless the user asks for a version or release prep.

## Versioned releases (only when asked)

When the user **explicitly** asks to cut a release or write release notes:

- Move Unreleased items into a new `## [x.y.z] - YYYY-MM-DD` block **above** older versions.
- Leave `## [Unreleased]` in place (empty, no leftover subsections).
- Match existing heading names and Keep a Changelog style.

## Contributor credits

Credit **external** contributors where it is appropriate (merged PRs, hardware enablement, substantial patches).

- **Do not** credit the maintainer **jboneng**.
- Put credits under `### Contributors` in that **release** block (not Unreleased, unless the user asks).
- Format each name as a markdown link to that person’s GitHub profile: `[@username](https://github.com/username)`.

```markdown
### Contributors

- [@TobbyTravel](https://github.com/TobbyTravel) for OpticFilm 8100 (V2) support.
```

---
> Source: [jboneng/pyopticfilm](https://github.com/jboneng/pyopticfilm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
