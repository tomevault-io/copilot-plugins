## yokubi-zh-cn

> This repository is the Simplified Chinese edition of the Yokubi Japanese grammar guide. Book content lives in `src/`: `SUMMARY.md` defines navigation, introductory pages sit at the top level, and numbered lessons are grouped under `Section1/` and `Section2/`. Images belong in `src/images/`. `book.toml` configures mdBook and the furigana preprocessor. Translation checks are in `scripts/validate_translation.py`; `scripts/preprocess-furigana.py` renders `{f|汉字|读音}` markup. Search customization is under `js/`, styling under `style/`, and GitHub Pages deployment under `.github/workflows/`. The generated `book/` directory is ignored and must not be committed.

# Repository Guidelines

## Project Structure & Module Organization

This repository is the Simplified Chinese edition of the Yokubi Japanese grammar guide. Book content lives in `src/`: `SUMMARY.md` defines navigation, introductory pages sit at the top level, and numbered lessons are grouped under `Section1/` and `Section2/`. Images belong in `src/images/`. `book.toml` configures mdBook and the furigana preprocessor. Translation checks are in `scripts/validate_translation.py`; `scripts/preprocess-furigana.py` renders `{f|汉字|读音}` markup. Search customization is under `js/`, styling under `style/`, and GitHub Pages deployment under `.github/workflows/`. The generated `book/` directory is ignored and must not be committed.

## Build, Test, and Development Commands

- `mdbook serve`: build the guide, watch for changes, and serve it locally.
- `MDBOOK_BUILD__CREATE_MISSING=false mdbook build`: perform the production-style build used by CI.
- `python3 scripts/validate_translation.py --source-rev b1c0938b0bda58e20c6ccd21288b46711b438239`: run completeness, structure, link, typography, and protected-content checks against the translation baseline.
- `git fetch upstream`: retrieve new commits from the English source repository without merging them.

Run the validator and production build before every pull request.

## Content Style & Naming Conventions

Follow `TRANSLATION_GUIDE.md` for the authoritative glossary and editorial rules. Write natural Simplified Chinese for beginners, preserve uncertainty in the source, and leave Japanese examples unchanged. Use full-width Chinese punctuation, `「」` quotation marks, and Unicode arrows such as `→`. Preserve HTML, Markdown targets, iframe attributes, two-space line breaks, and furigana macros exactly. Translate escaped placeholders while retaining their structure, for example `\<verb stem\>` becomes `\<动词词干\>`. Lesson files use `LessonN.md`; lesson headings use `第 N 课：……`. Python uses four-space indentation and standard-library conventions; JavaScript and CSS should match nearby files.

## Testing Guidelines

There is no separate unit-test framework or coverage target. Treat validator failures and mdBook warnings as release blockers. When changing navigation, confirm every entry in `src/SUMMARY.md` resolves. For CSS, search, or UI changes, inspect the local site at desktop and narrow viewport widths and include screenshots in the pull request.

## Commit & Pull Request Guidelines

Use short, imperative commit subjects consistent with history, such as `Translate remaining tutorial placeholders` or `Fix Japanese glyph rendering`. Keep translation, tooling, and styling changes logically separated. Pull requests should summarize affected lessons, explain intentional departures from the source, link relevant issues or upstream commits, and report validation/build results. Record corrected source errors or disputed translation choices in `TRANSLATION_STATUS.md`.

Pushes to `main` are intended to deploy the site through GitHub Actions. Keep `origin` pointed at `yokubi-zh-cn` and `upstream` pointed at `Morgawr/yokubi`.

---
> Source: [L-M-Sherlock/yokubi-zh-cn](https://github.com/L-M-Sherlock/yokubi-zh-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
