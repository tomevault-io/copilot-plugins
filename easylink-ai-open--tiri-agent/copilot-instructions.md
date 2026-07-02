## tiri-agent

> This repository is the Tiri local agent product built on top of

# Tiri Agent product repo

This repository is the Tiri local agent product built on top of
`agent-runtime`.

## Boundaries

- Import only the Python standard library, `agent_runtime`, modules within this
  repository, and dependencies declared by this repository.
- Do not import legacy application packages from another repository.
- Do not expose provider clients. Tiri backend config supplies
  `agent_runtime.LLMConfig`; `Agent` owns provider HTTP calling internally.
- Do not reimplement runtime primitives such as tool registries, prompt
  providers, or cache strategies. Import them from `agent_runtime` and
  configure them here.
- Tiri owns sandbox implementations and sandbox tools. Expose them to the
  runtime only as ordinary `agent_runtime.ToolRegistry` entries.
- This package owns the local Tiri entrypoints: `tiri` for CLI chat and
  `tiri server start` for the local FastAPI + Next.js development shell.
  Keep those entrypoints thin; they should compose `TiriBackend`, not
  reimplement the agent loop.
- Keep local single-user persistence in this package. The default durable store
  is SQLite under `~/.tiri/`; multi-tenant cloud concerns still stay out.
- Keep runtime policy in Tiri-specific modules here, while mechanism stays
  in `agent-runtime`.

## Module Map

- `agent/` — `TiriBackend`, backend config, and collaboration-mode policy.
- `prompts/` — Tiri prompt-provider composition around runtime prompt
  primitives. Default prompt assembly owns the Tiri base prompt, runtime
  environment prompt, local profile prompt, and local skills index. Do not move
  this policy into `agent-runtime`.
- `tools/` — tool registry composition and concrete tool sets.
- `channels/` — local channel settings, QR registration/login, targets,
  provider adapters, listener gateway, runtime boundary, inbound routing, and
  SQLite outbox worker. Real adapters/listeners must stay here without
  importing legacy application packages.
- `sandbox/` — Tiri-owned sandbox interface and implementations. Runtime
  only sees the resulting tools. Current backends are `local` and
  `local-docker`.
- `server/` — FastAPI app, SQLite persistence, and local process runner for API
  + Next.js.
- `cli/` — local command entrypoint.

Top-level `backend.py`, `config.py`, and `collaboration.py` are compatibility
re-export modules. Do not add new implementation there.

## Development

Run tests from the repository root with:

```bash
uv pip install -e ../agent-runtime -e ".[server]" --python .venv/bin/python
.venv/bin/python -m pytest tests -q
```

---
> Source: [easylink-ai-open/tiri-agent](https://github.com/easylink-ai-open/tiri-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-02 -->
