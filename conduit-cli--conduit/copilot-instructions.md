## conduit

> [Gemini CLI](https://github.com/google-gemini/gemini-cli) is Google's command-line interface for Gemini.

# Gemini CLI

[Gemini CLI](https://github.com/google-gemini/gemini-cli) is Google's command-line interface for Gemini.

## Features

- **Large Context Window** — 1M token context window
- **Tool Execution** — Read, write, and execute commands
- **Multiple Models** — Various Gemini models available

## Models

| Model | Best For |
|-------|----------|
| gemini-3-pro-preview | Complex reasoning, large context |
| gemini-3-flash-preview | Balanced performance |

## Build vs Plan Mode

Plan mode for Gemini is prompt-guided (best-effort). The agent follows the prompt guidance but isn't strictly enforced like Claude Code.

Toggle with `Tab` or `Ctrl+4`.

## Tools Available

Similar capabilities to other agents:
- File reading and writing
- Shell command execution
- Code search and analysis

## Installation

```bash
npm install -g @google/gemini-cli
```

## Configuration

Ensure Gemini CLI is in your PATH, or configure the path:

```toml
[tools]
gemini = "/path/to/gemini"
```

---
> Source: [conduit-cli/conduit](https://github.com/conduit-cli/conduit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
