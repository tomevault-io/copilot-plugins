## heyclaw

> Python 3.12 voice assistant powered by ElevenLabs, Gemini, Mem0, and MCP tools. Server code and tests live in `heyclaw/`; local audio and wake-word code lives in `satellite/app/`, with tests in `satellite/tests/`. `heyclaw/workspace/` contains the instructions, profile, and skills loaded by the assistant at runtime.

# HeyClaw

Python 3.12 voice assistant powered by ElevenLabs, Gemini, Mem0, and MCP tools. Server code and tests live in `heyclaw/`; local audio and wake-word code lives in `satellite/app/`, with tests in `satellite/tests/`. `heyclaw/workspace/` contains the instructions, profile, and skills loaded by the assistant at runtime.

## Configuration

- Requires Python 3.12, `uv`, GNU Make, `ngrok`, and `lsof`.
- Prepare `heyclaw/.env` from `heyclaw/.env.example`. It contains logging settings only.
- Prepare `heyclaw/config.json` from `heyclaw/config.example.json`. It defines server providers, gateway, agent defaults, workspace, memory, and MCP servers.
- Prepare `satellite/.env` and `satellite/config.json` from their example files. They define local logging, audio, wake word, and the ElevenLabs connection.
- Never commit credentials from either component.
- The full development stack exposes the HTTP API on port 8000, the Speech Engine WebSocket on port 3001, and the ngrok tunnel configured by `gateway.publicWsUrl`.

## Unix/WSL Commands

Use only `Makefile` targets from the repository root:

- `make setup`: installs Python 3.12 and dependencies for both components.
- `make backend`: stops local project processes, then starts the backend, Speech Engine, and ngrok tunnel.
- `make satellite`: starts the interactive voice satellite; it requires microphone and audio devices configured in `satellite/config.json`.
- `make kill`: stops processes on the ports used by the project.
- `make check`: formats the code, applies Ruff fixes, and runs type checking. It may modify files.

Always specify a target: the default `help` goal is not defined.

## Windows (Secondary)

Windows uses the separate `Makefile.windows`, backed by `heyclaw/scripts/dev.ps1`. Invoke it explicitly with `make -f Makefile.windows <target>`. It exposes the same active targets as the Linux Makefile and requires PowerShell 7, GNU Make, `uv`, and `ngrok` in `PATH`.

## Working Rules

- Treat `Makefile`, `Makefile.windows`, and each component's `.env.example`, `config.example.json`, and `pyproject.toml` as the operational sources of truth.
- Do not modify either Makefile unless explicitly requested.
- Never expose or commit either component's `.env` or `config.json`.
- Use `make check` for static verification; tests currently have no dedicated `Makefile` target.

---
> Source: [vericle/heyclaw](https://github.com/vericle/heyclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
