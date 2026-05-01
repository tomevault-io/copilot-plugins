## plugin-maintainer-flow

> Maintainer flow for developing and releasing this plugin repository.


# Plugin Maintainer Flow

When changes affect this plugin repository:

1. **Before commit**, run `/review-plugin-submission`.
2. Fix all blocking findings from submission review.
3. If release-impacting, keep versions synchronized:
   - `CHANGELOG.md` new section
   - `.cursor-plugin/marketplace.json` -> `metadata.version`
4. Validate templates with:
   - `node scripts/validate-template.mjs`
5. Commit only after checks pass.

For publishing a release:

- Use `./scripts/release-plugin.sh <version>`.

---
> Source: [J-Pster/Psters_AI_Workflow](https://github.com/J-Pster/Psters_AI_Workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
