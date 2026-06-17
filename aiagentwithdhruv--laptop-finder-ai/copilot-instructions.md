## 40-cache-redis

> Redis caching and queue rules


Redis usage:
- Use Redis for caching, rate limiting, session-like ephemeral state, queues, and short-lived coordination.
- Choose TTLs intentionally.
- Cache only what has a clear performance benefit.
- Make cache invalidation explicit.

Rules:
- Wrap Redis access in dedicated utilities/services.
- Use stable key naming conventions.
- Document key patterns if introducing new families of keys.
- Prefer idempotent worker/job behavior.
- Use retries carefully and avoid infinite retry loops.

Caching expectations:
- Cache expensive reads and repeated metadata lookups where appropriate.
- Do not cache highly sensitive data unless encrypted and justified.
- Do not rely on cache as the only source of truth for critical persistent state.

Do not:
- Scatter raw Redis calls across the codebase.
- Store large unbounded payloads without TTL or cleanup strategy.
- Mix queue semantics and cache semantics without clarity.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
