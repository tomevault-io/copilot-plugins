## latex-paper-backup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a LaTeX academic paper repository for writing research papers in machine learning and computer vision (specifically medical image segmentation with RWKV-based models). The repository supports Chinese and English content using XeLaTeX.

## Build Commands

### Compile LaTeX
```bash
# XeLaTeX (primary, supports Chinese)
xelatex -output-directory=out -synctex=1 -interaction=nonstopmode -file-line-error src/paper.tex

# PDFLaTeX (English-only)
pdflatex -output-directory=out -synctex=1 -interaction=nonstopmode -file-line-error src/paper.tex

# Full build with bibliography
xelatex -output-directory=out src/paper.tex && \
bibtex out/paper && \
xelatex -output-directory=out src/paper.tex && \
xelatex -output-directory=out src/paper.tex

# Using latexmk (recommended for full builds)
latexmk -xelatex -output-directory=out src/paper.tex
```

### Clean build artifacts
```bash
latexmk -c -output-directory=out
```

## Directory Layout

```
TexDocuments/
├── src/           # LaTeX source files
│   ├── paper.tex  # Main research paper
│   ├── resnet.tex # Technical report
│   └── refs.bib   # Bibliography
├── out/           # Compiled PDFs and aux files
└── pic/           # Figures (pdf/png/eps)
```

## Key LaTeX Tools Configuration

The project uses VS Code with LaTeX Workshop extension. Key settings:
- Output directory: `out/`
- Primary compiler: XeLaTeX (supports Chinese via `ctex`)
- Secondary: PDFLaTeX for English-only docs
- Recipes include full bibliography builds (run BibTeX twice between LaTeX passes)

## Paper Templates

### Conference Paper (`src/paper.tex`)
- Class: `IEEEtran` with `conference` option
- Packages: `ctex` (Chinese), `amsmath/amssymb/amsthm` (math), `graphicx` (figures), `algorithm/algpseudocode` (pseudocode)
- References: Uses `refs.bib` with IEEEtran style

### Technical Report (`src/resnet.tex`)
- Class: `article` for detailed reports
- Packages: `xeCJK` (Chinese), `listings` (code blocks), `geometry` (page layout)

## Tips for Working with LaTeX Files

1. **Chinese Content**: Use XeLaTeX as the compiler for files with Chinese characters (both main files use `ctex`/`xeCJK`)

2. **Image Paths**: Figures in `pic/` are referenced with relative paths like `../pic/filename.png`

3. **Bibliography**: References are in `src/refs.bib` and cited using `\cite{key}` format. Run BibTeX after first LaTeX pass to resolve citations

4. **Output**: All build artifacts (PDF, aux, log, etc.) go to `out/`, keeping the `src/` directory clean

5. **VS Code Auto-build**: LaTeX Workshop is configured to auto-build on save (`latex-workshop.latex.autoBuild.run: "onSave"`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kyochilian) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
