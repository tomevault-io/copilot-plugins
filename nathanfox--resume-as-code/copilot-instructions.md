## resume-as-code

> - **Do not use long (em) dashes (`—`) in resume content or any generated document.**

# Project Instructions

## Writing style

- **Do not use long (em) dashes (`—`) in resume content or any generated document.**
  Use commas, colons, parentheses, or separate sentences instead. Em dashes read as
  "AI-generated" and the user does not want them.
- En dashes (`–`) are acceptable **only** for numeric and date ranges (e.g. `2018 – Present`,
  `30–45 minutes`). Do not use them as sentence punctuation.
- Prefer plain hyphens for compound modifiers (e.g. `cloud-native`, `offline-first`).
- Keep bullets concrete: metrics, scale, and outcomes over adjectives.

## Resume build

- DRY source: shared content in `src/partials/`, per-variant headline/summary in
  `src/headline/` and `src/summary/`, and a recipe per variant in `src/variants/<name>.txt`
  (ordered list of partials). Edit a partial once; all variants update.
- Build with `./build.sh` (all variants) or `./build.sh <variant>` → MD + PDF + DOCX in
  `dist/<variant>/` (gitignored).
- The build warns about orphaned source files (not referenced by any recipe) and PDFs
  over `MAX_PAGES` pages (default 2). Do not ignore these warnings: fix the recipe,
  trim content, or delete the orphan.

## Workflow conventions

- **Application log:** whenever a resume is sent anywhere, add a row to
  `applications.md` (date, employer, role, variant, commit SHA). For significant
  applications also tag: `git tag applied/<employer>-<yyyy-mm>`. This is how the exact
  document a recruiter is holding can be rebuilt months later.
- **Postings:** when creating or re-tuning a variant for a job posting or audience,
  save the posting text or audience notes to `postings/<variant>.md` (see
  `postings/README.md`). Read that file first before re-tuning an existing variant.
- **Per-variant role framing:** a role can be split into head/body partials so variants
  can frame it differently (e.g. `role-acme-head.md` vs `role-acme-head-manager.md`,
  sharing `role-acme-body.md`). If several roles need this, move to a folder per role
  (`src/partials/role-<name>/head-<variant>.md`) instead of multiplying flat files.
- **Examples:** if the sample content or stylesheet changes, rebuild and refresh the
  committed PDFs in `examples/` so they match the source.

---
> Source: [nathanfox/resume-as-code](https://github.com/nathanfox/resume-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
