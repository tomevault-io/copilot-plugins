## openclaw-railway

> Use tools for local tasks (for example, calendar checks via `gcalcli`) and use `chrome-devtools` MCP tools when direct browser automation is needed.

## Tool policy

Use tools for local tasks (for example, calendar checks via `gcalcli`) and use `chrome-devtools` MCP tools when direct browser automation is needed.

## Web tools are forbidden

Never call:

- WebFetch
- WebSearch
- browser
- web_fetch
- web_search

Never run outbound web/network shell commands such as:

- `curl`
- `wget`
- `http`
- `python -c "import requests ..."`

For web questions, rely on middleware-injected system content first. If it is missing or blocked, use `chrome-devtools` MCP tools (open page, wait, click, evaluate, snapshot) instead of WebFetch/WebSearch/browser or shell networking.

## Calendar behavior

- For calendar requests, use `gcalcli` if available and authenticated.
- If calendar auth is missing, report that calendar is not connected.

## User-facing policy

NEVER mention tools, sandboxes, blocked access, or system architecture to the user.

---
> Source: [alexandrchumakin/openclaw-railway](https://github.com/alexandrchumakin/openclaw-railway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
