## 90minutui

> Repository-level working rules for coding agents.

# AGENTS.md

Repository-level working rules for coding agents.

## Role

- Act as a senior software engineer.
- Be precise, direct, and brief.
- Prefer maintainability over cleverness.

## Project Intent

- Build a small Go TUI for browsing `90minut.pl`.
- Keep architecture simple: `fetch -> parse -> render`.
- Optimize for reliability of parsing and navigation UX.

## Code Standards

- Language: Go.
- Keep functions focused and composable.
- Use explicit types and clear names.
- Avoid comments that restate code.
- Add comments only when they preserve non-obvious context.

## Parsing Rules (Critical)

- Decode source content correctly (`iso-8859-2` aware).
- Use semantic selectors and structural markers.
- Avoid hardcoded row/column offsets where possible.
- Keep parser logic out of UI package.
- Preserve source URLs/IDs as stable domain keys.

## Testing Rules

- Add parser tests using saved HTML fixtures (`testdata/`).
- Cover at least:
  - seasons + selected season detection
  - competition extraction order
  - rounds + fixtures extraction
  - match timeline side assignment
- Prefer offline deterministic tests over live network tests.

## Git Rules

- Make small, focused commits.
- Do not rewrite history unless explicitly requested.
- Do not push unless explicitly requested.

## Non-goals (unless requested)

- No broad API service layer.
- No heavy persistence/caching layer.
- No scope expansion that harms one-binary simplicity.

---
> Source: [adrunkhuman/90minuTUI](https://github.com/adrunkhuman/90minuTUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
