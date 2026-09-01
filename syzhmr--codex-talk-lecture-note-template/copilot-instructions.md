## codex-talk-lecture-note-template

> This file is the short, always-on entry point. It routes to detailed policies

# Talk/lecture-note Codex instructions

This file is the short, always-on entry point. It routes to detailed policies
without duplicating them.

## Start and scope

1. Run `git status --short --branch` and preserve pre-existing changes.
2. Classify the request as policy, inventory, handwriting, references,
   prose/proof, LaTeX, terminology, or Git work. Read only the matching heading
   in `WORKFLOW_STEPS.md`.
3. Lock content work to one coherent lecture topic, theorem, diagram, or
   handwritten page range.
4. For work spanning several topics, policy/tooling changes, major unknowns, or
   work likely to outlive one session, create an ExecPlan under `work/plans/`
   using `PLANS.md`.

The latest user instruction wins within the project. Preserve user changes and
locked decisions; do not rewrite a whole lecture day to fix a local issue.

## Read only what the task needs

- Routing and stop conditions: `WORKFLOW_STEPS.md`.
- Project invariants: the relevant part of `PROJECT_PROFILE.md`.
- Mathematical content: the target TeX, exact handwritten page images, and
  matching rows in `WORK_LEDGER.md` and `NOTE_POLICY.md`.
- Proof drafting, reconstruction, compression review, or proof-policy revision:
  read `HANDWRITTEN_PROOF_POLICY.md`, then use
  `$handwritten-proof-density`.
- LaTeX-only work: `LATEX_SOURCE_POLICY.md` and the target TeX.
- Terminology: `TERM_DICTIONARY.md`.
- References: the relevant index/note, a bounded search in `references/text/`,
  and finally the exact source page or tracked image.

Do not preload the full ledger or every policy. OCR and extracted text locate
evidence; formulas, diagrams, quotations, and ambiguous symbols require the
original page image. Handwritten notes are checked visually.

## Source priority

1. User's latest instruction and locked decisions.
2. Original handwritten lecture pages for topic order, content, notation,
   colors, diagrams, proof skeleton, and explanation density.
3. Speaker materials such as board photos, slides, and handouts according to
   the project-specific order in `PROJECT_PROFILE.md`.
4. References for verification and support, with hypothesis, notation, and
   scope differences recorded before use.
5. Existing TeX, which may be repaired when it compresses higher-priority
   lecture material.
6. Tool or model output, which supplies evidence or test obligations but never
   decides acceptance by itself.

Do not replace the lecture's argument with a cleaner or shorter reference
argument.

## Editing and verification

- Make the smallest defensible patch and preserve lecture notation.
- Before prose or proof content edits, record a compact target density profile
  and gap map. Use a detailed reconstruction table for reconstruction,
  handwritten integration, broad regeneration, or suspected compression.
  Deferred proofs always require the ledger register.
- Update the corresponding `WORK_LEDGER.md` row and any affected detailed
  record under `work/records/handwritten/` in the same work unit.
- Normal review includes source-aware and internal-reader passes.
- Build after TeX changes using the command in `README.md`.
- Treat build success as typesetting validation, not mathematical or
  readability validation.
- Before reporting, inspect `git diff --check`, the focused diff, and final
  status. Report what was not verified.

Do not install dependencies, commit, push, or rewrite history unless requested.

---
> Source: [syzhmr/codex-talk-lecture-note-template](https://github.com/syzhmr/codex-talk-lecture-note-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
