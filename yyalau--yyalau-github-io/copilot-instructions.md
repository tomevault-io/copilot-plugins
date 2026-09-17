## yyalau-github-io

> Guidance for AI coding agents (and humans) in this repo. Read before changing things.

# AGENTS.md

Guidance for AI coding agents (and humans) in this repo. Read before changing things.

## Project

Static **music-theory quiz** site on **GitHub Pages (Jekyll)** - plain
HTML/CSS/vanilla JS. No bundler, no framework, no npm, no build step.

- `index.html` - landing page.
- `music/` - quizzes + shared engine `quiz-core.js` (`QuizCore`).
- `resources/` - private reference notes (not served).
- `_config.yml` - Jekyll config. `exclude:` lists files kept off the site.
- `.githooks/pre-commit` - the pre-commit hook.

## Publishing

Anything in `exclude:` in `_config.yml` is **not** served (404) but stays in git.
`AGENTS.md` and `resources/` are excluded. Never serve agent-internal docs;
add new private files to `exclude:` and don't link to them.

## Local preview

    python3 -m http.server 8000   # then open http://localhost:8000/

## Conventions

Vanilla ES5 JS (`var`, `function`, IIFEs). Match existing style; shared logic
in `music/quiz-core.js`; keep quiz pages thin (data + `QuizCore` config).

## Commits (binding)

A pre-commit hook at `.githooks/pre-commit` runs on every commit. On a fresh
clone, enable once: `git config core.hooksPath .githooks`. It checks staged
content for whitespace/conflict markers, missing final newline, JS syntax
(`node --check`), `console.log`/`debugger`, and files >1 MB.

1. **Every commit must pass the pre-commit hook.**
2. **Never use `--no-verify`** - no exceptions. If it fails, fix the issue and
   re-stage; if a check is wrong, fix the hook (committed through the hook).
3. Don't disable or weaken the hook to force a commit through.

---
> Source: [yyalau/yyalau.github.io](https://github.com/yyalau/yyalau.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
