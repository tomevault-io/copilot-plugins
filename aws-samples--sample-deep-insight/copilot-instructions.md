## sample-deep-insight

> - All filenames lowercase-kebab: `foo-bar.md`, never `FooBar.md` or `20260411_FooBar.md`.

# Documentation Conventions

## Naming

- All filenames lowercase-kebab: `foo-bar.md`, never `FooBar.md` or `20260411_FooBar.md`.
- Date prefix `YYYY-MM-DD-` only on incidents and other time-bound posts.
- Numbered prefixes (`01-`, `02-`) only inside multi-doc plans that should be read in order. Don't number standalone docs.
- The leading underscore in `_template` marks a folder as "not a real feature" and pins it to the top of `ls` output.

## Folder vs. flat

- A **feature** is always a folder: `features/<slug>/`.
  - Simple feature: one file at `<slug>/README.md`.
  - Multi-stage feature: `README.md` plus chapter files (`01-plan.md`, `02-research.md`, `03-design.md`, `04-implementation.md`). Add chapters as the design grows; don't pre-create chapters you don't need.
- An **incident** is always a folder: `incidents/YYYY-MM-DD-<slug>/incident.md`.
- **Process** docs are flat: `process/<slug>.md`.

## Optional frontmatter

YAML frontmatter at the top of a file:

```yaml
---
title: Skill System
status: active        # draft | active | superseded | deprecated
last_updated: 2026-01-15
---
```

`status` is informational; lifecycle changes update the field, not the filename.

## Features bucket — when to regroup

Today: flat under `features/<slug>/`.

**When `features/` has more than 12 direct children, regroup into area sub-folders** using `git mv` only (no rename of slugs):

```
features/
├── agent/    LLM / orchestration concerns, cross-deployment.
├── web/      deep-insight-web concerns.
├── ops/      Admin / observability concerns.
└── infra/    Infrastructure concerns.
```

Pick one of those four buckets. If a feature genuinely needs a new bucket, propose it in the same PR that adds the feature.

## Sensitive data in docs

Don't paste raw runtime ARNs, AWS account IDs, S3 bucket names, internal IPs, customer brand references, or upload / session IDs into committed files. Sanitize before staging. Incident transcripts especially: redact ARNs, account IDs, and customer-identifying strings.

---
> Source: [aws-samples/sample-deep-insight](https://github.com/aws-samples/sample-deep-insight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
