## aegis-osint-ai

> Defensive OSINT investigation framework for authorized passive collection, evidence persistence, provider-backed analysis, and report rendering.


# Aegis v2

Aegis v2 is a defensive OSINT investigation framework. Use it for authorized,
passive investigation workflows, normalized findings, and analyst-facing
reports.

## Supported Workflows

- Run and extend the FastAPI backend.
- Add passive OSINT plugins under `backend/plugins/`.
- Add provider integrations under `backend/providers/`.
- Persist investigations, targets, findings, and reports.
- Render JSON, Markdown, and briefing reports from persisted findings.
- Check Kali Linux operator workstation compatibility with
  `scripts/kali_compatibility.py`.

## Guardrails

- Do not run or re-enable files in `legacy/quarantine/`.
- Do not add exploitation, session replay, credential replay, browser
  fingerprint cloning, payload execution, or banking-target automation to the
  v2 runtime.
- Do not commit API keys, tokens, `.env` files, or extracted credentials.
- Keep integrations configuration-driven and environment-backed.

## Quick Start

```bash
uvicorn backend.api.app:create_app --factory --reload
python3 scripts/kali_compatibility.py --json
python3 -m pytest backend/tests
```

---
> Source: [steelai-bot/Aegis-OSINT-AI](https://github.com/steelai-bot/Aegis-OSINT-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
