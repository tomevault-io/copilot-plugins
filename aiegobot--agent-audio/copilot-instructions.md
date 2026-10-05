## agent-audio

> Agent Audio provides agent-first local audio generation. The initial product

# Repository instructions

Agent Audio provides agent-first local audio generation. The initial product
focus is sound effects in both game and video work, without an additional paid
audio-generation service or a mandatory intermediate audio-selection workflow.
Read [project direction](docs/project-direction.md) before changing scope.

Keep the architecture open to different agents, models, hardware and use cases;
do not turn that direction into an unverified universal-support claim. Prefer
improvements to the proven user flow over speculative adapters or feature count.

## Architecture and working rules

- Keep MCP tools generic. Do not add game-engine-specific parameters to the core protocol.
- Keep Stable Audio implementation details inside backend and runtime modules.
- Add hardware support as backend adapters rather than scattered conditionals.
- Never include third-party model weights in this repository.
- Never bypass gated-model license or authentication flows.
- Prefer verified fallback backends over untested acceleration claims.
- Consider Windows, macOS and Linux when changing the installer.
- Use the open Agent Skills `SKILL.md` format wherever possible.
- Write repository documentation in English. Maintain one default `README.md`; do not add translation copies unless requested.
- Preserve existing user settings, Skills, models and runtimes. Follow [SECURITY.md](SECURITY.md) for conflict handling and security boundaries.

## Product and evidence rules

- Keep routine sound decisions with the agent within the user's requested task. Do not add mandatory creative approvals; do preserve permissions, license acceptance and explicit user constraints.
- Distinguish MCP tool availability, actual generation, implicit Skill use, final integration and listening quality. One passing layer does not prove the next.
- Skill changes affect user behavior. Use [end-to-end validation](docs/end-to-end-validation.md) and verify installed Skill copies before real-work tests.
- Do not claim better sound quality from WAV metadata alone, or mark work as tested when the environment could not run it.
- Do not add paid-cloud fallback, model switching, new hardware support or engine-specific core APIs merely to broaden the first release.

## Important files

- [INSTALL_AGENT.md](INSTALL_AGENT.md): installation, registration and real generation verification.
- `src/agent_audio/installer.py`: Skill installation and client configuration merging.
- `src/agent_audio/runtime.py`, `src/agent_audio/download_models.py`: dedicated runtime and model setup.
- `src/agent_audio/mcp_server.py`: MCP tools.
- `src/agent_audio/storage.py`, `src/agent_audio/process.py`: file preservation and inference process control.
- `tests/`: regression tests that do not require model weights.

## Definition of done for installer changes

Run from the repository root:

```text
uv sync --frozen
uv run --frozen ruff check src install tests
uv run --frozen ruff format --check src install tests
uv run --frozen python -m compileall -q src install
uv run --frozen pytest -q
uv run --frozen python install/bootstrap.py --doctor
```

- Compilation, static checks and unit tests must pass.
- Unit tests must not require model weights.
- `--doctor` must work without network access.
- Merge an existing Cursor `mcp.json`; never replace it wholesale.
- Unsupported accelerators must use a documented backend.
- Report real model generation verification separately from model-free tests.

---
> Source: [AIEGOBOT/agent-audio](https://github.com/AIEGOBOT/agent-audio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-05 -->
