## my-paper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a LaTeX master's thesis titled **"LLM-Driven Algorithmic Trading: A Reinforcement Learning Fine-Tuning Framework"** by Adham Helmy, supervised by Prof. Mohamed Ashour and Prof. Ayman El Badawy.

## Build Commands

```bash
# Compile the thesis (recommended: latexmk for automatic re-runs)
latexmk -pdf thesis.tex

# Or compile manually with pdflatex (may need multiple runs for TOC/refs)
pdflatex thesis.tex
bibtex thesis
pdflatex thesis.tex
pdflatex thesis.tex

# Clean build artifacts (keep .pdf)
latexmk -c

# Full clean including .pdf
latexmk -C
```

## Structure

- `thesis.tex` — single-file document containing all chapters
- `references.bib` — BibTeX bibliography (IEEE-style numbered citations via `\cite{}`)
- `figures/` — image assets (PNG format)

## LaTeX Conventions

- Citation style: IEEE numbered (`\cite{key}`), using the `cite` package
- Cross-references: use `\cref{}` (from `cleveref`) rather than `\ref{}`
- Theorem environments defined: `theorem`, `lemma`, `definition`, `proposition` — numbered per chapter
- Figures go in `figures/` and are included via `\includegraphics`
- Pseudocode uses `algorithm` + `algpseudocode` packages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adhamhelmy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
