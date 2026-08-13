## aaronjgirard-github-io

> Two files must stay in sync:

# Project notes for Claude

## Publications -> collaborator-map sync

Two files must stay in sync:
- `publications.qmd` — each bibliography entry has `[]{#pubid}` anchor at the start.
- `collaborators.qmd` — each pubid must appear as a key in the `PUBLICATIONS` dict
  and in the `papers: [...]` array of every coauthor on that paper.

`scripts/check_publications.py` runs on every `quarto render` (wired as
`pre-render` in `_quarto.yml`) and fails the render if anything is out of sync.

### When the script reports a NEW PAPER

1. `WebFetch` the paper's URL listed by the script.
2. From the returned HTML, extract the full author list with affiliations.
3. For each author, normalize to `lastname_firstinitial` and match against
   existing entries in `COAUTHORS` (collaborators.qmd, line ~158 onward).
   - **Matched author**: append the new pubid to that person's `papers` array.
     Do NOT create a second entry for the same person.
   - **Unmatched author**: stop and tell the user. New people need their
     institution mapped to an `INSTITUTIONS` key (or a new institution added),
     which requires human judgment.
4. Add a new `PUBLICATIONS` entry with `year` and a short `label`.
5. If the link is a PDF / Google Drive / `files/` entry, the script will flag
   it as unfetchable — tell the user the authors+affiliations must be entered
   manually, do not guess.

### Author name matching

Names appear in many forms ("Jeffrey Shragge", "J. Shragge", "Shragge, J.").
Match conservatively on `lastname + first initial`. If two existing entries
would both match the same name (e.g., two people with surname "Nakata"),
ask the user which one — never guess.

### Publication ordering

See `~/.claude/projects/-Users-agirard-Dropbox-quarto-website/memory/feedback_publications_ordering.md`
for the convention (in-review at top of journal list; newly accepted papers
go to the top of the published block).

---
> Source: [aaronjgirard/aaronjgirard.github.io](https://github.com/aaronjgirard/aaronjgirard.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
