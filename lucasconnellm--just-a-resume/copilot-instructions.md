## just-a-resume

> This repository is designed to be edited with an AI coding agent. Treat the

# Resume agent contract

This repository is designed to be edited with an AI coding agent. Treat the
career inventory as factual data, not creative-writing material.

## Non-negotiable rules

1. Never invent an employer, title, date, technology, degree, certification,
   metric, scope, or outcome.
2. Never change a factual claim merely to match a job description.
3. Ask for confirmation when source material conflicts or when a stronger
   bullet would require an unverified number.
4. Add confirmed evidence to `content/master.yaml` before selecting it in a
   variant.
5. Variants may select and reorder stable IDs. They must not smuggle in new
   claims through summaries, metadata, alt text, attachments, or hidden layers.
6. Metadata keywords must be derived from skills visible in the rendered
   variant.
7. Preserve a linear reading order, semantic headings and lists, and ordinary
   selectable Unicode text.
8. Prefer relevance cuts over smaller type. Do not reduce body text below
   9.5 pt or margins below 0.5 inches to force a one-page result.
9. Run `./scripts/build all` after changing content, selection logic, or layout.
10. Never upload a real private resume to a third-party parser without explicit
    permission.
11. Never add `pull_request`, `pull_request_target`, `merge_group`, or another
    externally triggerable Actions event without the repository owner's explicit
    approval. Public pull requests must not execute this repository's workflows.

## Editing workflow

- Put raw user material in `content/inbox.md` or an ignored
  `content/imports/` file.
- Normalize chronology and flag contradictions before polishing prose.
- Give every new entry and bullet a short, stable, descriptive ID.
- Write bullets as evidence: outcome or owned result, then action and context.
- Quantify only when the owner supplied or confirmed the number.
- Keep the master comprehensive; make each variant selective.
- Check the extracted `.txt` output, not just the PDF preview.

---
> Source: [lucasconnellm/just-a-resume](https://github.com/lucasconnellm/just-a-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
