## dev-cache-api

> How to use the dev-cache HTTP API


# dev-cache API

In-memory HTTP cache for **trusted dev environments only**. Keys map to URL paths (`users/123` → `/users/123`). Data is lost on restart.

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/*key` | Return stored value. `404` if missing/expired |
| POST | `/*key` | Store request body. Optional `?ttl=<seconds>` |
| PUT | `/*key` | Same as POST |
| DELETE | `/*key` | Remove key. Always `204` |
| HEAD | `/*key` | `200` if exists, `404` otherwise |
| GET | `/_health` | Health check (no auth required) |
| GET | `/_stats` | Key count and uptime |

## Auth

When `DEVCACHE_TOKEN` is set, all routes except `/_health` require:

```
Authorization: Bearer <token>
```

## Examples

```bash
# Set (no TTL)
curl -X POST http://localhost:8080/users/123 \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ada"}'

# Set with 60s TTL
curl -X POST 'http://localhost:8080/users/123?ttl=60' \
  -H 'Content-Type: application/json' \
  -d '{"name":"Ada"}'

# Get / delete
curl http://localhost:8080/users/123
curl -X DELETE http://localhost:8080/users/123
```

```javascript
await fetch('http://localhost:8080/api/data?ttl=300', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ cached: true }),
});
const data = await (await fetch('http://localhost:8080/api/data')).json();
```

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `DEVCACHE_ADDR` | `:8080` | Listen address |
| `DEVCACHE_TOKEN` | _(empty)_ | Bearer token; empty disables auth |
| `DEVCACHE_CORS_ORIGINS` | `*` (no auth) / `""` (with auth) | Comma-separated origins |
| `DEVCACHE_MAX_BODY` | `1048576` | Max POST/PUT body (bytes) |
| `DEVCACHE_READ_TIMEOUT` | `10s` | HTTP read timeout |
| `DEVCACHE_WRITE_TIMEOUT` | `10s` | HTTP write timeout |

## Status codes

`200` (get), `201` (set), `204` (delete), `404` (missing), `401` (unauthorized), `413` (body too large).

## Constraints

- No persistence, pub/sub, transactions, or clustering
- String/blob values only; `Content-Type` preserved on GET
- Not for production secrets or sensitive data

---
> Source: [fahidsarker/dev-cache](https://github.com/fahidsarker/dev-cache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
