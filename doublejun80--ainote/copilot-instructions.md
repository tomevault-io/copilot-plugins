## ainote

> Use this repository to write, review, and publish a beginner-friendly Korean AI/development book.

# AI Note Repository

Use this repository to write, review, and publish a beginner-friendly Korean AI/development book.

## First Stops

- Read [vision.md](/C:/Users/05507/Documents/Github/AINOTE/docs/vision.md) for product intent.
- Read [audience.md](/C:/Users/05507/Documents/Github/AINOTE/docs/audience.md) before drafting.
- Read [style-guide.md](/C:/Users/05507/Documents/Github/AINOTE/docs/style-guide.md) for tone and section order.
- Read [quality-bar.md](/C:/Users/05507/Documents/Github/AINOTE/docs/quality-bar.md) before moving any section to `reviewed`, `verified`, or `final`.
- Read [source-index.md](/C:/Users/05507/Documents/Github/AINOTE/docs/source-index.md) when collecting or checking references.
- Read [execution-plan.md](/C:/Users/05507/Documents/Github/AINOTE/docs/execution-plan.md) for the operating workflow.

## Repository Map

- [manuscript/AGENTS.md](/C:/Users/05507/Documents/Github/AINOTE/manuscript/AGENTS.md): Writing instructions for the book files.
- [docs/glossary.md](/C:/Users/05507/Documents/Github/AINOTE/docs/glossary.md): Canonical editorial definitions.
- [skills/source-research/SKILL.md](/C:/Users/05507/Documents/Github/AINOTE/skills/source-research/SKILL.md): Source collection and verification workflow.
- [skills/section-drafter/SKILL.md](/C:/Users/05507/Documents/Github/AINOTE/skills/section-drafter/SKILL.md): Drafting workflow for one section.
- [skills/tone-review/SKILL.md](/C:/Users/05507/Documents/Github/AINOTE/skills/tone-review/SKILL.md): Beginner-tone review workflow.
- [skills/book-gardener/SKILL.md](/C:/Users/05507/Documents/Github/AINOTE/skills/book-gardener/SKILL.md): Cleanup and doc-gardening workflow.

## Commands

- Sync outline and create missing manuscript files: `node tools/sync-manuscript.mjs`
- Check outline drift only: `node tools/sync-manuscript.mjs --check`
- Validate manuscript structure: `node tools/validate-manuscript.mjs`
- Check internal links: `node tools/check-links.mjs`
- Generate heuristic review JSON: `node tools/review-manuscript.mjs`
- Render the book locally when Quarto is installed: `quarto render`

## Working Rules

- Keep `AGENTS.md` short; store detailed rules in `docs/` and skills.
- Treat `docs/` as the system of record for editorial policy.
- Treat each `manuscript/chXX/section-YY.md` file as the smallest drafting unit.
- Keep one change scoped to 1 to 3 sections when possible.
- Preserve existing content when syncing; only create missing files automatically.
- Prefer official and primary sources for technical verification.

---
> Source: [doublejun80/ainote](https://github.com/doublejun80/ainote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
