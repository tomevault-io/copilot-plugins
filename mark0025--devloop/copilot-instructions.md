## devloop

> • LLM inference runs on a remote Ollama server; the endpoint is supplied via `$OLLAMA_URL` in `.env` (never hard-coded).

# Default Infra & Tools

• LLM inference runs on a remote Ollama server; the endpoint is supplied via `$OLLAMA_URL` in `.env` (never hard-coded).

• Agent logic is written in Python 3.11 and structured with LangChain. Dependency management uses Poetry (`pyproject.toml`).

• Event orchestration is handled by n8n webhooks. GitHub _push_ is the canonical trigger; additional schedules or manual triggers must route through n8n as well.

• All scriptable paths (data, repos, logs) are parametrised through environment variables or config files. Interactive path selection is **not** allowed.

• GPU compute is provided by RunPod. Default policy:
– Use _Flex_ workers for burst workloads.
– Auto-stop any on-demand pod that has been idle for > 10 minutes.

• The agent may spin up additional pods when queue latency exceeds 5 seconds, but must tag each pod with the GitHub SHA it is serving.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mark0025)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mark0025)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
