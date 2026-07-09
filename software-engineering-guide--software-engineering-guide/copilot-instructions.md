## software-engineering-guide

> Guidance for AI agents and human contributors working in this repository. Read

# AGENTS

Guidance for AI agents and human contributors working in this repository. Read
this first. It is short on purpose; the details live in the linked files.

## What this repo is

An open guidebook of software engineering best practices for large developer
teams, including enterprise and government. It is 100 chapters across 12 parts,
plus front matter and appendices. The writing is deliberately warm, plain, and
opinionated, and it follows a strict house style. The book is published as a
static site built with Zensical (see `SITE.md`).

## Golden rules (do not break these)

1. **No em-dashes.** Never use "—" (U+2014). Use a comma, colon, parentheses, or
   two sentences. En-dashes "–" are allowed only in numeric ranges.
2. **No stock LLM phrasing.** No "not only ... but also", "but also", or
   "load-bearing"; no "It's important to note", "In today's fast-paced world",
   and similar filler.
3. **Follow the chapter template.** Content chapters use the fixed section order
   in [`docs/contributing/chapter-template.md`](docs/contributing/chapter-template.md).
4. **Define terms on first use** and **link key concepts to Wikipedia** on first
   mention. Real references only; never fabricate a work or a URL.
5. **The spec is the source of truth.** Structure is declared in
   [`docs/spec/structure.md`](docs/spec/structure.md); style is declared in
   [`docs/spec/conventions.md`](docs/spec/conventions.md). Change the spec and
   the chapters together.
6. **Tests must pass.** Run `just test` before you consider a change done.

The full, enforceable version of rules 1 through 5 is
[`docs/contributing/style-rules.md`](docs/contributing/style-rules.md) and
[`docs/spec/conventions.md`](docs/spec/conventions.md).

## Repository layout

- `docs/` : everything the published site contains (built with Zensical).
  - `docs/chapters/` : the 100 chapter files, named `N.M-slug.md` (flat, decimal-numbered).
  - `docs/front-matter/` : the opening essay, introduction, and table of contents.
  - `docs/spec/` : the specification-driven source of truth (structure, conventions, roadmap).
  - `docs/examples/` : small illustrative examples (a chapter skeleton, OKRs, an ADR).
  - `docs/contributing/` : contributor and agent guides, plus shared snippets.
  - `docs/project/` : project documentation and the changelog.
- `tools/` : `gen_nav.py`, which generates the README TOC, the site home page,
  the contents page, the subject index, the `nav` block in `zensical.toml`,
  and the chapter map in `guide_xref/chapters.py`.
- `guide_xref/` : the Markdown extension that auto-links plain-text chapter
  cross-references ("chapter 8.1") when the site is built.
- `tests/` : `validate.py`, the enforcement suite.
- `zensical.toml`, `justfile`, `pyproject.toml`, `SITE.md` : site configuration,
  task runner, dependencies, and the site build guide.

## Task guides

- Writing or editing a chapter: [`docs/contributing/authoring.md`](docs/contributing/authoring.md)
- Regenerating navigation after structure changes: [`docs/contributing/navigation.md`](docs/contributing/navigation.md)
- Running and understanding the tests: [`docs/contributing/testing.md`](docs/contributing/testing.md)

## Shared snippets

- Blank chapter template: [`docs/contributing/chapter-template.md`](docs/contributing/chapter-template.md)
- Style rules (enforceable): [`docs/contributing/style-rules.md`](docs/contributing/style-rules.md)
- Part index: [`docs/contributing/part-index.md`](docs/contributing/part-index.md)

## The usual workflow

1. Read the relevant task guide above.
2. Make the smallest change that satisfies the request.
3. If you changed the set of chapters, update `docs/spec/structure.md` and run
   `just nav`.
4. Run `just test`. Fix anything it reports.
5. Update `docs/project/changelog.md` with a one-line summary of what changed.

Every file in `docs/contributing/` is kept small (well under 40 KB) so it loads
cheaply into an agent's context.

---
> Source: [software-engineering-guide/software-engineering-guide](https://github.com/software-engineering-guide/software-engineering-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
