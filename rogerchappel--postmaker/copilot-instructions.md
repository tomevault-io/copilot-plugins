## postmaker

> Use this skill when an agent needs to turn a local repo, product brief, or changelog into social draft material with explicit claim review.

# postmaker Skill

## When To Use

Use this skill when an agent needs to turn a local repo, product brief, or changelog into social draft material with explicit claim review.

## Required Inputs

- A local source directory.
- One or more requested platforms.
- A local output directory.

## Side-Effect Boundaries

The skill reads local evidence files and writes local Markdown/JSON drafts. It does not post, schedule, scrape profiles, or call social APIs.

## Approval Requirements

Ask for explicit approval before publishing, scheduling, using private profile context, or calling any external platform API.

## Workflow

```bash
postmaker from-repo ./repo --platform linkedin --platform x --out posts
postmaker check posts/post-pack.json --source ./repo
```

Review `posts/launch.md` and the claim list in `post-pack.json`. Rewrite or remove any `needs-review` claim before publication.

## Validation

Run `npm test`, `npm run check`, and `npm run smoke`. The checker validates schema shape, evidence paths, claim statuses, and platform length limits.

---
> Source: [rogerchappel/postmaker](https://github.com/rogerchappel/postmaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
