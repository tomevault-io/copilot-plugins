## dpisr-evaluation-skill

> - Before changing scoring logic, verify the applicable GB/T 46071-2025 rule source first; treat Appendix A / configured YAML evidence criteria as the authoritative scoring reference and do not present implementation fallbacks as the national standard.

# Social Responsibility Eval Agent Rules

## Scoring standard synchronization
- Before changing scoring logic, verify the applicable GB/T 46071-2025 rule source first; treat Appendix A / configured YAML evidence criteria as the authoritative scoring reference and do not present implementation fallbacks as the national standard.
- Indicator scores must remain integer values from 0 to 10. Do not reintroduce fractional midpoint/bonus scores in JSON, rationale text, gap reports, or Excel/export output.
- Whenever a numeric score is revised, regenerate or synchronize the corresponding rationale and export text in the same change so the standard, score, and explanation stay consistent.

## Post-sync parity and cleanup
- Scoring-standard refactors are complete only after checking all maintained roots: `.opencode/skills/social-responsibility-eval`, `social-responsibility-eval`, and `social-responsibility-eval-wk`.
- Sync only source guidance/config/prompt/lib/script files, verify copied-file parity with `diff -ru`, `cmp -s`, or `shasum -a 256`, exclude `__pycache__/`, `*.pyc`, and generated outputs, then run final diagnostics/tests/validation in every root.

## Regression-debugging preflight
- Before code changes or long workflow reruns for a suspected regression, verify the active context: `pwd`, `git rev-parse --show-toplevel`, current branch/status, active skill name and resolved path, and whether `.opencode/skills/social-responsibility-eval`, `social-responsibility-eval`, or `social-responsibility-eval-wk` is actually in use.
- When relevant, confirm the runtime/interpreter/process that is executing the skill, then inspect local `git log`, `git blame`, and recent diffs before attributing the issue to a recent modification.

## Score-comparison audit (compare-runs)
- Before explaining why a score or rating changed, cite objective run-delta evidence first. Do not attribute score changes to model instability or scoring-policy/scoring-logic changes until parser failures, xparse/PDF/DOCX fallback status, Phase 1 matching deltas, sub-item/cross-file aggregation config, web-gate state, mandatory-gate status, NA/red-flag changes, and scoring formula/version changes are ruled out.

## Phase 4 / final rating evidence gates
- Phase 4 web evidence is never scoring-active merely because a search result exists, looks relevant, or was user-confirmed. It can affect coverage, score, or star level only after it has been evaluated against the exact YAML `star_levels[*].sub_items` / criteria for the target indicator and target star.
- Coverage upgrades and web-result counting must happen only after the stricter criteria check passes. Do not count descriptive, indirect, generic, promotional, or topic-adjacent web evidence as coverage before that check.
- Phase 5 / final scoring must fail closed: entries without both `verified=true` and an explicit criteria gate pass from Phase 4 must not influence coverage, score, or star level.
- The final rating gate must enforce all ★ mandatory items: indicators 1, 5, 9, 13, and 14 must meet the mandatory minimum score. For large-network-platform cases, include the template-backed ☆ indicators 2, 11, and 15 in the same mandatory gate.

## TextIn/xparse DOCX enhancement gate
- Before recommending or changing DOCX parsing behavior, consult official TextIn docs first, especially `https://docs.textin.com/xparse/v1/parse-config.md`, plus parse-response and SDK/OpenAPI docs when needed.
- Validate documented knobs against `config.yaml` `document_parsing.xparse` and `lib/file_loader.py` `_try_xparse_docx_hybrid()`: `include_table_structure`, `include_hierarchy`, `title_tree`, `include_char_details`, `include_inline_objects`, `table_view`/`table_flavor`, `parse_mode`, `get_image`/`image_output_type`, and local gates such as `min_text_length`/`min_image_size` where present.
- Require a representative-DOCX test comparing current xparse hybrid behavior, enhanced knobs, and fallback `python-docx`, with attention to OCR quality, table ordering, and evidence preservation. Treat TextIn/xparse as an optional enhancement layer with transparent fallback, not a mandatory full replacement.

---
> Source: [intsigSecurity/DPiSR-evaluation-skill](https://github.com/intsigSecurity/DPiSR-evaluation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
