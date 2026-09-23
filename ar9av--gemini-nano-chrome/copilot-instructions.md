## gemini-nano-chrome

> This repo runs Chrome's on-device Gemini Nano model behind a local OpenAI-compatible API. If you're an agent that wants to use it as a model backend (not modify the repo itself), this is what you need.

# AGENTS.md

This repo runs Chrome's on-device Gemini Nano model behind a local OpenAI-compatible API. If you're an agent that wants to use it as a model backend (not modify the repo itself), this is what you need.

## Get it running

```bash
git clone https://github.com/Ar9av/gemini-nano-chrome.git
cd gemini-nano-chrome
npm start
```

This launches Chrome with the required flags set automatically, starts the API on `:8788`, and serves a chat UI on `:8123`. First run downloads a ~4GB model once; after that, startup is near-instant. Requires Chrome 138+, Node 22+, and macOS/Windows/Linux with 22GB free disk.

## Check readiness before calling it

```bash
curl http://localhost:8788/health
```

Always returns HTTP 200. The field that matters is `availability`:

```json
{"status":"ok","model":"gemini-nano","availability":"available"}
```

`availability` is one of `"unavailable"`, `"downloadable"`, `"downloading"`, `"available"`, or `"error: <message>"`. Only `"available"` means a chat completion will succeed without first paying the download cost.

## Call it

Base URL: `http://localhost:8788/v1`. Standard OpenAI chat completions shape, so any OpenAI SDK works, point it at the local base URL with any non-empty `api_key` value, since nothing checks it.

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8788/v1", api_key="not-needed")
resp = client.chat.completions.create(
    model="gemini-nano",
    messages=[{"role": "user", "content": "..."}],
)
```

```bash
curl http://localhost:8788/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "..."}]}'
```

`stream: true` works (SSE, standard `data: {...}\n\ndata: [DONE]\n\n` framing). `response_format: {"type": "json_schema", "json_schema": {"schema": {...}}}` constrains output to match the schema.

## Things to know before you build against it

- **9216-token context window**, input and output combined. Going over it doesn't truncate, the underlying model throws, and that surfaces as a 500 with `QuotaExceededError: The input is too large.` somewhere in `error.message`, not a clean 400. Keep prompts well under the limit or watch for that string.
- **~50 tokens/sec generation**, measured on Apple M4 Pro with the GPU backend. Slower or faster depending on the machine Chrome is running on, there's no API to query expected throughput ahead of time.
- **No conversation state on the server.** There's no `session_id` or equivalent. Send the full message history every time, the server rebuilds a fresh model session from it on each request.
- **The last message must have role `"user"`.** A 400 with `"the last message must have role 'user'"` means you sent something else last (e.g. ending on an assistant turn).
- **1MB request body cap.** Larger bodies get a 413.
- **`temperature`/`top_k` are best-effort.** Only honored when both are set together, and may be silently ignored depending on how Chrome was set up. Don't depend on them changing output in a verifiable way.
- **One local model, one local browser tab.** This isn't built for concurrent high-throughput serving. Expect it to behave more like a single long-lived worker than a scalable API.

## If something's failing

- 500 with a CDP/websocket-flavored message (e.g. mentions a closed tab or connection): the underlying Chrome tab died and is being recreated automatically on the next request. Retry once.
- 404: check the path. Only `GET /health`, `GET /v1/models`, and `POST /v1/chat/completions` exist, no `/v1/completions`, no `/v1/embeddings`.
- Hangs on first request: the model is still downloading (~4GB). Poll `/health` until `availability` is `"available"`.

## Repo map, if you need to look deeper

- `server/index.js`, `server/mapping.js`, `server/page.js`: the API itself
- `web/index.html`: a browser-based chat UI that talks to the model directly, no API server involved
- `extension/`: the same chat UI packaged as an installable Chrome extension
- `README.md`: full setup docs, measured performance numbers, and the Prompt API reference this is built on

---
> Source: [Ar9av/gemini-nano-chrome](https://github.com/Ar9av/gemini-nano-chrome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
