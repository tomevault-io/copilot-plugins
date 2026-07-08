## family-dashboard

> Full spec: `family-dashboard.md`.

# Family Dashboard

Full spec: `family-dashboard.md`.

## Commands
- Backend: `cd backend && uvicorn main:app --reload`
- Frontend: `cd frontend && npm run dev`
- Backend tests: `cd backend && python -m pytest --tb=short`
- Frontend tests: `cd frontend && npx vitest run`
- Lint check: `cd backend && ruff check . && cd ../frontend && npx eslint src/`

## MUST follow — build order
1. MUST write Gherkin feature file first, before any code
2. MUST write failing tests before implementation
3. MUST write minimum code to pass tests — nothing more
4. MUST run mutation tests after implementation; MUST NOT leave surviving mutants without documented justification
5. MUST confirm all tests pass before committing
6. MUST update `MUTANTS.md` for any surviving mutants that will not be addressed — record the mutant ID, what was mutated, and why it is acceptable

## Session startup
- Fetch deferred tools before starting any task:
  `ToolSearch: "select:AskUserQuestion,TodoWrite"`
- Confirm both tools are available before proceeding

## MUST follow — git
- MUST check current branch before starting: `git branch --show-current`
- MUST NEVER write files or commit on `main` — hooks enforce this and will block you
- MUST name branches `feature/<name>` cut from `main`
- MUST commit atomically with conventional commit messages after each logical step
- MUST inform the user when a feature is complete — NEVER merge or raise a PR autonomously
- MUST use the AskUserQuestion tool to ask clarifying questions before writing any code if anything in the current task is ambiguous. Do not guess. If the session prompt is explicit and complete, proceed without asking.

## NEVER do
- NEVER edit `.env` — hooks will block this; update `.env.example` instead
- NEVER run `rm -rf`, `git push --force`, or `git reset --hard` — hooks will block these
- NEVER run ruff or eslint manually — hooks run them automatically on file save
- NEVER implement behaviour not covered by a feature file

## Before writing any UI component
- MUST identify the purpose, audience, and tone of the component before writing any JSX or CSS
- MUST choose a deliberate aesthetic direction and state it — NEVER default to "clean and minimal" without justification
- MUST use tabular/monospaced numerals for times and figures so they align across cards
- MUST reuse existing spacing scale (multiples of 4 px or 8 px) and colour variables — NEVER set arbitrary values inline
- MUST keep internal padding consistent across all cards
- MUST use the existing delay colour system (green / amber / red) for any status indicators
- NEVER use Inter, Roboto, or Arial as the primary display face
- NEVER use purple gradients on white backgrounds
- NEVER add animation that does not communicate meaning; if animating, use transform/opacity only (150–300 ms, ease-out on enter)
- NEVER add decorative elements not justified by the design rationale

## When compacting
Preserve: current branch name, list of modified files, last test run status, any surviving mutants noted.

---
> Source: [Ryan684/family-dashboard](https://github.com/Ryan684/family-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
