## qml-noise

> - Authoritative manuscript: `report_pra_fixed.tex` with bibliography in `report_pra_fixedNotes.bib`; keep PDFs (`report_pra_fixed.pdf`, `paper_skelet.pdf`) synced.

# Repository Guidelines

## Project Structure & Module Organization
- Authoritative manuscript: `report_pra_fixed.tex` with bibliography in `report_pra_fixedNotes.bib`; keep PDFs (`report_pra_fixed.pdf`, `paper_skelet.pdf`) synced.
- Use `report_pra.tex` as a staging ground before promoting changes to the fixed manuscript.
- Noise experiments live in `noise_qml.ipynb` and `plancf_noise.ipynb`; keep outputs lightweight and annotate parameter choices inline.
- Generated artifacts stay beside their sources; never edit compiled PDFs directly.

## Build, Test, and Development Commands
- `latexmk -pdf report_pra_fixed.tex`: full LaTeX build with BibTeX and cross-reference passes.
- `pdflatex report_pra_fixed.tex && bibtex report_pra_fixed && pdflatex report_pra_fixed.tex`: fallback manual sequence.
- `jupyter nbconvert --to notebook --execute noise_qml.ipynb`: batch-execute notebooks to surface runtime issues.
- `jupyter lab`: launch an interactive workspace rooted in this repository.

## Coding Style & Naming Conventions
- Follow existing LaTeX indentation and line breaks; split long equations or sentences at logical clauses.
- Group new `\usepackage{}` calls and macro definitions near the top; name macros after QML noise concepts.
- Name notebooks and data with snake_case (e.g., `variance_scan.ipynb`) and figures with prefixes such as `fig_` or `tab_`.
- Default to ASCII and add concise comments only when the intent is not obvious from the code.

## Testing Guidelines
- Recompile after editing `.tex`; inspect the resulting `.log` until it is free of `Warning` and `Citation` messages.
- Execute notebooks from a fresh kernel before saving so outputs reflect a deterministic run; record seeds and simulator settings inside the notebook.

## Commit & Pull Request Guidelines
- Use Conventional Commit prefixes (`feat:`, `fix:`, `docs:`) with imperative subject lines.
- Rebase on the latest main branch prior to opening a PR; summarize sections touched and link supporting notebooks or figures.
- Attach regenerated PDFs or key plots in PR descriptions so reviewers can verify rendered changes quickly.

## Artifact Management & Reproducibility
- Store intermediate data and figures alongside their generating scripts; document metadata within the file headers.
- Keep notebook outputs minimal to ease diffs, but include enough context to replay experiments end-to-end.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BramDo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BramDo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
