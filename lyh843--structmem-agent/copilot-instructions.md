## structmem-agent

> This repository extends ALFWorld with a custom structured-memory text agent. The upstream-style package lives in `alfworld/alfworld/`, with environments, training agents, generators, and planner code. Custom project logic is under `alfworld/my_agent/`: `agent/` contains the framework, structure parser, and action agent; `memory/` and `object/` hold shared state models; `test_agent/` contains runnable evaluation and debugging scripts. Configuration files are in `alfworld/configs/`, executable helper scripts are in `alfworld/scripts/`, and project documentation/assets are in `dosc/`. Generated evaluation reports and run logs are written under `alfworld/my_agent/logs/`; avoid committing new large or transient outputs.

# Repository Guidelines

## Project Structure & Module Organization

This repository extends ALFWorld with a custom structured-memory text agent. The upstream-style package lives in `alfworld/alfworld/`, with environments, training agents, generators, and planner code. Custom project logic is under `alfworld/my_agent/`: `agent/` contains the framework, structure parser, and action agent; `memory/` and `object/` hold shared state models; `test_agent/` contains runnable evaluation and debugging scripts. Configuration files are in `alfworld/configs/`, executable helper scripts are in `alfworld/scripts/`, and project documentation/assets are in `dosc/`. Generated evaluation reports and run logs are written under `alfworld/my_agent/logs/`; avoid committing new large or transient outputs.

## Build, Test, and Development Commands

Use the project from the repository root unless a command says otherwise.

```bash
conda activate AgentTry
cd alfworld
pip install -e .
```

Installs ALFWorld in editable mode for local development.

```bash
export ALFWORLD_DATA=/home/lyh/AgentTry/alfworld_data
python my_agent/test_agent/run_framework.py --episodes 1 --max-steps 20
```

Runs a quick custom-agent smoke test in the TextWorld environment.

```bash
python my_agent/test_agent/run_framework.py --config configs/base_config.yaml --episodes 10 --max-steps 25
```

Runs a longer evaluation using the default config. Use `--quiet` to reduce step logs.

## Coding Style & Naming Conventions

Write Python with 4-space indentation and keep functions focused around one responsibility. Follow the existing mixed style when editing nearby code: upstream ALFWorld files use snake_case modules and functions, while custom agent classes use names such as `AgentFramework`, `ActionAgent`, and `StructContributeAgentLLM`. Prefer explicit imports, clear JSON/YAML keys, and small helper methods over prompt or memory updates embedded in long control-flow blocks.

## Testing Guidelines

There is no dedicated pytest suite in this checkout. Treat `alfworld/my_agent/test_agent/run_framework.py` as the primary regression check for custom-agent changes. For parser, memory, or action-selection edits, run at least one short smoke test and inspect generated reports in `alfworld/my_agent/logs/eval_reports/`. Name new test or debug scripts descriptively, for example `debug_obs.py` or `eval_agent.py`.

## Commit & Pull Request Guidelines

Recent commits use short, direct messages, often in Chinese, for example `README.md文档完善，项目文件框架完善` or `update`. Prefer more specific summaries when possible: `Improve memory priority updates` or `完善 README 运行说明`. Pull requests should describe the behavior change, list the commands run, note required data/API configuration, and include report snippets or screenshots when evaluation results change.

## Security & Configuration Tips

Keep API credentials in a local `.env` with `OPENAI_API_KEY` and optional `OPENAI_BASE_URL`; never commit real keys. Set `ALFWORLD_DATA` explicitly so experiments use the intended dataset path. Avoid committing downloaded datasets, model checkpoints, cache directories, or regenerated PowerPoint/build artifacts.

---
> Source: [lyh843/StructMem-Agent](https://github.com/lyh843/StructMem-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
