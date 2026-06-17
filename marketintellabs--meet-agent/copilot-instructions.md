## meet-agent

> Join Google Meet and Zoom calls as an AI agent that listens, reasons, and speaks. Uses any OpenAI-compatible LLM, pluggable STT/TTS providers, and optional real-time avatar rendering.

# MeetAgent — AI Meeting Participant

Join Google Meet and Zoom calls as an AI agent that listens, reasons, and speaks. Uses any OpenAI-compatible LLM, pluggable STT/TTS providers, and optional real-time avatar rendering.

## Requirements

- Python 3.10+
- A DeepInfra, OpenAI, or compatible LLM API key
- Chromium (installed automatically via Playwright)

## Quick Start

```bash
pip install meet-agent
playwright install chromium
meet-agent join "https://meet.google.com/abc-defg-hij" --name "My Agent"
```

## Configuration

Set via environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `LLM_API_KEY` | (required) | API key for the LLM provider |
| `LLM_API_BASE` | `https://api.deepinfra.com/v1/openai` | OpenAI-compatible endpoint |
| `LLM_MODEL` | `nvidia/Nemotron-Mini-4B-Instruct` | Model to use |
| `STT_PROVIDER` | `deepinfra` | `deepinfra`, `openai`, or `faster-whisper` |
| `TTS_PROVIDER` | `openai` | `openai` or `piper` |
| `AGENT_NAME` | `MeetAgent` | Display name in the meeting |

## API Server

```bash
meet-agent serve --port 8080

# Create a session
curl -X POST http://localhost:8080/sessions \
  -H "Content-Type: application/json" \
  -d '{"meeting_url": "https://meet.google.com/abc-defg-hij"}'
```

## Docker

```bash
docker compose up
```

## Links

- GitHub: https://github.com/MarketIntelLabs/meet-agent
- Issues: https://github.com/MarketIntelLabs/meet-agent/issues

---
> Source: [marketintellabs/meet-agent](https://github.com/marketintellabs/meet-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
