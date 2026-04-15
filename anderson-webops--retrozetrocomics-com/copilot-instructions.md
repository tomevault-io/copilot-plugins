## retrozetrocomics-com

> - Do not leave completed work uncommitted. After each coherent, validated change set, create a commit and push it in the same session.

# Workspace Instructions

## Lockfiles, Commits, Tags, And Releases
- Do not leave completed work uncommitted. After each coherent, validated change set, create a commit and push it in the same session.
- Use multiple commits and pushes when that keeps unrelated changes, partial validations, or follow-up fixes clearly separated. Prefer small, logically grouped commits over one mixed commit.
- Keep `package-lock.json` synchronized before every commit or push.
- Use lowercase annotated semver tags only. Do not invent ad-hoc labels such as `V1`, `torca-r07`, `pre-lfs-migration-*`, or similar one-off names.
- This repo follows the stable `v2.x` line. Stay on `v2` for routine work; only cut `v3` for an intentional breaking site or API change.
- Create an annotated tag when comic/catalog browsing, storefront-like behavior, content structure, dependency/security, or deploy/health behavior materially changes.
- Create a GitHub release when that tag represents a meaningful public-site or operations milestone. Release notes should summarize scope, validation, rollout notes, and any migration or recovery steps.
- If the existing tag or release history contains stale drafts, redundant entries, or ad-hoc labels, clean that history up instead of preserving clutter.
- Skip tags and releases for trivial doc-only edits, formatting-only changes, or routine housekeeping unless they change deployment, operations, or a consumer-facing contract.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anderson-webops)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anderson-webops)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
