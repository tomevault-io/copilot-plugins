## hermes-model-fallback

> >


# Hermes Model Fallback Skill

Automatically switches AI models when HTTP 429 / 503 / Timeout errors occur.
Retries the **same model up to 2 times**, then moves to the next model in the list.

---

## Quick Start (5 minutes)

### Step 1 — Configure your models

Open `scripts/config.json` and fill in your API keys and preferred model order:

```json
{
  "models": [
    {
      "provider": "gemini",
      "model": "gemini-1.5-pro",
      "api_key": "YOUR_GEMINI_KEY"
    },
    {
      "provider": "openai",
      "model": "gpt-4o",
      "api_key": "YOUR_OPENAI_KEY"
    },
    {
      "provider": "claude",
      "model": "claude-3-5-sonnet-20241022",
      "api_key": "YOUR_ANTHROPIC_KEY"
    }
  ],
  "retry_limit": 2,
  "timeout_seconds": 30
}
```

> ✅ You only need to fill in the providers you actually use.  
> ✅ The order in the list = the fallback order (first = primary).

---

### Step 2 — Use the Python version

```python
from scripts.fallback_client import FallbackClient

client = FallbackClient("scripts/config.json")
response = client.chat("Tell me a joke about robots.")
print(response)
```

### Step 3 — Use the JavaScript version

```javascript
const { FallbackClient } = require('./scripts/fallbackClient');

const client = new FallbackClient('./scripts/config.json');
client.chat("Tell me a joke about robots.").then(console.log);
```

---

## How Fallback Works

```
User Request
    │
    ▼
[Model 1 - Primary]
    ├── ✅ Success → return response
    ├── ❌ 429/503/Timeout → retry (attempt 2)
    │       ├── ✅ Success → return response
    │       └── ❌ Still failing → retry (attempt 3 = final)
    │               ├── ✅ Success → return response
    │               └── ❌ Still failing ──────────────────┐
    │                                                      ▼
    │                                          [Model 2 - Fallback #1]
    │                                              ├── ✅ Success → return
    │                                              └── ❌ Failing (3x) ──┐
    │                                                                     ▼
    │                                                         [Model 3 - Fallback #2]
    │                                                              ├── ✅ Success
    │                                                              └── ❌ All failed → raise error
```

**Retry logic:** Each model gets **2 retries** (3 total attempts) before switching.  
**Errors handled:** `HTTP 429`, `HTTP 503`, `Timeout`, `Connection Error`.  
**Other errors** (e.g. 401 Unauthorized, bad prompt) → fail immediately, no retry.

---

## Supported Providers

| Provider   | Value in config  | Docs |
|------------|-----------------|------|
| Google Gemini | `"gemini"`   | [Gemini API](https://ai.google.dev) |
| OpenAI     | `"openai"`      | [OpenAI API](https://platform.openai.com) |
| Anthropic Claude | `"claude"` | [Claude API](https://docs.anthropic.com) |
| Mistral    | `"mistral"`     | [Mistral API](https://docs.mistral.ai) |
| Groq       | `"groq"`        | [Groq API](https://console.groq.com) |
| Ollama (local) | `"ollama"`  | [Ollama](https://ollama.com) |

> Want to add a new provider? See `references/add-provider.md`.

---

## Files in This Skill

```
hermes-model-fallback/
├── SKILL.md                    ← You are here
├── scripts/
│   ├── config.json             ← Your model list & API keys
│   ├── fallback_client.py      ← Python client (ready to use)
│   └── fallbackClient.js       ← JavaScript client (ready to use)
└── references/
    ├── add-provider.md         ← How to add a new AI provider
    └── troubleshooting.md      ← Common errors & fixes
```

---

## Tips

- **Keep API keys safe**: Use environment variables instead of hardcoding.  
  Set `"api_key": "ENV:GEMINI_API_KEY"` and the client reads from `os.environ`.
- **Ollama (local model)** requires no API key — great as a final offline fallback.
- **Timeout** default is 30 seconds. Lower it if you want faster switching.
- Read `references/troubleshooting.md` if you hit unexpected errors.

---
> Source: [Cheerhuan/hermes-model-fallback](https://github.com/Cheerhuan/hermes-model-fallback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
