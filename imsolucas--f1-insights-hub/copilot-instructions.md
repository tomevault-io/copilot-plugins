## f1-insights-hub

> Rules for OpenAI API integration used for generating race summaries, chat-based insights, and explanations. These rules apply to all AI/LLM functionality in the application.


Scope

Rules for OpenAI API integration used for generating race summaries, chat-based insights, and explanations. These rules apply to all AI/LLM functionality in the application.

Non‑negotiables
- Never hardcode OpenAI API keys; always use environment variables.
- All AI responses must be cached to reduce costs and improve performance.
- Rate limit all OpenAI API calls to prevent quota exhaustion.
⸻
1) Use Cases

- Race summaries: Generate human-readable summaries of race results and key moments.
- Chat-based insights: Answer user questions about F1 races, drivers, teams, and statistics.
- Explanations: Provide context and explanations for F1 concepts, rules, and historical events.
- No predictions: Do not generate predictions about future race outcomes or driver performance.
⸻
2) API Integration

- Abstract OpenAI behind a service layer (e.g., `lib/ai/openai-service.ts`).
- Never hardcode API keys; use environment variables (OPENAI_API_KEY).
- Use OpenAI Node.js SDK with proper error handling.
- Configure API client with timeout and retry logic.
- Version pinning: specify OpenAI SDK version in package.json to avoid breaking changes.
⸻
3) Rate Limiting & Quotas

- Implement rate limiting per user/IP to prevent abuse.
- Respect OpenAI rate limits (requests per minute, tokens per minute).
- Use exponential backoff for rate limit errors (429).
- Track usage per user/session to enforce usage limits.
- Set maximum tokens per request to control costs.
⸻
4) Prompting Guidelines

- Ground responses in real F1 data: always provide context from database/external APIs.
- Use system prompts to define AI role and constraints (e.g., "You are an F1 expert assistant...").
- Include relevant data in user prompts (race results, driver stats, etc.) rather than relying on AI's training data.
- Avoid hallucinations: structure prompts to reference specific data sources.
- Use few-shot examples for consistent output format.
- Set temperature appropriately: lower (0.3-0.5) for factual summaries, slightly higher (0.7) for conversational insights.
⸻
5) Caching Strategy

- Cache all AI-generated responses with appropriate TTL:
  - Race summaries: cache indefinitely (historical data doesn't change).
  - Chat insights: cache for shorter periods (1-24 hours) or per session.
- Cache key format: `ai:race-summary:{raceId}` or `ai:chat:{sessionId}:{queryHash}`.
- Invalidate cache only when source data changes (e.g., race results updated).
- Store cache in Redis or database; include timestamp and metadata.
⸻
6) Cost Management

- Enforce usage limits per user/session (e.g., max tokens per day).
- Monitor token usage: log tokens consumed per request.
- Use appropriate models: GPT-3.5-turbo for most tasks, GPT-4 only when necessary.
- Set max_tokens in requests to prevent excessive generation.
- Track costs per feature/endpoint to identify optimization opportunities.
⸻
7) Error Handling

- Handle API errors gracefully: network failures, rate limits, invalid responses.
- Provide fallback responses when AI service is unavailable (e.g., return cached data or generic message).
- Log all errors with context: request details, error type, retry attempts.
- Never expose OpenAI API errors directly to users; translate to user-friendly messages.
- Implement circuit breaker pattern: stop calling OpenAI after repeated failures.
⸻
8) Response Validation

- Validate AI responses before returning to users (check for required fields, format).
- Sanitize AI output: remove any potentially harmful content or formatting issues.
- Set response timeouts: fail fast if OpenAI takes too long (> 30 seconds).
- Verify AI responses align with source data (e.g., race summary matches actual race results).
⸻
9) Security & Privacy

- Never send sensitive user data to OpenAI unless necessary and anonymized.
- Do not log full prompts/responses containing PII in production logs.
- Review OpenAI data usage policy: ensure compliance with data retention settings.
- Consider using OpenAI API with data privacy options if handling sensitive data.
⸻
10) Testing

- Mock OpenAI API calls in unit tests; never call real API in tests.
- Test error scenarios: rate limits, network failures, invalid responses.
- Test caching behavior: verify cache hits/misses, TTL expiration.
- Test prompt engineering: verify prompts produce expected output formats.
- Integration tests: use OpenAI test mode or mock responses for E2E tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imsolucas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
