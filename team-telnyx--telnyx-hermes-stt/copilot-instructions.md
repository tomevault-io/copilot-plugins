## telnyx-hermes-stt

> Telnyx STT provider contribution for Hermes Agent — integrates into tools/transcription_tools.py.


# Telnyx Hermes STT Provider

Use this skill when integrating or validating the Telnyx Speech-to-Text provider for Hermes.

## Architecture

This is **not** a standalone plugin. Hermes handles STT through built-in
providers dispatched in `tools/transcription_tools.py`. This repo contains the
Telnyx provider function and tests, ready to be contributed upstream.

## Provider details

| Field | Value |
|-------|-------|
| Provider ID | `telnyx` |
| Endpoint | `https://api.telnyx.com/v2/ai/audio/transcriptions` |
| Default model | `openai/whisper-large-v3-turbo` |
| Protocol | OpenAI-compatible (`multipart/form-data`) |
| Auth | `TELNYX_API_KEY` (Bearer) |
| Base URL override | `TELNYX_STT_BASE_URL` env var |
| Language override | `TELNYX_STT_LANGUAGE` env var (ISO-639-1, default `en`) |

## Integration

See `README.md` for step-by-step instructions on adding the Telnyx STT provider
to `tools/transcription_tools.py` in hermes-agent. No new dependencies are
required — the existing `openai` package is reused with the Telnyx base URL.

## Configure

```bash
export TELNYX_API_KEY="***"
```

Optional:

```bash
export TELNYX_STT_BASE_URL="https://api.telnyx.com/v2/ai"
export TELNYX_STT_LANGUAGE="en"
```

## Running tests

```bash
# No credentials needed
python -m pytest tests/test_telnyx_stt_static.py tests/test_telnyx_stt_runtime.py -q

# Live test (requires TELNYX_API_KEY)
export TELNYX_API_KEY=***
python -m pytest tests/test_telnyx_stt_live.py -q
```

## Notes

- No `pip install` needed — the provider function is copy-pasted into hermes-agent.
- Local tests use a mock OpenAI client; live tests hit the real Telnyx API.
- The endpoint override (`TELNYX_STT_BASE_URL`) is covered by tests.

---
> Source: [team-telnyx/telnyx-hermes-stt](https://github.com/team-telnyx/telnyx-hermes-stt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
