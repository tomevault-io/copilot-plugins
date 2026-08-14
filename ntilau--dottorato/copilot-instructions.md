## dottorato

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

LaTeX sources for a PhD dissertation ("Contributions to the Art of Finite Element Analysis in Electromagnetics", University of Florence, 2011-2013) and its defense presentation. The repository contains the complete manuscript, Beamer slides, related drafts, archives of published work, and notes.

Build instructions and dependency setup are documented in README.md. Run `./configure` to install LaTeX toolchain dependencies, then `make` to build both documents.

## Build commands

```
make                  # Build manuscript + presentation
make manuscript       # Build only manuscript/dissertation.pdf
make presentation     # Build only presentation/defense.pdf
make clean            # Remove LaTeX temp files (keep PDFs)
make distclean        # Remove temp files and PDFs
make help             # Show all targets and variables
```

Override defaults: `make MS_BIB=biber manuscript` (use biber for bibliography) or `make MS_FILE=mainThesis` (different source file).

## Document structure

### Manuscript (`manuscript/`)

The entry point is `dissertation.tex` — a `\documentclass{book}` that pulls in chapters via `\include{}`:

- `abs.tex` — Italian abstract (Sommario) + English abstract
- `ack.tex` — Acknowledgments
- `title.tex` — Title page
- `pub.tex` — List of publications
- `intro.tex` — Introduction
- `ch1.tex` — Chapter 1: Finite elements for the wave equation
- `ch2.tex` — Chapter 2: Domain decomposition methods for large problems
- `ch3.tex` — Chapter 3: Nonlinear analysis of passive microwave devices (harmonic balance FE)
- `concl.tex` — Conclusion
- `dissertation.bib` — Bibliography (BibLaTeX, IEEE style, biber backend)

All custom macros (math commands like `\vect{}`, `\dyad{}`, `\rot`, `\div`, field vectors `\vE`, `\vH`, etc.) are defined in the preamble of `dissertation.tex`. Editing commands `\add{}` (blue text) and `\remove{}` (red strikethrough) are available for corrections.

Images live under `manuscript/img/` with chapter-specific subdirectories (`img/ch1/`, `img/ch2/`, `img/ch3/`). Each chapter sets its own `\graphicspath{{img/chN/}}`.

The generated PDF is at `manuscript/dissertation.pdf`.

### Presentation (`presentation/`)

Beamer slides for the PhD defense. Entry point is `defense.tex`; theme and package config is in `custom.tex`. Images under `presentation/img/`. Uses `-shell-escape` flag during compilation.

### Other directories

- `drafts/` — Earlier work: MEng thesis, conference papers, journal articles. Organized by year/topic.
- `archives/` — ZIP bundles of published/submitted papers and presentations.
- `notes/` — Reference document (`ln_Thesis_Dottorato.pdf`).

## Editing tips

- Each `.tex` file begins with `% !TEX root = ln_diss.tex` (or equivalent) for editor LaTeX tooling, but the actual entry point is `dissertation.tex`.
- When adding a new chapter, include it in `dissertation.tex` with `\include{filename}` (no `.tex` extension) and add the corresponding `\graphicspath`, `\cleardoublepage`, etc.
- New bibliography entries go in `manuscript/dissertation.bib`.
- The bib backend is `biber` (not bibtex). The style is `ieee`.

---
> Source: [ntilau/Dottorato](https://github.com/ntilau/Dottorato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
