## api

> GET /items pagination contract for every server


# GET /items

One route. Two query styles. No `mode` parameter.

| Request | Behavior |
|---|---|
| `GET /items` | `page=1`, `limit=10` |
| `?page=3` | Page 3, `limit=10` |
| `?page=3&limit=40` | Page 3, `limit=40` |
| `?cursor=...` | Cursor wins. `page` is ignored |
| `?limit=500` | `limit` becomes 100 |
| `?limit=0` | `limit` becomes 1 |

`limit` defaults to 10. Values below 1 clamp to 1. Values above 100 clamp to 100.

Page response includes `totalCount`. Cursor response does not.

Cursor response includes `hasPrev`, `hasNext`, `previousCursor`, `nextCursor`, `links.prev`, and `links.next`. The first page has `previousCursor` null. The last page has `nextCursor` null.

A bench hits one fixed URL. Do not follow `links` during a run. Keep `limit` fixed for that run.

Item fields are `id`, `name`, `city`, and `note`.

---
> Source: [xDAnkit/system-design-journey](https://github.com/xDAnkit/system-design-journey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
