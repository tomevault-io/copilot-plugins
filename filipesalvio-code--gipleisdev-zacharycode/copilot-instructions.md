## gipleisdev-zacharycode

> Integration module constraints (HTTP clients, rate limits, retries, credentials)


# Integrations (gipleis/integrations/)

- **HTTP clients:** Use `httpx` (async). Set explicit timeouts on every request.
- **Rate limits:** Respect API rate limits; implement exponential backoff with jitter.
- **Retries:** Max 3 retries with increasing delay. Log every retry with context (URL, attempt number, error).
- **Credentials:** Never hardcode. Use environment variables via `python-dotenv`. Reference `.env.example` for required keys.
- **Error handling:** Catch `httpx.HTTPStatusError`, `httpx.TimeoutException`, and `httpx.ConnectError` explicitly. No naked `try/except`.

Jules API rate limits depend on tier (FREE: 10 req/min, PRO: 60 req/min, ULTRA: 200 req/min). Always check `X-RateLimit-Remaining` headers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filipesalvio-code) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
