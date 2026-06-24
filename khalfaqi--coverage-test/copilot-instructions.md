## coverage-test

> Build `/app/` from this bundle.

# Goal

Build `/app/` from this bundle.

# Inputs

- `pseudocode/` — per-feature, per-swimlane behavioural specs (markdown)
- `summary/` — feature briefs (HTML + PDF) with end-user copy
- `assets/` — SVG + PNG + `.structure.txt` for each canvas node
- `reference_code/` — optional; reuse if present, else derive from pseudocode
- `tech-stack.md` — the technologies, project layout, and verification gate the generated `/app/` must conform to
- `prompts/` — the staged build instructions (one file per stage)

# Output

`/app/` — a project that follows the **Project layout** section of `tech-stack.md` and uses the languages, frameworks, and libraries it names.

# How to build

Run the staged prompts under `prompts/` in order. Each stage has its own success criterion stated at the bottom of the file; do not advance to the next stage until the current stage's criterion is met.

1. **`prompts/01-scaffold.md`** — create the empty project skeleton from `tech-stack.md`'s **Project layout**.
2. **`prompts/02-data.md`** — generate types, schemas, and validators from `pseudocode/<feature>/Data/`.
3. **`prompts/03-logic.md`** — implement controllers from `pseudocode/<feature>/Logic/` and the **Verification gate** endpoint.
4. **`prompts/04-frontend.md`** — build components and pages from `pseudocode/<feature>/Frontend/`, `assets/`, and `summary/`.
5. **`prompts/05-verify.md`** — install, boot, and confirm the **Verification gate** passes end-to-end.

If any stage fails its success criterion, stop, report the failing stage and the specific check that failed, and do not run subsequent stages.

# Verification

Stage 05's success criterion is the final pass/fail for the whole bundle. The criterion's authoritative source is the **Verification gate** section of `tech-stack.md`.

# Notes

- If `/reference_code/` is absent from the bundle, ignore every instruction in any stage that tells you to reuse or mirror code from `/reference_code/` (or `/code/`) and derive everything from `/pseudocode/`, `/summary/`, and `/assets/` instead.
- Every stage defers stack-specific decisions (language, frameworks, library names, env-var names, install/dev commands, the verification gate's exact route and response) to `tech-stack.md`. If `tech-stack.md` is silent on a detail, follow the convention idiomatic to the **Frontend framework** or **Backend framework** named there.

---
> Source: [khalfaqi/coverage-test](https://github.com/khalfaqi/coverage-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
