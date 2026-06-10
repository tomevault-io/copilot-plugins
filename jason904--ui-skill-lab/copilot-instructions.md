## ui-skill-lab

> This file provides guidance to Claude Code when working in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working in this repository.

## Project Overview

UI Skill Lab is a test and validation lab for AI-driven UI development skills. It contains 8 Codex skills forming a complete "Prompt to Pixel" pipeline and end-to-end test infrastructure to validate them.

## Skill Directory Convention

Every skill lives under `.codex/skills/<name>/`:

```
skill-name/
├── SKILL.md              # Required: skill definition with frontmatter
├── references/           # Required: reference docs referenced by SKILL.md
├── examples/             # Required: example outputs
└── README.md             # Recommended
```

## Pipeline Order

Skills are numbered by pipeline order (01 → 08):

1. **image2-prompt-pack** — PRD → prompt pack
2. *(02 reserved for reference.png generation)*
3. **visual-to-spec** — Screenshot → tokens + component tree
4. **visual-spec-review** — Adversarial spec review
5. **design-system-governor** — Static spec → design system
6. *(Implementation written manually)*
7. **spec-compliance-review** — Code vs spec audit
8. **visual-acceptance-review** — Screenshot visual comparison
9. **visual-diff-fix** — Structured visual fixes
10. **ui-ux-pro-max-skill** — Standalone design intelligence (not in pipeline)

## Test Infrastructure

### Scripts (zero npm deps, Node.js built-ins only)

| Script | Purpose |
|--------|---------|
| `scripts/validate-skill-source-contracts.mjs` | Validates SKILL.md frontmatter, reference file existence, example JSON |
| `scripts/validate-ui-skill-e2e.mjs` | Validates complete pipeline output per fixture case |
| `scripts/seed-e2e-fixtures.mjs` | Generates deterministic fixture cases |
| `scripts/seed-visual-reconstruction.mjs` | Creates benchmark case directories |
| `scripts/capture-visual-reconstruction.mjs` | Playwright screenshots of candidate implementations |
| `scripts/write-visual-baseline-candidates.mjs` | Writes baseline HTML/CSS for benchmarks |
| `scripts/compare-visual-reconstruction.py` | Pixel metrics (numpy + Pillow) |
| `scripts/capture-real-run-screenshots.mjs` | Screenshots for real-run cases |
| `scripts/run-real-image-led-landing.mjs` | Regenerates image-led-landing test case |

### Running Tests

```bash
npm run fixtures:seed   # Seed E2E fixtures (required first)
npm test                # Run all tests
npm run test:skills     # Skill source contracts only
npm run test:e2e        # E2E pipeline validation
```

## Code Conventions

- **Node.js scripts:** ES modules, `node:` built-ins only (no npm deps)
- **Python scripts:** Standard library preferred; external deps in `requirements.txt`
- **Documentation:** English for all docs and code comments
- **Skill frontmatter:** Every SKILL.md must have `name`, `description`, `argument-hint` in YAML frontmatter and `Version: X.Y.Z` in the body

## Key Terms

See `docs/glossary.md` for full definitions.

- **source** — Where a value came from. Validator-supported values: `reference-visible`, `screenshot-estimated`, `screenshot-inferred`, `inferred-implementation`, `prd-required`, `openspec-required`, `prompt-pack-intent`, `ui-ux-pro-max`, `web-design-guidelines`, `frontend-design`, `design-fidelity-reviewer`, `visual-ui-verification`, `playwright-skill`, `human-approved`, `design-system-inferred`, `responsive-required`, `script-backed`
- **confidence** — 0.0 to 1.0 numerical confidence
- **strictness** — `required`, `recommended`, `optional`, `review-required`, `flexible` (aligned with validator contract)
- **UI Type** — `application-dashboard`, `image-led-landing`, `panorama-scene`, `mobile-app`, `poster-like-ui`, `unknown`
- **bbox** — Bounding box `[x, y, width, height]` locating an element in the reference screenshot

## Third-Party Code

`ui-ux-pro-max-skill` is a vendored copy of <https://github.com/nextlevelbuilder/ui-ux-pro-max-skill> (MIT). See `THIRD_PARTY_NOTICES.md`.

---
> Source: [Jason904/ui-skill-lab](https://github.com/Jason904/ui-skill-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
