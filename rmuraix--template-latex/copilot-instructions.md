## template-latex

> - `npm install` installs the textlint toolchain used for content checks.

# Repository Guidelines

## Build & Lint Commands
- `npm install` installs the textlint toolchain used for content checks.
- `npm run lint` runs textlint over `**/*.{md,tex}` files (main quality gate).
- `npm run lint:fix` runs textlint with auto-fix enabled.
- To compile a document, run `latexmk` from within the project directory (e.g., `cd projects/abstract && latexmk`). The `.latexmkrc` files configure LuaLaTeX automatically; output goes to `out/`.

## Writing Your Paper

### Document entry point
Each project lives under `projects/<name>/`. Edit `projects/<name>/main.tex` to write your paper.

### Title metadata
Each project defines its own title macros in its style file. Set them in the preamble of `main.tex` — refer to `projects/<name>/style/` for the available commands.

### Bibliography
- Add entries to `references/main.bib` using the `authorYYYY` key convention (e.g., `smith2020`).
- Cite with `\cite{key}` and print the reference list with `\printreferences`.
- For Japanese-language entries, add `keywords = {ja}` so name delimiters switch to `，` automatically.
- The style is IEEE (biblatex + biber). Use `@article`, `@inproceedings`, etc.

### Figures and tables
- Place figures with `\includegraphics` inside a `figure` environment.
- Use descriptive labels with prefixes: `fig:` for figures, `tab:` for tables, `sec:` for sections.
- Reference with `\ref{fig:example}` or `\autoref{fig:example}`.

### Japanese writing conventions
- End sentences with a fullwidth full stop `．` (U+FF0E, not the ASCII period `.`).
- The textlint rule `ja-no-mixed-period` enforces this automatically.
- Use `\cite{a,b}` for multiple citations; they render as a single bracketed list `[1,2]`.

### Style files
- Shared settings (fonts, geometry, headings) are in `shared/style/jp-common.sty`. Avoid editing this unless the change applies to all projects.
- Project-specific overrides belong in `projects/<name>/style/`.

## Commit & Pull Request Guidelines
- Use Conventional Commits format: `type: summary` (e.g., `feat:`, `fix:`, `chore:`).
- Run `npm run lint` before submitting changes that touch prose or LaTeX text.
- Follow the organization-level contributing guide linked in `README.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rmuraix)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rmuraix)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
