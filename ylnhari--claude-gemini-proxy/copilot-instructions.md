## claude-gemini-proxy

> > **Setup:** copy `AGENTIC.local.md.example` → `AGENTIC.local.md` (gitignored) for machine-local notes. It is read only when the active host has a verified adapter that explicitly loads it. Personal/global preferences live in your own agent's global instructions file.

# Agent Instructions — claude-gemini-proxy

> **Setup:** copy `AGENTIC.local.md.example` → `AGENTIC.local.md` (gitignored) for machine-local notes. It is read only when the active host has a verified adapter that explicitly loads it. Personal/global preferences live in your own agent's global instructions file.

## Role
Maintain a LiteLLM gateway that routes supported client requests to the Google Gemini API. Keep it working, minimal, and explicit about current provider availability and billing.

## Context
Used to route compatible client requests to Gemini through a local gateway. Client-specific launch helpers are optional adapters, not the gateway's core contract.

**Key insight:** This project exists purely as infrastructure — it has no business logic. Changes should be minimal and targeted.

## Architecture

```
litellm_config.yaml      # Model routing config — primary file to edit for model changes
start_proxy.py           # Starts LiteLLM proxy
start_claude.py          # Starts Claude Code pointed at the proxy
scripts/                 # Model availability checker, helpers
all_models.json          # Cached model list from last check
model_checker_results.json
```

## Credentials
- `GEMINI_API_KEY` — the same key you use for your other Gemini projects. Set in environment or `.env`.
- Never commit API keys. Check `.gitignore` before adding files.

## Rules
1. **Verify availability and billing at use time.** Model access, quotas, and pricing are provider/account-specific; do not promise free-tier eligibility from static configuration.
2. **Minimal changes.** This is glue code — don't refactor it unless something is broken.
3. **Model list can go stale.** Run `scripts/` checker to refresh `all_models.json` periodically.
4. When `litellm_config.yaml` is updated, restart the proxy for changes to take effect.

## Running
```bash
# Install (uv recommended)
uv sync
# Start proxy
python start_proxy.py
# In another terminal, start Claude Code via proxy
python start_claude.py
```

---
> Source: [ylnhari/claude-gemini-proxy](https://github.com/ylnhari/claude-gemini-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
