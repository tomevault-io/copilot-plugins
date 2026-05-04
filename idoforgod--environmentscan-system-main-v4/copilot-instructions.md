## environmentscan-system-main-v4

> - **SOT validation**: Before running any workflow, execute `python3 env-scanning/scripts/validate_registry.py` and confirm exit code 0 (68 checks).

# Gemini CLI Instructions

@AGENTS.md

---

## Gemini CLI-Specific Directives

### Execution Notes

- **SOT validation**: Before running any workflow, execute `python3 env-scanning/scripts/validate_registry.py` and confirm exit code 0 (68 checks).
- **Master Gate M4**: After integration, `python3 env-scanning/scripts/validate_completion.py --sot {SOT} --date {DATE}` must PASS (10 CG checks, including CG-010 dashboard). M4 is NEVER skippable — even in autopilot mode.
- **Dashboard validation**: After dashboard generation, `python3 env-scanning/scripts/validate_dashboard.py --dashboard {path} --date {DATE}` (6 DB checks).
- **Pipeline Gate 2 (all WFs)**: `python3 env-scanning/scripts/validate_phase2_output.py --sot {SOT} --workflow {wf} --date {DATE} --json` — 9 checks (PG2-001~009 including title_ko).
- **Dashboard bilingual**: Dashboard now displays EN/KO bilingual signal titles and narrative subtabs. D3.js signal map in timeline tab requires CDN (offline fallback to markdown).
- **Task Management**: `python3 env-scanning/core/master_task_manager.py --action init` generates task specs; `--action verify` post-checks completeness. LLM copies Python output verbatim.
- **Report validation**: After generating any report, execute `python3 env-scanning/scripts/validate_report.py <report_path> --profile <profile>` and confirm exit code 0.
- Validation profiles: `standard` (WF1/WF2), `naver` (WF3), `multiglobal-news` / `multiglobal-news_en` (WF4), `integrated`, `weekly`, `timeline` (Timeline Map).
- **Timeline Map validation**: Timeline Map has dedicated quality defense — `validate_timeline_map.py` (L2a, 18 checks), `validate_timeline_map_quality.py` (L2b, 11 TQ checks), `narrative_gate.py` (B4, 5 NG checks). These verify Python-enforced determinism compliance (PB-1~6 verbatim match).

### File Search

When looking for configuration or data files, refer to the SOT at `env-scanning/config/workflow-registry.yaml` — it defines all paths. Never assume file locations.

### Report Generation

1. Read the appropriate skeleton template (paths listed in AGENTS.md Section 3.4)
2. Fill content into the skeleton structure — do NOT generate free-form
3. Run `validate_report.py` on the generated report
4. On failure: fix issues and re-validate (max 2 retries)

### Language

- Internal analysis and reasoning: English
- All report output and user-facing content: Korean
- STEEPs terminology must be preserved exactly during translation
- **WF4 multilingual pipeline**: WF4 scans 43 global news sites in 11 languages. Source articles are translated to English during Phase 1 collection, then the final report follows the standard English-first → Korean translation pipeline. Use `translation_validator.py` to verify structural integrity after translation.

---
> Source: [idoforgod/EnvironmentScan-system-main-v4](https://github.com/idoforgod/EnvironmentScan-system-main-v4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
