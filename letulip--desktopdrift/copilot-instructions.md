## desktopdrift

> This project is a pure client-side HTML5 Canvas arcade drift-racing game.

# Desktop Drift — Claude Code Memory

This project is a pure client-side HTML5 Canvas arcade drift-racing game.
Full architecture and conventions are documented in **AGENTS.md**.
Development rules and coding principles are in **rules.md**.
UI/visual work follows **DESIGN.md** (see AGENTS.md → Design language for tokens).

## ▶ Start here — every session (read first)

The user will just say what to do. Before editing, in this order:

1. **Use the skills, don't re-derive the workflow.** Any code/asset change →
   follow the **desktopdrift-pr** skill (branch → `npm test` + `node --check` →
   browser-verify with SW cleared → bump SW cache → PR). Adding a Time Attack
   track → **desktopdrift-new-track** skill. They are concise; lean on them.
2. **Context is precious — don't read files whole.** AGENTS.md is 500+ lines:
   `grep` the section you need (`## Gotchas`, the module you're touching), don't
   cat it. Same for any file — read by offset/grep, not end to end.
3. **State a 1–3 line plan before editing** so a wrong direction is caught early.

### Hard rules (breaking these breaks production)
- **NEVER push to `main`** — it is the live GitHub Pages deploy. Always a new
  branch (`feat/ fix/ chore/ refactor/ perf/`) → PR. Never commit on `main`.
- **Before any commit:** `npm test` green **+** `node --check js/*.js`.
- **Changed JS/CSS/HTML/SVG?** Bump `const CACHE='desktop-drift-vN'` in `sw.js`
  (SW is stale-while-revalidate). A new JS module → also add it to `ASSETS`.
  Clear the SW before browser-verifying (snippet in the pr skill) or you see stale code.
- **All code is English** — comments, identifiers, strings (rules.md: "use only
  english throughout the entire project"). No Russian, no exceptions.
- **Minimal diff** — exactly what was asked, no speculative abstractions/features.
- **New pure logic ships with a unit test** in the same change (`tests/`, `node --test`).
- If anything is ambiguous, **ask — don't guess silently.**

## Quick-start

```bash
python3 -m http.server 8777   # serve from inside DesktopDrift/
# open http://localhost:8777/index.html
```

## Test + syntax check (run before every commit)

```bash
npm test   # node --test tests/*.test.js — must be green
node --check js/*.js && echo OK
```

## Branch policy

- `main` → production (GitHub Pages). Pushing to `main` = live deploy.
- New feature / fix → new branch (`feat/…`, `fix/…`), then PR.
- **New branch for every new request with new logic** (don't keep accumulating unrelated changes on the same branch).

## Development rules (summary — full text in rules.md)

### Think Before Coding
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First
- Minimum code that solves the problem. Nothing speculative.
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- If you write 200 lines and it could be 50, rewrite it.

### Core Principles
- **KISS** — as simple as possible, never simpler.
- **DRY** — single source of truth for every piece of logic.
- **YAGNI** — add a feature only when it is actually needed.
- **SoC** — each module/function addresses one concern.

### Before every git push
Update AGENTS.md to reflect actual structure, new files, and any gotchas discovered.

---
> Source: [letulip/DesktopDrift](https://github.com/letulip/DesktopDrift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
