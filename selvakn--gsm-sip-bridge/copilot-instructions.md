## release-notes

> Guidelines for writing release notes before tagging a new version


# Release Notes Process

`RELEASE_NOTES.md` is a cumulative changelog that carries historical release notes for all versions (newest first). Before creating a new git tag, a new section MUST be added at the top of the file.

## Format

```markdown
# Release Notes

## vX.Y.Z

- **Feature Name** -- One-sentence description of the feature and its user-facing value.
- **Another Feature** -- Description.

## vX.Y.W (previous release)

- ...
```

## Rules

1. Add a new `## vX.Y.Z` section at the top (below the `# Release Notes` heading) before tagging.
2. Each bullet must start with a bold feature name followed by a concise description.
3. Focus on user-facing changes: new capabilities, changed behavior, removed functionality.
4. Do NOT list internal refactors, CI changes, or dependency bumps unless they affect users.
5. For patch releases (x.y.Z), list bug fixes instead of features.
6. NEVER remove or edit older version sections -- the file is the historical record.
7. The publish workflow extracts the section matching the tagged version and includes it on the GitHub Release page.

## Workflow

1. As features merge to main, add bullets under a new `## vX.Y.Z` heading at the top.
2. Review and finalize the section before running `git tag vX.Y.Z`.
3. Push the tag -- CI extracts the matching section and publishes the release with those notes.

---
> Source: [selvakn/gsm-sip-bridge](https://github.com/selvakn/gsm-sip-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
