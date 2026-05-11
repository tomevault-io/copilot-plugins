## cursor-azure-gpt-5

> This repository is an LLM proxy for Cursor.

# AGENTS.md

## Project Context

This repository is an LLM proxy for Cursor.

If you are running inside Cursor and using this project's configured endpoint, your LLM requests are going through this proxy rather than directly to OpenAI or Azure. Keep that in mind when reading, debugging, or changing the code: request shaping, forwarding, streaming adaptation, auth, and logging behavior in this repo may directly affect Cursor agent/chat behavior.

## What This Proxy Does

- Accepts incoming Cursor-compatible requests
- Adapts them to Azure's Responses API
- Forwards them to the configured Azure deployment
- Adapts streamed responses back into the format Cursor expects

## Practical Reminder

When investigating model behavior from Cursor, assume this proxy is part of the execution path and a likely place to inspect first.

---
> Source: [gabrii/Cursor-Azure-GPT-5](https://github.com/gabrii/Cursor-Azure-GPT-5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
