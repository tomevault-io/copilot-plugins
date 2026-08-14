## community-forestry-zambia

> Quarto website project (report on Community Forest Management in Zambia).

# CLAUDE.md

Quarto website project (report on Community Forest Management in Zambia).
Full context, architecture, and current state: **[HANDOFF.md](HANDOFF.md)** —
read it before making changes.

## Before touching anything, know this:

1. **There is likely a large uncommitted diff sitting in the working tree.**
   Run `git status` first. Don't assume a clean tree, and don't commit broadly
   without reviewing what's actually staged — see HANDOFF.md §4 for what was
   in flight as of the last session.
2. **Chapter filenames contain spaces on purpose** (`CFMG ET.qmd`,
   `CFMG Survey.qmd`, etc. — renaming breaks published URLs). This triggers a
   known Quarto freeze bug where PDF figures get written to a hyphenated path
   but the freeze cache looks for them under the space-containing path,
   silently dropping figures from PDF renders. See HANDOFF.md §3 and
   `feedback_quarto_pdf_rendering.md` in memory for the fix procedure if this
   recurs after editing: CFMG ET, CFMG BS, CFMG Governance, CFMG SFM,
   CFMG Survey, or Remote Sensing.
3. **`PUBLICATION_STRATEGY.md` is the plan of record**, not just a note — it
   defines the target structure (navbar order, index.qmd layout, per-chapter
   conclusions boxes) and what phase the project is in. Check it before
   restructuring navigation, the index page, or chapter layout.
4. **Workflow: work directly on `main`, push directly — no branches or PRs**
   for this project. Milestones are marked with git tags (`Draft1.0`,
   `Draft2.0`, ...), not branches.

---
> Source: [rhettdharrison/community_forestry_zambia](https://github.com/rhettdharrison/community_forestry_zambia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
