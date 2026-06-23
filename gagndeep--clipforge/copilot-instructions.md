## clipforge

> You are an autonomous coding agent (Jules) working in this repo. Follow these rules:

# Agent instructions for this repository

You are an autonomous coding agent (Jules) working in this repo. Follow these rules:

## Working style
- Complete the ENTIRE task described in your prompt before opening a PR. Do not stop halfway to ask a question unless something is truly impossible — make reasonable decisions yourself and note them in the PR description.
- Never ask for clarification on visual/styling choices: pick the best modern option and proceed.
- Keep commits focused; write a clear PR description summarizing what was built and any decisions made.

## Quality bar
- Production quality only: no placeholder text like "lorem ipsum", no TODO stubs, no broken links, no console errors.
- HTML/CSS must be responsive (mobile-first) and content must never overflow the viewport. Test narrow widths mentally: long words wrap, tables scroll, slides fit.
- All pages share the repo's design system (CSS variables, fonts, spacing) — never inline one-off styles that drift from it.
- Validate any JSON you produce; keep data files parseable.

## Scope discipline
- Touch ONLY the files your task requires. Other agents may be working on sibling directories in parallel sessions — never refactor, rename, or "clean up" files outside your assigned scope.
- Do not modify this AGENTS.md, CI workflows, or other tasks' content directories unless your prompt explicitly says so.

---
> Source: [GagnDeep/clipforge](https://github.com/GagnDeep/clipforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
