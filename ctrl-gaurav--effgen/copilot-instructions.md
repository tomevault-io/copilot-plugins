## effgen

> The `GeminiAdapter` calls Google's Generative Language API. effGen ships

# Gemini & Gemma in effGen

The `GeminiAdapter` calls Google's Generative Language API. effGen ships
a small registry of free / cheap text-out models so you can pick one
without leaving the framework.

## Quick start

```python
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.models.base import GenerationConfig

model = GeminiAdapter(model_name="gemini-3.1-flash-lite")
model.load()
result = model.generate(
    "Summarise the second law of thermodynamics in one sentence.",
    config=GenerationConfig(max_tokens=64, temperature=0.2),
)
print(result.text)
```

The adapter reads `GOOGLE_API_KEY` from `~/.effgen/.env` (or the project's
`.env`) automatically when `python-dotenv` has been initialized; you can
also pass `api_key=...` explicitly.

## Recommended models (text-out only)

```python
import effgen

# Models that are reliably callable on the free tier today.
for m in effgen.gemini_recommended_models(tier="free"):
    print(m["id"], m["family"], "RPM=", m["rpm"], "RPD=", m["rpd"])

# All registered models (free + premium):
effgen.gemini_recommended_models(tier="all")

# Inspect a single model:
effgen.gemini_model_info("gemini-3.1-flash-lite")  # alias resolves
```

| Model                                | Family       | Tier    | RPM | TPM     | RPD    | Tools | Thinking | Grounding |
|--------------------------------------|--------------|---------|-----|---------|--------|-------|----------|-----------|
| `gemini-3.1-flash-lite`              | flash-lite   | free    | 15  | 250 K   | 500    | yes   | yes      | no¹       |
| `gemini-3-flash-preview`             | flash        | free    | 5   | 250 K   | 20     | yes   | yes      | yes       |
| `gemini-3-pro-preview`               | pro          | premium | —   | —       | —      | yes   | yes      | yes       |
| `gemini-3.1-pro-preview`             | pro          | premium | —   | —       | —      | yes   | yes      | yes       |
| `gemini-2.5-flash-lite`              | flash-lite   | free    | 10  | 250 K   | 20     | yes   | no       | yes       |
| `gemini-2.5-flash`                   | flash        | free    | 5   | 250 K   | 20     | yes   | no       | yes       |
| `gemini-2.5-pro`                     | pro          | premium | —   | —       | —      | yes   | yes      | yes       |
| `gemini-2.0-flash`                   | flash        | premium | —   | —       | —      | yes   | no       | yes       |
| `gemini-2.0-flash-lite`              | flash-lite   | premium | —   | —       | —      | yes   | no       | yes       |
| `gemma-4-26b-a4b-it` / `gemma-4-31b-it` | gemma     | free    | 15  | unlim.  | 1 500  | no    | no       | no        |

¹ Google Search grounding hits quota on the free-tier for `gemini-3.1-flash-lite`. Use `gemini-2.5-flash` or higher for grounding.

Limits reflect Google's free-tier defaults as of 2026-04-25 — check
[ai.google.dev/gemini-api/docs/rate-limits](https://ai.google.dev/gemini-api/docs/rate-limits)
for the current numbers.

## Thinking budget (extended reasoning)

Gemini 2.5-Pro and the full Gemini 3.x family support an optional internal
reasoning step ("thinking") before producing an answer. You can control it
via two `GenerationConfig` fields:

| Field | Type | Default | Meaning |
|---|---|---|---|
| `thinking_budget` | `int \| None` | `None` | `None` = field not sent (model decides). `0` = disabled. `> 0` = token budget for reasoning. |
| `include_thoughts` | `bool` | `False` | When `True`, the reasoning trace is surfaced in `result.metadata["thinking"]`. |

```python
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.models.base import GenerationConfig

model = GeminiAdapter(model_name="gemini-3.1-flash-lite")
model.load()

result = model.generate(
    "A train travels 120 km in 1.5 hours. What is its average speed?",
    config=GenerationConfig(
        thinking_budget=8192,
        include_thoughts=True,
        max_tokens=512,
    ),
)
print("Thinking:", result.metadata.get("thinking", "(none)"))
print("Answer:", result.text)
print("Thinking tokens:", result.metadata["thoughts_token_count"])
```

**Notes:**
- Models that don't support thinking (Gemma, Gemini 2.5-Flash-Lite, Gemini 2.0)
  silently ignore `thinking_budget`. No error is raised.
- `include_thoughts=False` (default) still uses the thinking budget internally
  but doesn't return the trace, saving output tokens.
- The thinking trace can be long. Set `include_thoughts=True` only when you
  need to inspect reasoning.

### Models with thinking support

| Model | Thinking | Notes |
|---|---|---|
| `gemini-3.1-flash-lite` | yes | Free tier, 15 RPM / 500 RPD |
| `gemini-3-flash-preview` | yes | Free tier, 5 RPM / 20 RPD |
| `gemini-3-pro-preview` | yes | Paid only |
| `gemini-2.5-pro` | yes | Paid only |
| `gemini-2.5-flash` | no | Free tier |
| `gemini-2.5-flash-lite` | no | Free tier |
| `gemma-*` | no | Open weights, generous free-tier RPD |

## Google Search grounding

When `GenerationConfig.grounding=True` and the model supports it, the adapter
activates Gemini's built-in Google Search tool. The model fetches live web
results and attributes its answer to real URLs returned in
`result.metadata["grounding_chunks"]`.

```python
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.models.base import GenerationConfig

model = GeminiAdapter(model_name="gemini-2.5-flash")
model.load()

result = model.generate(
    "What's a major news headline from this week?",
    config=GenerationConfig(grounding=True, max_tokens=512),
)
print(result.text)
for chunk in result.metadata["grounding_chunks"]:
    print("Source URL:", chunk["url"])
    print("Title:     ", chunk["title"])
model.unload()
```

**Notes:**
- Only `gemini-2.5-flash` and higher reliably support grounding on free-tier
  keys. The lite model (`gemini-3.1-flash-lite`) hits quota on the
  grounding endpoint.
- `grounding=False` (default): `metadata["grounding_chunks"]` is always
  an empty list.
- Grounding and user-supplied `tools=` cannot be combined in the same call
  (Google restriction). Use one or the other.

## Files API (upload + reference)

The `upload_file()` helper wraps the Gemini Files API, returning a `FileRef`
that you can pass directly to `generate()` via the `files=` argument. The
model can then read the file content before answering your prompt.

```python
from effgen.models.gemini_files import upload_file
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.models.base import GenerationConfig

# Upload once, reuse in multiple calls
ref = upload_file("paper.pdf")          # auto-detects MIME type
# ref = upload_file("report.txt", mime_type="text/plain", display_name="Q1 Report")

model = GeminiAdapter(model_name="gemini-3.1-flash-lite")
model.load()

result = model.generate(
    "Summarise the key findings of this document in three bullet points.",
    config=GenerationConfig(max_tokens=256),
    files=[ref],
)
print(result.text)
model.unload()
```

`FileRef` fields:

| Field | Type | Description |
|---|---|---|
| `uri` | `str` | `https://generativelanguage.googleapis.com/…` URI |
| `mime_type` | `str` | MIME type (auto-detected or overridden) |
| `display_name` | `str` | Human-readable label (defaults to file name) |
| `raw` | `Any` | Raw SDK file object for low-level access |

**Limits:**
- 2 GB per file (Google hard limit).
- Uploaded files are stored server-side for **48 hours**.
- Supported types: `text/plain`, `application/pdf`, `image/*`, `audio/*`,
  `video/*`, and others listed in Google's Files API docs.
- Files are owned by the API key — the same key can list/delete them via
  `client.files.list()` / `client.files.delete(name=...)` if needed.

## Rate-limit handling

`GeminiAdapter` honors the `retry_delay` Google attaches to 429
`ResourceExhausted` errors. If you trip the per-minute quota the adapter
sleeps for the duration the SDK suggested, retries up to
`MAX_RATE_LIMIT_RETRIES` (default 5), and only then re-raises. Transient
5xx / `DEADLINE_EXCEEDED` errors fall back to exponential backoff.

For tight free-tier loops, prefer Gemma models (30 RPM, 14 400 RPD) or
batch your work with longer sleeps between bursts.

## Tool calling

Native function calling works end-to-end for Gemini 2.0+ and Gemini 3.x.
Gemma models do **not** support function calling; pair them with the
ReAct strategy or no tools.

```python
from effgen.core.agent import Agent, AgentConfig
from effgen.models.gemini_adapter import GeminiAdapter
from effgen.tools.builtin import Calculator

model = GeminiAdapter(model_name="gemini-3.1-flash-lite")
model.load()
agent = Agent(config=AgentConfig(
    name="GeminiAgent",
    model=model,
    tools=[Calculator()],
    max_iterations=6,
))
print(agent.run("Use the calculator to compute (137 * 248) - 1024.").output)
```

Under the hood the adapter:

1. Converts effGen tool specs (OpenAI dict format) into Gemini
   `FunctionDeclaration` objects, stripping JSON-Schema fields Google
   doesn't accept (`minLength`, `maximum`, etc.).
2. Re-emits Gemini's structured `function_call` parts as Qwen-style
   `<tool_call>{...}</tool_call>` tokens so the agent's native parser
   can dispatch them with no Gemini-specific code path.

## Model aliases

These short forms resolve to the canonical ID in the registry:

| Alias                              | Canonical                       |
|------------------------------------|---------------------------------|
| `gemini-3.1-flash-lite-preview`¹   | `gemini-3.1-flash-lite`         |
| `gemini-3-flash-lite`              | `gemini-3.1-flash-lite`         |
| `gemini-3-flash`                   | `gemini-3-flash-preview`        |
| `gemini-3-pro`                     | `gemini-3-pro-preview`          |
| `gemini-3.1-pro`                   | `gemini-3.1-pro-preview`        |
| `gemini-flash-lite-latest`         | `gemini-3.1-flash-lite`         |

¹ The older `-preview` flash-lite ID was retired by Google (returns 404); it is
kept as a backward-compatible alias that resolves to the GA `gemini-3.1-flash-lite`.

---
> Source: [ctrl-gaurav/effGen](https://github.com/ctrl-gaurav/effGen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
