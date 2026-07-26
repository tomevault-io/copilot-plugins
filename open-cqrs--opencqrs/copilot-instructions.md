## opencqrs

> This repo doubles as the workshop for OpenCQRS blog articles. The full writing pipeline — dialogue, brainstorm, write, grill, enrich, prospect — is specified in:

# OpenCQRS — Claude Code Project Notes

## Article Pipeline

This repo doubles as the workshop for OpenCQRS blog articles. The full writing pipeline — dialogue, brainstorm, write, grill, enrich, prospect — is specified in:

**`.claude/article-pipeline.md`** — read this first when working on articles or modifying the article skills.

All intermediate artifacts (transcripts, briefs, enrichment notes, grill findings) live under `.article-work/{date}-{slug}/` and are gitignored. Published articles go to `mkdocs/docs/blog/posts/`.

## OpenCQRS Terminology

- Never use the term **"aggregate"** in articles, docs, or skill-generated content. OpenCQRS does not have aggregates. Use **"instance"** or **"state"** depending on context.

---
> Source: [open-cqrs/opencqrs](https://github.com/open-cqrs/opencqrs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
