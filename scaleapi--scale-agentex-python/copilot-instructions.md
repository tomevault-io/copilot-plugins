## 30-cli-and-commands

> Guidance for working with the agentex CLI and commands


The `agentex` CLI exposes:

- `agentex agents` for get/list/run/build/deploy agents
- `agentex tasks` for get/list/delete tasks
- `agentex secrets` for sync/get/list/delete secrets
- `agentex uv` as a UV wrapper with AgentEx-specific enhancements
- `agentex init` to initialize new agent projects

Development tips:

- For agent development, use `agentex agents run --manifest manifest.yaml`
- For debugging, append `--debug-worker` and optionally `--debug-port 5679`

---
> Source: [scaleapi/scale-agentex-python](https://github.com/scaleapi/scale-agentex-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
