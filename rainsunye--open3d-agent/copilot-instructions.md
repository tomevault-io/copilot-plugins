## open3d-agent

> Read these files before you execute a user request:

# Claude Code Driver

Read these files before you execute a user request:

1. [docs/claude_code_driver.md](/home/yf/3D-IC-CLAW/docs/claude_code_driver.md)
2. [docs/request_schema.md](/home/yf/3D-IC-CLAW/docs/request_schema.md)
3. [docs/stage_playbook.md](/home/yf/3D-IC-CLAW/docs/stage_playbook.md)
4. relevant files under `docs/skills/`

Then follow this contract:

1. Convert the natural-language task into a structured request JSON.
2. Save it under the target run or working directory.
3. Run `python3 -m 3dic_claw run --request-file <request.json>`.
4. Stay with the run or resume the same run directory until `final_report.json` and `final_report.md` both exist.
5. Read `run_status.json`, stage reports, and the final reports before answering.

Do not send natural-language prompts directly into Python.

## Driver Rules

- Prefer `case_type=orfs_case` when the user provides a runnable ORFS package.
- Treat missing ORFS inputs as blockers. Do not invent RTL, SDC, or config files.
- Keep all generated files inside the current run directory.
- Leave `reuse_orfs_results=false` unless the user explicitly asks to reuse upstream ORFS artifacts.
- Leave `use_llm_advisor=true` unless the user explicitly disables Claude-assisted hybrid stages.
- When a stage fails, inspect the stage report, the latest log, and any `debug/llm_*` records before deciding whether to retry or stop.
- For full-flow prompts, follow `3dic-single-shot-end-to-end-completion`: long backend runtime is not a reason to stop, and a task is not complete without both final report files.
- For full-flow prompts with OpenROAD-3D backend results, also follow `3dic-drc-auto-fix-loop`: do not stop at the first routed result when route DRC is still above the configured target. In the default `drc_fix_mode=llm_guided`, Python generates evidence and run-local scaffolding while Codex/Claude chooses the repair strategy, patch, rerun point, and stop decision. Allow the repair loop to fall back to `Place-3D -> OpenROAD-3D` when the evidence is structural.
- DRC repair is bounded to initial run plus at most two LLM-guided repair runs.
- Complete success also requires checked GDSII export status and real 3D / MoL structural acceptance; `6_final.def` alone is not sufficient.
- If an existing run has upstream reports but is missing downstream reports or final reports, resume it with the structured continuation path instead of rerunning completed upstream stages.

## Agent Model

- Deterministic agents: pure Python and shell orchestration
- Hybrid agents: deterministic core plus Claude advisor
- Reporting agents: summarize state, blockers, risks, and next steps

The current hybrid stages are:

- `technology_adaptation`
- `log_diagnose`
- `repair`

## Skill-Style References

- [docs/skills/3dic-project-intake.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-project-intake.md)
- [docs/skills/3dic-open3dbench-technology-discovery.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-open3dbench-technology-discovery.md)
- [docs/skills/3dic-orfs-package-validation.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-orfs-package-validation.md)
- [docs/skills/3dic-orfs-artifacts.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-orfs-artifacts.md)
- [docs/skills/3dic-technology-adaptation.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-technology-adaptation.md)
- [docs/skills/3dic-place3d-package.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-place3d-package.md)
- [docs/skills/3dic-place3d-execution.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-place3d-execution.md)
- [docs/skills/3dic-openroad3d-package.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-openroad3d-package.md)
- [docs/skills/3dic-openroad3d-execution.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-openroad3d-execution.md)
- [docs/skills/3dic-drc-auto-fix-loop.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-drc-auto-fix-loop.md)
- [docs/skills/3dic-log-diagnose-repair.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-log-diagnose-repair.md)
- [docs/skills/3dic-result-management.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-result-management.md)
- [docs/skills/3dic-final-report.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-final-report.md)
- [docs/skills/3dic-single-shot-end-to-end-completion.md](/home/yf/3D-IC-CLAW/docs/skills/3dic-single-shot-end-to-end-completion.md)

---
> Source: [RainsunYe/Open3D-Agent](https://github.com/RainsunYe/Open3D-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
