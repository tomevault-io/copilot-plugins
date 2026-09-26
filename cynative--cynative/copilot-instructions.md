## cynative

> **Bifrost provider id:** `gemini`

# Google Gemini (AI Studio)

**Bifrost provider id:** `gemini`

## Quick start

```bash
export GEMINI_API_KEY=...
export CYNATIVE_LLM_PROVIDER=gemini
export CYNATIVE_LLM_MODEL=gemini-2.5-pro
cynative -p "..."
```

## YAML

```yaml
llm:
  provider: gemini
  model: gemini-2.5-pro
  api_key: env.GEMINI_API_KEY
```

## Authentication

Get a Gemini API key from <https://aistudio.google.com/app/apikey>. This is
the AI-Studio-flavored Gemini, separate from Vertex AI — use the `vertex`
provider above for Google Cloud / IAM-authenticated Gemini access.

## Links

- Gemini API docs: <https://ai.google.dev/gemini-api/docs>
- Bifrost Gemini provider source: <https://github.com/maximhq/bifrost/tree/main/core/providers/gemini>

---
> Source: [cynative/cynative](https://github.com/cynative/cynative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
