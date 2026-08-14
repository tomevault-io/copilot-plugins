## tfy4220-companion

> This is a **course study guide**. It pins the `study-companion` framework via a

# Course repo — agent brief

This is a **course study guide**. It pins the `study-companion` framework via a
git tag in `package.json` and contains **only content**.

## Hard rules

- **Author ONLY under `content/`** — `course.yaml`, `flashcards.yaml`, and
  `sections/*.mdx`. Static assets go in `public/` (figures, sims, PDFs, favicon).
- **Do NOT add components, pages, layouts, styles, or toolchain config.** The
  framework injects all of it. There is no `src/pages/` here.
- **To change the design or add a widget, change the FRAMEWORK repo** and bump
  the pinned tag here — never work around a missing feature with ad-hoc HTML.
- Keep everything **data-driven**: formulas, glossary, exams, and metadata live
  in `course.yaml`, not hard-coded in prose.

## Before you write content

**Read `AUTHORING.md` in the framework repo** (top level). It is the primary
brief: the mental model, the per-archetype section shapes, the widget decision
guide (including course-owned `<Simulation>` canvas sims and sim-driven
`<CodeBlock>` stepping), the conventions (KaTeX/YAML escaping, explicit
numbering, external references), and the **per-section definition-of-done**.

## Workflow

1. Fill `content/course.yaml` (identity, accent, `courseUrl`, exam, formulas,
   glossary).
2. Outline sections by `order` (and optional `part`), one file per module.
3. Draft each module against an archetype (see `AUTHORING.md` / the example
   sections in this template). Wire cross-refs: `<Term>`, `<FormulaRef>`,
   `<Statement>` ids.
4. Verify against the section definition-of-done.

## Run

```bash
pnpm install
pnpm dev      # preview (search needs build/preview)
pnpm build    # static output to dist/ — FAILS on any dead <Term>/<FormulaRef>
pnpm preview  # preview the build (search works here)
```

The build validates every cross-reference: a `<Term name>` / `<FormulaRef id>`
with no matching `course.yaml` entry, or a duplicate `<Statement>`/formula
anchor, fails `pnpm build` with a message naming the file and target.

---
> Source: [MartinSA04/TFY4220_companion](https://github.com/MartinSA04/TFY4220_companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
