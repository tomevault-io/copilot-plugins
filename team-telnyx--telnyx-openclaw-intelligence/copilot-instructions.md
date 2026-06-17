## telnyx-openclaw-intelligence

> Add Telnyx as an OpenClaw text-inference provider with TELNYX_API_KEY auth and OpenAI-compatible webchat completions support.


# Telnyx Intelligence Provider (AIF-127)

Native Telnyx AI text-inference plugin for OpenClaw. Registers the `telnyx` provider for OpenAI-compatible chat completions — no LiteLLM proxy needed.

## Install

```bash
openclaw plugins install /path/to/telnyx-openclaw-intelligence
```

Or from your project root:

```bash
cd ~/projects/telnyx-openclaw-intelligence
npm install && npm run build
openclaw plugins install .
```

## Configure

```bash
export TELNYX_API_KEY="KEY_your_api_key_here"
```

## Use

```bash
/model telnyx/meta-llama/Meta-Llama-3.1-70B-Instruct
```

## Key Models

- `telnyx/meta-llama/Meta-Llama-3.1-70B-Instruct` (default)
- `telnyx/openai/gpt-5`
- `telnyx/anthropic/claude-opus-4-6`
- `telnyx/google/gemini-2.5-flash`
- `telnyx/Qwen/Qwen3-235B-A22B`
- `telnyx/moonshotai/Kimi-K2.6`

## Source

https://github.com/team-telnyx/telnyx-openclaw-intelligence

---
> Source: [team-telnyx/telnyx-openclaw-intelligence](https://github.com/team-telnyx/telnyx-openclaw-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
