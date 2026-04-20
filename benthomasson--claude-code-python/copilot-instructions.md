## claude-code-python

> This is an educational reimplementation of Claude Code in Python. The goal is to make the core agentic tool-use loop easy to understand — how an LLM decides to call tools, how the host executes them, and how results flow back. It uses the Anthropic Claude API via Google Cloud Vertex AI (not the direct Anthropic API).

# CLAUDE.md

## Project overview

This is an educational reimplementation of Claude Code in Python. The goal is to make the core agentic tool-use loop easy to understand — how an LLM decides to call tools, how the host executes them, and how results flow back. It uses the Anthropic Claude API via Google Cloud Vertex AI (not the direct Anthropic API).

## Architecture

- `main.py` — Entry point. Runs the interactive conversation loop. Sends messages to Claude via Vertex AI, handles tool use responses, and manages conversation history.
- `tools.py` — Defines the tools Claude can use (read_file, write_file, run_command) and contains the logic to execute them.

## Key technical decisions

- Uses `anthropic[vertex]` SDK with `AnthropicVertex` client, not the base `Anthropic` client.
- Model: `claude-sonnet-4-20250514` (or latest available on Vertex).
- The tool loop runs synchronously: send message -> check for tool_use -> execute tool -> append result -> repeat until Claude responds with text only.
- No streaming. Responses are returned as complete messages.
- Tools run without user confirmation (this is a simple prototype).

## Dependencies

- `anthropic[vertex]` — Claude SDK with Vertex AI support

## Environment variables

- `GOOGLE_CLOUD_PROJECT` — Required. Your GCP project ID.
- `GOOGLE_CLOUD_REGION` — Required. Vertex AI region (e.g. `us-east5`).

## Commands

```bash
# Run
python main.py

# Install deps
pip install -r requirements.txt
```

## Style

- Clarity over cleverness. This is a learning tool — code should be obvious, not elegant.
- No classes unless truly needed — plain functions are fine.
- Prefer verbose/explicit code over abstractions. A reader should be able to follow the tool-use loop without jumping between files.
- Comments should explain *why*, not *what* — but do comment the key steps of the agentic loop since that's the whole point of the project.
- Standard library where possible; only external dep is the Anthropic SDK.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benthomasson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
