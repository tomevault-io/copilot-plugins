## litprog-skill

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This is a **Claude Code custom skill** for literate programming. It is installed as a skill (via SKILL.md) and provides Claude Code with the ability to transform codebases into literate programs -- `.lit.md` files that weave prose, diagrams, and code into a human-readable narrative while also tangling back into the original source files.

This repo is NOT a typical application. It contains:
- `SKILL.md` -- the skill definition (the main artifact)
- `scripts/tangle.ts` -- the tangler that extracts source files from `.lit.md` files, emits a `.lit.map.json` source map
- `scripts/untangle.ts` -- reverse-sync engine that updates the `.lit.md` when source files change
- `scripts/hook-reverse-sync.ts` -- PostToolUse hook entry point for automatic reverse-sync
- `references/` -- supporting docs on chunk syntax, analysis workflow, and pandoc setup
- `assets/pandoc-header.yaml` -- LaTeX/PDF template for weave output

## Commands

```bash
# Tangle: extract source files from a literate program (also emits .lit.map.json)
bun run scripts/tangle.ts <file.lit.md> --output-dir <dir>

# Verify: compare tangled output against existing files without overwriting (exit 1 if any differ)
bun run scripts/tangle.ts <file.lit.md> --output-dir <dir> --verify

# Untangle: reverse-sync a changed source file back into the .lit.md
bun run scripts/untangle.ts <changed-file> [--lit-map <path>] [--weave]

# Weave: generate PDF from a literate program (requires pandoc + xelatex + mermaid-filter)
pandoc project.lit.md -o project.pdf --pdf-engine=xelatex --filter mermaid-filter --toc --number-sections -V geometry:margin=1.5cm
```

There are no tests, linter, or build steps for this repo itself.

## Architecture

The tangler (`scripts/tangle.ts`) works in three phases:
1. **Parse** -- regex-based extraction of fenced code blocks with `{chunk="name" file="path"}` attributes
2. **Build dictionary** -- same-named chunks are concatenated (additive chunks)
3. **Expand** -- recursive resolution of `<<chunk-ref>>` references with indent preservation, cycle detection, and unused-chunk warnings

Root chunks (those with `file=`) produce output files. Fragment chunks are only included transitively via `<<ref>>`.

### Reverse-sync flow

When a source file is edited directly:
1. The PostToolUse hook (`scripts/hook-reverse-sync.ts`) detects the edit
2. It checks the `.lit.map.json` source map to see if the file was tangled
3. If yes, it runs `scripts/untangle.ts` which splices changes back into the `.lit.md`
4. The untangler re-tangles to regenerate all source files and refresh the map
5. A `.lit.untangle.lock` file prevents infinite loops during re-tangling

## Key Conventions

- Chunk names use lowercase kebab-case: `parse-config`, `main-loop`
- Use `bun` (not `npm`/`node`) for all JS/TS execution
- The `.lit.md` format uses standard Markdown fenced code blocks extended with curly-brace attributes -- it is NOT a custom file format, just a convention on top of Markdown

---
> Source: [tlehman/litprog-skill](https://github.com/tlehman/litprog-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
