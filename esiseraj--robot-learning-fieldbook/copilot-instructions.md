## robot-learning-fieldbook

> These rules apply to every change in this repository, whether made by a person or by an AI assistant.

# Robot Learning Fieldbook — working agreement

These rules apply to every change in this repository, whether made by a person or by an AI assistant.

## Priorities

- Educational and technical correctness outrank UI polish.
- Teach mechanisms before model names.
- Preserve every substantive curriculum obligation when reorganizing or adding material.
- Connect intuition, formalism, architecture, implementation, training, inference, robot behavior, controller assumptions, trade-offs, failures, evidence, and interview expectations.
- Never invent citations, numbers, dates, or results. Mark unknowns as unknown.

## Repository map

| Path | Role |
|---|---|
| `src/` | The React app (`learning-studio.tsx` views, `learning-studio-state.ts` routing/persistence/search, `globals.css`). |
| `content/source/original-curriculum.md` | **Immutable** source curriculum. Never edit it. |
| `scripts/build-content-index.mjs` | Stage 1 compiler: source → inventories, coverage, semantic routes, backlogs. |
| `scripts/build-learning-content.mjs` + `scripts/*-specs.mjs`, `*-contracts.mjs`, `concept-*.mjs`, `model-family-enrichment.mjs`, `comparison-content.mjs`, `evidence-resource-specs.mjs`, `claim-verification.mjs` | Stage 2 compiler and its **canonical authored inputs**: chapters, concepts, equations, diagrams, implementations, interview questions, resources, comparisons, search index, and the app payload. |
| `scripts/build-deepmind-flashcards.py` | Extracts the flashcard deck from `DeepMind/DeepMindPrep.apkg` into `content/flashcards/` and `public/deepmind-flashcards/media/`. |
| `content/` (everything except `source/`) | **Generated** output of the compilers: `.mdx` audit exports, catalogs, metadata, `app-content.json`. |
| `examples/runnable/` | 108 generated NumPy mechanism examples; `examples/tests/test_mechanisms.py` holds their numerical tests. |
| `scripts/validate-*.mjs` | Validators for coverage, resources, code, diagrams, formalism, interviews, flashcards. |
| `reports/` | Dated ledgers: claim verification, resource link health and waivers, model-history verification. |
| `docs/` | Pedagogy, content architecture, technical review, evidence policy, diagram and implementation standards, testing, and how to add material. |
| `tests/` | Vitest + Testing Library suites for the app; `tests/browser/` holds the Playwright + axe gate that runs against the production build. |

## Source and generated-content integrity

- Never edit `content/source/original-curriculum.md`. Treat it as two logical documents: `curriculum-a` (lines 1–1918) and `curriculum-b` (lines 1919–3970); numeric references are scoped per logical document.
- The compilers own everything under `content/` except `source/`, plus `examples/runnable/`. Change the canonical specification in `scripts/`, run `npm run content:compile`, and review the diff. Do not hand-edit generated `.mdx`, catalogs, indices, or runnable examples.
- Do not equate "mapped", "authored", "implemented", "verified", "technically reviewed", or "independently reproduced". Record each state truthfully.
- Preserve stable source, chapter, section, concept, diagram, implementation, question, resource, and provenance IDs.

## Teaching and evidence contract

- Define every symbol, random variable, tensor shape, unit, coordinate frame, action horizon, rate, and relevant observability assumption.
- Explicitly separate training from inference, and the model sampler from the robot feedback/controller loop.
- Every algorithmic or architectural addition requires an accurate, accessible diagram.
- Every implementable mechanism requires a minimal faithful implementation and proportionate tests.
- Every external claim requires provenance. Prefer primary sources, store verification dates, qualify company claims, and mark unknown details instead of inferring them.
- Never imply independent reproduction from open artifacts, local execution, or technical review.

## Workflow for a change

1. Inspect existing coverage and concepts before adding a new section (`content/metadata/concept-graph.json`, `content/coverage/source-coverage-matrix.md`).
2. Edit the owning canonical input in `scripts/`.
3. Regenerate: `npm run content:compile`.
4. Validate: `npm run validate:formalism && npm run validate:interviews && npm run validate`.
5. Test: `npm run typecheck && npm run test:unit && npm run test:mechanisms`.
6. Build, gate, and look at it: `npm run build && npm run test:browser && npm run preview`.
7. Report files changed, checks run, failures, and anything still pending verification — without inflation.

`npm run check:links` audits every resource URL over the network and rewrites `reports/resource-link-health.*`; run it deliberately, not as part of routine edits.

## Publishing

Pushing to `main` runs `.github/workflows/deploy.yml`, which validates, tests, builds, and deploys `dist/` to GitHub Pages. Keep `main` green; open a pull request for anything larger than a typo.

Detailed standards live in `docs/`.

---
> Source: [EsiSeraj/robot-learning-fieldbook](https://github.com/EsiSeraj/robot-learning-fieldbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
