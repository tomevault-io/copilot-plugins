## uglylatex

> A collection of LaTeX document classes (UglyNote, UglyPaper, UglyRep) for Pandoc-compatible typesetting.

# UglyLaTeX — AGENTS.md

A collection of LaTeX document classes (UglyNote, UglyPaper, UglyRep) for Pandoc-compatible typesetting.
**Author**: Githubonline1396529 | **License**: LPPL 1.3c (maintained)

## Repository Structure

```
uglylatex/
├── texmf/tex/latex/uglylatex/
│   ├── uglynote.cls / uglypaper.cls / uglyrep.cls  # Document classes
│   ├── classoptions.tex  # Kvoptions definitions (\input, not a package)
│   ├── dependencies.sty  # Central package loader (ctex, hyperref, biblatex…)
│   ├── docfmt.sty  # Fonts, lists, code blocks, tables, quotes
│   ├── docpapersize.sty  # 8 device sizes via geometry
│   ├── doccolor.sty  # ecolor: 6 ElegantLaTeX + 5 original colours
│   ├── doccaptions.sty  # Figure/table/subcaption bold ecolor style
│   ├── doclanguage.sty  # cn/en: theorem envs, section labels, \keywords
│   ├── docadmonitions.sty  # tcolorbox callout blocks
│   ├── docstem.sty  # Optional STEM: tikz, pgfplots, chemfig…
│   ├── commands.sty  # \subtitle, \fig, \maketitleabstracttwocol
│   ├── customized_abstract/  (abstractblock/runinabstract/abstractpage)
│   ├── customized_headers/  (noteheaders/paperheaders/reportheaders)
│   └── customized_toc/  (toclines/tocpage)
├── example/  # Entry .tex + mainbody-cn/ (18 chapters) + refs + figures
├── image/  # uglylatex.jpg showcase
├── Makefile / config.mk  # Build automation & TEXMF path
└── metadata.yaml  # Pandoc YAML header for README → PDF
```

## Document Classes

| Class | Base | Purpose | Abstract | Headers | ToC |
|---|---|---|---|---|---|
| `uglynote.cls` | `article` | Lightweight notes | Block-style, kaishu (cn) | Centered, `\arabic{section}` | Indented 0/2/4em |
| `uglypaper.cls` | `article` | Compact papers | Run-in `[runin]` | Left-aligned, same numbering | Indented 0/2/4em |
| `uglyrep.cls` | `report` | Long reports | Full-page, bold ecolor | Chapter-based, centred | Page-styled, tocdepth=3 |

**Common**: require `xelatex` (check commented out); chain: dependencies→docfmt→docpapersize→doccolor→doccaptions→doclanguage→docadmonitions→commands; `\pagestyle{fancy}`; appendix `\Alph{}` in ecolor; `\maketitle` colours title/author/date.

**UglyRep extras**: ToC patched `\apptocmd{\clearpage\markboth{}{}}` to fix header bleed; floats per chapter.

## Class Options (`classoptions.tex`, kvoptions family `UGLY`)

| Option | Type | Default | Values |
|---|---|---|---|
| `color` | string | `black` | elegant(green/cyan/blue/sakura/brown), blue, cyan, crimson, green, olive |
| `lang` | string | `cn` | cn, en |
| `usefa` | string | `true` | dousefa, notusefa |
| `device` | string | `normal` | pc, pad, kindle, normal, compact, screen, booklet, textbook |
| `stem` | bool | `false` | Loads docstem.sty (tikz, pgfplots, chemfig…) |
| `bib` | bool | `true` | Use `nobib` to disable biblatex |
| `fontsize` | string | `11pt` | 9pt–20pt |
| *(future)* | — | — | bgmode, chinesefont, citestyle, bibstyle, bibend, math (commented) |

Unknown → passed to base class via `\DeclareDefaultOption`.

## Colour Schemes (`doccolor.sty`)

- **ElegantLaTeX (6)**: black(0,0,0), elegantgreen(0,120,2), elegantcyan(31,186,190), elegantblue(1,126,218), elegantsakura(255,183,197), elegantbrown(109,62,18)
- **UglyLaTeX (5)**: blue(0,91,150), cyan(0,128,128), crimson(184,15,10), green(11,102,35), olive(128,128,0)
- **Future (commented)**: army, airforce, navy

## Build System

| Command | Action |
|---|---|
| `make install` | Copy texmf/ → `$(TEXMF)`, refresh FNDB |
| `make test` | Compile examples via `latexmk -xelatex --shell-escape` |
| `make example` | `test` → copy PDFs → `make doc` |
| `make doc` | Pandoc README.md → PDF (`xelatex`, metadata.yaml) |
| `make clear` | Remove AUX files and PDFs from example/ |

`config.mk`: `DISTRIBUTION` (texlive/miktex), `TEXMF` (default `/d/texlive/texmf-local`).

## Conventions

- **Naming**: lowercase underscore — `uglynote.cls`, `doccolor.sty`, `runinabstract.sty`
- **Comments**: English throughout
- **Example suffix**: `-cn` for Chinese (e.g. `uglynote-cn.tex`)
- **Gitmoji**: `:sparkles:`(feature), `:bug:`(fix), `:memo:`(docs), `:pencil2:`(typo), `:art:`(refactor), `:bricks:`(build), `:truck:`(rename), `:heavy_plus_sign:`(deps), `:globe_with_meridians:`(i18n)

## Development Workflow

1. Edit source under `texmf/tex/latex/uglylatex/`
2. Install: `make install` (or copy to example/ for quick testing)
3. Compile: `make test` (or `latexmk -xelatex --shell-escape example/uglynote-cn.tex`)
4. Inspect generated PDF visually
5. Full build: `make example`

## Key Architecture

- **`classoptions.tex`** is `\input` (not a package) — kvoptions keys before `\ProcessKeyvalOptions`.
- **`dependencies.sty`** loads all shared packages; `abstract` is NOT in it — each `customized_abstract/*.sty` loads it with per-class options (`[runin]` for UglyPaper).
- **`docstem.sty`** is conditional: `\ifUGLY@stem\RequirePackage{docstem}\fi`.
- **No automated tests** — manual PDF inspection.

---
> Source: [GitHubonline1396529/uglylatex](https://github.com/GitHubonline1396529/uglylatex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
