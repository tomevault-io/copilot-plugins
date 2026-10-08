## mcp

> MCP server (ship-status) for SHIP Dashboard REST API tools


MCP server **`ship-status`** wraps the dashboard REST API for AI agents. Code lives in:

- `mcp/shared.py` -- `DashboardClient` and `ShipStatusAPI` (HTTP client and domain logic)
- `mcp/public_server.py` -- public read-only MCP server (no credentials required)
- `mcp/auth_server.py` -- authenticated write MCP server (behind oauth-proxy, requires SA token)

Local config: [`.mcp.json`](../../.mcp.json) → `mcp/run.sh` (runs `public_server.py` by default)

- Env: `SHIP_STATUS_PUBLIC_API_URL`, `SHIP_STATUS_PROTECTED_API_URL`, `SHIP_STATUS_AUTH_TOKEN_FILE`
- Tests: `mcp/.venv/bin/pytest mcp/` (install `mcp/requirements-dev.txt` first)

The two servers are entirely separate entry points:

- **`public_server.py`**: Read-only tools, accepts unauthenticated traffic. Stateless, holds no credentials.
- **`auth_server.py`**: Write tools, sits behind oauth-proxy. Holds its own SA token to call the dashboard's protected API. Only authenticated callers can reach it.

The deployment overrides `CMD` to run `auth_server.py` for the authenticated container.

SLO reads (`get_team_slo`, `get_team_slo_summary`) belong on `public_server.py`. SLO writes (`upsert_slo_item`, `add_slo_item_link`) belong on `auth_server.py`.

See `security.instructions.md` for the full auth model and credential placement rules.

Do not add dev workflow tools here -- use **`ship-status-dev`** (`ship-status-dev/`).

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
