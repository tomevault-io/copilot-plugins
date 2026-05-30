## mcp-kanboard

> Python MCP (Model Context Protocol) server that wraps Kanboard's JSON-RPC API. Distributed via `uvx` from GitHub.

# mcp-kanboard

Python MCP (Model Context Protocol) server that wraps Kanboard's JSON-RPC API. Distributed via `uvx` from GitHub.

## What this is

- ~55 tools (prefix `kb_`) covering projects, tasks, comments, subtasks, users, categories, tags, search, time tracking, file attachments.
- Single active Kanboard instance per process. Switch local↔remote by changing env vars and restarting the MCP host (Claude Code, MCP Inspector, etc.).
- Destructive tools (`kb_delete_*`, `kb_remove_user`) require explicit `confirm=true` — they raise `ConfirmRequiredError` otherwise.

## Stack

- Python 3.11+ (3.13 dev), `uv` for deps and entry points
- `mcp[cli]>=1.2.0` (FastMCP decorator style)
- `httpx` sync client with Basic auth, 2× retry on transient network errors only
- `pydantic` (schema gen from type hints), `python-dotenv` (local `.env` for dev)

## File map

```
src/mcp_kanboard/
├── __main__.py          # python -m mcp_kanboard → server.main()
├── config.py            # load_settings(): env → Settings (frozen dataclass)
├── client.py            # KanboardClient.call(method, params) → result | raise
├── errors.py            # KanboardError, AuthError, NotFoundError, ConfirmRequiredError
├── guards.py            # require_confirm(confirm, action)
├── server.py            # FastMCP("kanboard"), eager `mcp` for `mcp dev`
├── smoke.py             # mcp-kanboard-smoke entry — getVersion + getAllProjects
└── tools/               # one module per Kanboard domain, each exports register(mcp, client)
```

## Auth model

`KanboardClient` uses HTTP Basic. Two token types both work transparently:

| Token | Username | Source |
|---|---|---|
| Application | `jsonrpc` (literal) | Kanboard → Settings → API |
| Personal | user's login | Kanboard → My Profile → Actions → API |

Env vars (all read by `config.load_settings()`):
- `KANBOARD_URL` (required) — `/jsonrpc.php` suffix appended if missing
- `KANBOARD_API_TOKEN` (required)
- `KANBOARD_USERNAME` (default `jsonrpc`)
- `KANBOARD_TIMEOUT` (default 30)
- `KANBOARD_MAX_ATTACHMENT_MB` (default 25 — applies to both upload + download)
- `KANBOARD_VERIFY_SSL` (default true; set false for self-signed remote)
- `MCP_PASSPHRASE` (required in `--http` mode unless `--insecure-no-auth`) — gates the OAuth login form claude.ai redirects users to
- `MCP_HTTP_HOST` / `MCP_HTTP_PORT` (default `127.0.0.1` / `8765`) — bind for `--http` mode

Env vars consumed by `scripts/start-web.ps1` only (not the server):
- `MCP_PORT` — port override when run without `-Port`
- `KANBOARD_NGROK_AUTHTOKEN` — passed to ngrok as `--authtoken`. Use when another MCP on this machine also uses ngrok and needs a different account. Falls back to `NGROK_AUTHTOKEN`, then ngrok global config.
- `KANBOARD_NGROK_DOMAIN` — static domain (no `https://` prefix). Passed as `--url https://<domain>`. **Required** when the authtoken's account has another agent running, otherwise ngrok picks the wrong default domain → `ERR_NGROK_334`.

## Tool conventions

- **Names**: `kb_<verb>_<noun>` snake_case. `kb_` prefix is the MCP-level disambiguator.
- **Signatures**: positional Kanboard required params first; optional params as keyword args defaulting to `None`. Only non-None values are forwarded — Kanboard doesn't tolerate stray nulls in many endpoints.
- **Returns**: domain-shaped dicts/lists/ints (e.g. `kb_create_task` returns the new id). Destructive tools return `{"removed": bool, "<id_field>": int}`.
- **Errors**: bubble `KanboardError` subclasses; the server-level FastMCP wrapper converts them to clean tool errors.

## Adding a new tool

The pattern is mechanical. See `src/mcp_kanboard/tools/comments.py` for a small reference. To add e.g. `kb_get_task_links`:

1. Pick the right module under `tools/` (or create one and register it in `tools/__init__.py`).
2. Add a function decorated with `@mcp.tool()`. Type hints + the docstring drive the schema and the description the model sees.
3. Translate to Kanboard's JSON-RPC method via `client.call("methodName", {param: value, ...})`.
4. If destructive: take `confirm: bool = False`, call `require_confirm(confirm, "<verb> <subject>")` first, and return `{"removed": ok, "<id>": id_value}`.
5. Smoke-test via `uv run mcp dev src/mcp_kanboard/server.py` (MCP Inspector) before pushing.

## Local development

```powershell
uv sync                                       # install deps into .venv
Copy-Item .env.example .env                   # fill in KANBOARD_URL + KANBOARD_API_TOKEN
uv run mcp-kanboard-smoke                     # auth + connectivity check
uv run mcp dev src\mcp_kanboard\server.py     # MCP Inspector in browser
```

The smoke test calls `getVersion` + `getAllProjects` and exits non-zero on failure — the fastest way to catch auth/URL mistakes.

## Release flow

Distribution is `uvx` from the GitHub repo — no PyPI publish needed.

1. Edit code in `src/mcp_kanboard/`.
2. Commit + push to `main` on `github.com/mrquanga3/mcp-kanboard`.
3. Users (or you) refresh the cached install:
   ```powershell
   uvx --refresh --from git+https://github.com/mrquanga3/mcp-kanboard mcp-kanboard --help
   ```
   Then restart Claude Code. The MCP host will respawn the server from the new commit.

Claude Code MCP registration (already done at user scope):
```
claude mcp add kanboard -s user \
  -e KANBOARD_URL=http://localhost/kanboard \
  -e KANBOARD_USERNAME=jsonrpc \
  -e KANBOARD_API_TOKEN=<token> \
  -- uvx --from git+https://github.com/mrquanga3/mcp-kanboard mcp-kanboard
```

Config lives in `C:\Users\ADMIN\.claude.json` under `mcpServers.kanboard`. To change the token without re-running the command, edit that file directly and restart Claude Code.

## Running alongside other ngrok-based MCPs

ngrok free plan = 1 static domain per account. When this MCP runs in parallel with another (e.g. mcp-gitlab) on the same machine, both must NOT share the same ngrok account, otherwise the second agent fails with `ERR_NGROK_334` ("endpoint already online").

Setup:
1. Register a second ngrok account (separate email). It gets its own static domain like `chevron-handball-impromptu.ngrok-free.dev`.
2. In `.env`, set both `KANBOARD_NGROK_AUTHTOKEN` (that account's token) and `KANBOARD_NGROK_DOMAIN` (that account's static domain, no scheme).
3. `start-web.ps1` passes them as explicit CLI flags to ngrok (`--authtoken` + `--url https://<domain>`). CLI flags override ngrok global config — required because env-var-only setups can lose to a stale `ngrok config add-authtoken` from the other account.

The script's cleanup (`Stop-NgrokOnPort`) only kills ngrok agents whose cmdline targets *this* MCP's `$Port` (matched with `\b` word boundary). Agents for other MCPs survive.

When `KANBOARD_NGROK_DOMAIN` is set, the script skips probing `/api/tunnels` and trusts the explicit URL. When it's NOT set, the script probes ports 4040–4044 (second agent's web inspector bumps to 4041, third to 4042, etc.) and filters tunnels by `config.addr` containing `:$Port` to avoid picking up the wrong agent's tunnel.

## Things to NOT do

- **Don't remove the dual-transport (stdio + http) flow in `server.py`.** stdio is what Claude Code spawns; `--http` is what `scripts\start-web.ps1` exposes to claude.ai web via ngrok. Both must keep working.
- **Don't drop the OAuth bearer middleware on the HTTP path.** It's the only thing standing between the public ngrok URL and full Kanboard write access. `--insecure-no-auth` exists for short manual tests only.
- **Don't store OAuth tokens anywhere persistent without thinking it through.** Current design keeps them in process memory (`OAuthState` dicts) on purpose — server restart wipes them and claude.ai silently re-auths via refresh token / passphrase form. Disk persistence introduces a leak vector and migration headaches.
- **Don't switch the passphrase comparison to `==`.** `hmac.compare_digest` matters — same-time comparison protects against timing attacks even on a single-user server.
- **Don't add an async transport.** stdio MCP throughput is fine with sync httpx; async doubles the code paths for no win.
- **Don't add a caching layer.** Kanboard's API is fast enough; caching introduces correctness bugs for write-then-read flows.
- **Don't expose multiple Kanboard instances in one process.** Switching is restart-based by design — keeps the env model simple.
- **Don't bypass the `confirm` guard.** It's the safety net for destructive ops; it must remain a positional/keyword param the LLM has to set explicitly.
- **Don't retry mutating calls** on HTTP errors. Network-layer retry only (transport errors / read timeouts); 4xx/5xx are not retried to avoid duplicate creates/updates.
- **Don't put non-ASCII characters in `scripts/*.ps1`.** Windows PowerShell 5.1 reads `.ps1` files without BOM as Windows-1252, not UTF-8. Em dashes (`—`), multiplication signs (`×`), curly quotes, etc. get mis-decoded and the parser breaks with cryptic "Missing expression after ','" errors on innocent comment lines. Stick to plain ASCII (`--`, `x`, straight quotes).
- **Don't drop `--authtoken` / `--url` from the ngrok launch** when env vars are present. CLI flags are the only authoritative source — `NGROK_AUTHTOKEN` env var alone can lose to a previously-installed global config token (`ngrok config add-authtoken`), causing the wrong account to be used and `ERR_NGROK_334`.
- **Don't widen `Stop-NgrokOnPort`'s cmdline regex** beyond `http\s+$LocalPort\b`. Removing `\b` makes port `8765` match `87650`, `87651`, etc., and could kill ngrok agents for unrelated MCPs.

## Out of scope (v1, intentional)

Webhooks, custom fields, automation rules, swimlane/column/group CRUD, task links, LDAP/OAuth, backups, import/export, group membership management. Add in v2 if users ask.

---
> Source: [mrquanga3/mcp-kanboard](https://github.com/mrquanga3/mcp-kanboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
