## design-system-extractor

> - After any change that affects `viewer.html`, `run_pipeline.py`, viewer layout, viewer data shape, or embedded run output rendering, regenerate `viewer.html` before finishing.

# Repository Instructions

## Viewer Regeneration

- After any change that affects `viewer.html`, `run_pipeline.py`, viewer layout, viewer data shape, or embedded run output rendering, regenerate `viewer.html` before finishing.
- Use the repo virtual environment to do it:
  - `./venv/bin/python - <<'PY'`
  - `from run_pipeline import generate_viewer, RUNS_DIR, PROJECT_DIR`
  - `generate_viewer(RUNS_DIR, PROJECT_DIR / "viewer.html")`
  - `print("viewer regenerated")`
  - `PY`
- Prefer regenerating `viewer.html` proactively rather than waiting for the user to ask.

## Prompt Versioning

- When improving or experimenting with pipeline prompts, create a new `runs/vNNN/` version folder first and write the updated prompt files there.
- Do not edit prompt files inside a completed or broken run folder to fix that run retroactively; those folders are the system of record for what happened.
- Keep previous run prompt files intact unless the user explicitly asks to repair or backfill that specific version.
- Only update source default prompts in `src/screenshot_to_template/prompts.py` when the user explicitly asks to promote a versioned prompt into the defaults.

## Change Log Maintenance

- For any repo change tied to a pipeline version folder, create or update `runs/vNNN/changes.md` in that version folder before finishing.
- Keep `changes.md` factual and current: list prompt edits, source-code edits, generated artifacts, viewer regeneration, verification commands, and known follow-up work.
- If a change is not tied to a version folder, update the nearest relevant changelog or create a short `changes.md` beside the affected artifact when that would help future debugging.
- When creating a new version folder, initialize `changes.md` early and keep it updated as changes are made, not only after the final run.

## Run Manifest Status

- When entering, inspecting, or modifying a `runs/vNNN/` folder, inspect `manifest.json` first if it exists, then read `changes.md`.
- Treat `manifest.status: not_run` or `pipeline_run_completed: false` as a work-in-progress prompt/config bundle, not a completed pipeline run.
- Do not infer generated artifacts exist from planned inputs. Use `screenshots` for completed run outputs and `planned_screenshots` only as intended inputs.
- When the pipeline completes, the completed-run manifest may replace or expand the pre-run status manifest.

## Design System Token Guidance

- Fix recurring generated-site fidelity failures at the design-system prompt or site-generation prompt level first. Do not add deterministic post-processing, HTML rewriting, or CSS injection guards for these failures unless the user explicitly asks for that kind of repair.
- When the failure is caused by CSS mechanics, encode the mechanic in prompt language so future design systems and generations avoid it by construction. Example: content-hugging buttons or eyebrows inside column flex stacks need non-stretch parent alignment or explicit `align-self`; `inline-flex` plus `width:auto` is not enough because flex-column children stretch by default.
- When checking whether a prompt fix affected a run, inspect that run's local prompt files under `runs/vNNN/`; version folders may contain stale prompt copies that override newer source defaults.
- Never suggest section-specific design-system variable names such as `promoInverse`, `ctaCardBackground`, or `heroAccent` as a preferred fix.
- Design-system tokens and schema rules should describe generic reusable visual patterns and surface relationships, so the same pattern can be applied in non-promo, non-hero, or otherwise different contexts without confusing generators.
- If a source screenshot shows a one-off visual relationship, suggest capturing it as a generic pattern/rule, such as inverse-surface variant, high-contrast inset card, warm-on-dark control family, or conditional border behavior.
- When discussing possible fixes, make this distinction explicit: exact source values may be mapped into generic roles, but the role names and rules should not be tied to a specific section or content use case.
- Design-system prose may include section-aware rules for hero, footer, or one-off generic section behavior when that helps generation fidelity, but those section-aware rules must never become section-specific variable/token names.
- Prompt and design-system fixes should be palette-agnostic and broadly reusable unless the user explicitly asks for a one-off diagnosis. Do not encode a current run's colors, hues, section names, or content examples into general pipeline instructions; translate them into generic surface, contrast, hierarchy, nesting, border, and component-relationship rules.
- Before suggesting or writing a prompt change, check whether the wording would still make sense for a completely different source site with different colors, content, and section order. If not, rewrite it as a general principle.
- Section grounding should be a detailed factual inventory, not a short evidence summary. Preserve approximate hex/rgb values, detailed typography, image/graphic creative direction, component layout, and parent/child surface relationships. Avoid making "evidence", "local only", or "low confidence" the dominant output style.
- Surface-specific component variant names such as `button-primary-on-inverseStrong` or `eyebrow-accent-on-inverseStrong` are acceptable when they refer to reusable surface roles. They are different from forbidden section/content-specific variable names.

## Section Separator Versioning

- Use only `runs/section-separator/vNNN-*` directories for section-separator experiments, debug runs, bakeoffs, reviews, and artifacts.
- Do not create or rely on `runs/section-separator/latest/` as a second namespace.
- Allocate the version folder first, then write all run outputs inside that folder.
- Put primary outputs under `artifacts/`.
- If scratch space is needed, use `temp/` inside the same `vNNN-*` folder.
- When updating docs or references, point to the canonical `vNNN-*` path only.

---
> Source: [stanchev-flow/design-system-extractor](https://github.com/stanchev-flow/design-system-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
