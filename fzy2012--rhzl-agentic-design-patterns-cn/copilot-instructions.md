## rhzl-agentic-design-patterns-cn

> - All content lives at the repository root as numbered Markdown files (e.g., `00-Table-of-Contents.md`, `03-Foreword.md`, `07-Chapter-01.md`).

# Repository Guidelines

## Project Structure & Module Organization
- All content lives at the repository root as numbered Markdown files (e.g., `00-Table-of-Contents.md`, `03-Foreword.md`, `07-Chapter-01.md`).
- Keep bilingual layout per `rules.md` and preserve chapter numbering and order.
- Do not introduce new folders unless discussed; keep images inline links to external sources or propose an `images/` folder in the PR if needed.

## Build, Test, and Development Commands
- Preview Markdown: use your editor’s preview (e.g., VS Code) for quick checks.
- Optional link check: `npx markdown-link-check README.md -q` (run per edited file).
- Optional lint: `npx markdownlint-cli2 .` to catch headings, lists, and spacing issues.
- Quick local server (if embedding images): `python3 -m http.server` and open `http://localhost:8000/`.

## Coding Style & Naming Conventions
- Follow `rules.md` as the single source of truth for translation rules.
- File names: `NN-Title.md` (e.g., `06-What-Makes-Agent.md`, `21-Chapter-15.md`).
- Bilingual format: English paragraph followed by its Chinese translation; short chapters may use separate English/Chinese sections.
- Highlight Chinese with `<mark>…</mark>`; add spaces between Chinese/English and between Chinese/numbers.
- Markdown: GitHub‑flavored Markdown; lists use 2‑space indentation; avoid unnecessary bolding.

## Testing Guidelines
- Before submitting, ensure: no broken links, consistent terminology, matching English/Chinese paragraph pairs, and valid Markdown rendering.
- If using tools: run `markdownlint` and `markdown-link-check` with zero errors for changed files.
- Keep diffs focused—avoid reflowing untouched English source text.

## Commit & Pull Request Guidelines
- Commit messages (English):
  - `Add: [chapter] translation`
  - `Update: [chapter] formatting`
  - `Fix: [issue] in [chapter]`
- Pull Requests must include: clear scope/intent, list of touched files, notes on format decisions, and linked issues. Screenshots are helpful for complex formatting.

## Agent-Specific Instructions
- Only modify targeted chapters; do not renumber files.
- Preserve original English; do not paraphrase. Apply translations and formatting per `rules.md`.
- Keep contributions minimal and reversible; avoid adding tooling/config files without prior discussion.

---
> Source: [fzy2012/rhzl-Agentic-Design-Patterns-cn](https://github.com/fzy2012/rhzl-Agentic-Design-Patterns-cn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
