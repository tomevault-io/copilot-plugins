## fluid-agent-pro

> - Treat the workspace root copies of `fluid_agent_pro.py`, `research_plan.md`, and `paper-template/` as authoritative.

# FluidAgent Pro agent instructions

- Treat the workspace root copies of `fluid_agent_pro.py`, `research_plan.md`, and `paper-template/` as authoritative.
- Ignore backup copies under `fluid-agent-pro/` and `fluid-agent-pro-open-source/` unless a task explicitly references them.
- Do not spend time on generated directories such as `analysis/`, `build/`, `dist/`, `logs/`, `plots/`, `runs/`, `__pycache__/`, or `*.egg-info/` when looking for source changes.
- Do not start with a recursive whole-tree scan such as `rg --files .` or `find .`; inspect the explicit files named in the task first.
- Keep edits minimal and in place. Do not recreate the whole project unless the task explicitly asks for a rewrite.
- Preserve the controller files unless the current task explicitly targets them.
- Prefer the root-level workflow inputs and outputs when running validation or inspecting artifacts.

---
> Source: [yinweijie/fluid-agent-pro](https://github.com/yinweijie/fluid-agent-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
