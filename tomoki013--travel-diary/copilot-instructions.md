## travel-diary

> This file provides priority mandates for Gemini CLI when working on this repository.

# GEMINI.md

This file provides priority mandates for Gemini CLI when working on this repository.

## Blogging Mandates

The following rules are foundational for all blog post creation, editing, and formatting tasks.

### Rule Source
- **Primary Rules:** `draft-posts/rules/*.md`
- Gemini MUST always check and adhere to the latest instructions in `draft-posts/rules/` before creating or editing any blog posts.
- **Specific Guides:**
  - `EDITORIAL_BASELINE.md`: Shared editorial baseline for all articles.
  - `CONTENT_STRATEGY.md`: Category-specific intent and tone.
  - `TRAVEL_DIARY_RULES.md`: `series: travel-diary` specific structure and selection rules.
  - `OPERATIONS.md`: Frontmatter, naming, link placement, and publishing workflow.
  - `SOURCE_CONVERSION.md`: Rules for rewriting Note or other source text into blog posts.

### Core Principles
- **Style Consistency:** Match the existing patterns in the `posts/` directory.
- **Fixed Targets:** Always define the target file path before starting edits.
- **Structural Integrity:** Use proper Markdown headings (`##`, `###`), unified "Desu/Masu" tone, and chronological re-organization.
- **Noise Removal:** Rigorously remove all non-essential elements (Images, ToC, CTAs, external links).
- **Verification:** Double-check for residual noise keywords (URLs, "ToC", "images", etc.) before finalizing any post.

## Workflow Mandates
- When tasked with "cleaning up" or "writing" a blog post, Gemini must first read the rules in `draft-posts/rules/` to ensure full compliance.

## Documentation Maintenance
- Shared development and operations docs live in `docs/` and are the source of truth for repository workflow and structure.
- When implementation changes architecture, workflow, page responsibilities, or content operations, Gemini must update the relevant files in `docs/` in the same change.
- If the change affects AI behavior or working rules, Gemini must also update `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md` together in the same change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomoki013) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
