## specmint

> - Primary LLM: qwen2.5:latest (7.6B, Q4_K_M quantization)


# Ollama Integration Rules

## Local LLM Configuration
<ollama_setup>
- Primary LLM: qwen2.5:latest (7.6B, Q4_K_M quantization)
- Endpoint: http://localhost:11434
- Always validate Ollama connectivity before LLM operations
- Implement health checks with /api/tags endpoint
- Support model auto-pull for missing models
- Use deterministic generation with seed parameter
</ollama_setup>

## LLM Client Implementation
<ollama_client_rules>
- Use persistent HTTP connections with keep-alive
- Implement connection pooling (max 4 concurrent connections)
- Set appropriate timeouts: 30s for generation, 5s for health checks
- Include circuit breaker pattern for fault tolerance
- Cache system prompts to reduce redundant calls
- Batch field enrichments when possible
</ollama_client_rules>

## Prompt Engineering
<prompt_guidelines>
- Use structured prompts with clear constraints
- Include schema validation rules in system prompts
- Specify output format (JSON only, no explanations)
- Include seed in prompt for deterministic behavior
- Limit context length to model's window (32k tokens for qwen2.5)
- Implement prompt caching for repeated patterns
</prompt_guidelines>

## Fallback Strategy
<fallback_behavior>
- Always have deterministic fallback for LLM failures
- Log LLM errors without exposing sensitive data
- Gracefully degrade to deterministic-only mode
- Mark manifest with degradation status
- Continue processing other records on individual failures
</fallback_behavior>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/copyleftdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
