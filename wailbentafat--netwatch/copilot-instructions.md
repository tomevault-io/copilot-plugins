## netwatch

> Written for LLM agents calling `netwatch` as a subprocess. Read this once; always run `netwatch --describe` to confirm exact flags against the installed version since this file can drift from a release.

# AGENTS.md — netwatch reference for AI agents

Written for LLM agents calling `netwatch` as a subprocess. Read this once; always run `netwatch --describe` to confirm exact flags against the installed version since this file can drift from a release.

## What this tool is for

`netwatch` reads a live, authenticated Chrome session's network traffic, cookies, storage, and console output via the Chrome DevTools Protocol. Use it when a task requires knowing what a web app actually sends, stores, or evaluates — not what its source code suggests it does.

**Authorization:** confirm you're permitted to inspect the target session before using this tool. It can surface session cookies and auth tokens.

---

## Preconditions

Chrome must be listening on a debugging port. The cleanest way:

```bash
netwatch launch
```

This opens a dedicated debug Chrome window that copies your real session's cookies (so you're already logged in) and returns a JSON line when ready:

```json
{"status":"launched","port":9222,"sessionSynced":true}
```

If Chrome is already running on that port: `"status":"already_running"`.

**Never launch Chrome yourself with a temp `--user-data-dir`.** A fresh profile loses the authenticated session, which defeats the point of this tool.

---

## Discovering tabs

```bash
netwatch tabs
```

```json
{"index":0,"label":"localhost:5173","title":"App","url":"http://localhost:5173/"}
{"index":1,"label":"github.com","title":"GitHub","url":"https://github.com/wailbentafat"}
```

Always resolve the tab first. Pass `--tab-index <n>` (from `tabs` output) to every command for exact matching, or `--tab <substring>` for fuzzy matching.

**`netwatch wizard` is for humans only — never call it from agent context.** It blocks on stdin at every step.

---

## Commands

### `attach` — live network stream

```bash
timeout 15 netwatch attach --tab-index 1 --filter api.github.com --bodies
```

Always wrap with `timeout` (or equivalent); this command runs until interrupted.

Default output (XHR/Fetch only, slim fields):
```json
{"url":"https://api.github.com/user","method":"GET","status":200,"size":872}
```

Flags:
- `--filter <str>` — only URLs containing this string. Always set this; unfiltered traffic on a busy page is very noisy.
- `--bodies` — include response bodies (text/JSON only).
- `--headers` — include request/response headers.
- `--all` — include all resource types (CSS, fonts, images…). Rarely needed.
- `--verbose` — full record: requestId, resourceType, fromDiskCache, fromServiceWorker…
- `--out <file>` — also append to file. Useful for long captures.

**Recommended pattern for agents:** start in background, trigger traffic, read back:
```bash
netwatch attach --tab-index 1 --filter api --bodies --out /tmp/cap.ndjson &
PID=$!
sleep 10
kill $PID
cat /tmp/cap.ndjson | jq '.url'
```

---

### `cookies` — list tab cookies

```bash
netwatch cookies --tab-index 1
netwatch cookies --tab-index 1 --find session_token   # print just the value
netwatch cookies --tab-index 1 --http-only            # only httpOnly cookies
netwatch cookies --tab-index 1 --count                # group by domain
```

Output (default NDJSON):
```json
{"name":"_gh_sess","value":"...","domain":".github.com","httpOnly":true,"secure":true}
```

`--find <name>` prints just the raw cookie value and exits — ideal for extracting a specific token:
```bash
TOKEN=$(netwatch cookies --tab-index 1 --find access_token)
```

---

### `storage` — read localStorage / sessionStorage

```bash
netwatch storage --tab-index 1
netwatch storage --tab-index 1 --type all
netwatch storage --tab-index 1 --find access_token    # print just the value
netwatch storage --tab-index 1 --count
```

Output:
```json
{"key":"user_id","value":"42","store":"localStorage"}
```

Exit code `1` when `--find` target doesn't exist.

---

### `eval` — execute JS

```bash
netwatch eval --tab-index 1 "document.title"
netwatch eval --tab-index 1 --raw "document.title"
netwatch eval --tab-index 1 "fetch('/api/me').then(r=>r.json())"
```

Promises are awaited automatically. Default output: `{"value":"...","type":"string"}`. With `--raw`: bare string value.

Exit code `1` when the expression throws.

---

### `console` — stream JS console output

```bash
timeout 30 netwatch console --tab-index 1 --level error,warn
```

Always wrap with `timeout`. Output:
```json
{"level":"error","text":"TypeError: null","ts":1750000001.4,"uncaught":true}
```

---

### `snapshot` — one-shot page state dump

```bash
netwatch snapshot --tab-index 1 --out-dir /tmp/snap
netwatch snapshot --tab-index 1 --include html,cookies,localStorage
```

Prints one manifest JSON line to stdout; writes files into `--out-dir`:
```json
{"url":"https://github.com","cookies":{"file":"/tmp/snap/cookies.json","count":12}}
```

A missing key in the manifest means that section failed (check stderr) — not that the data was empty.

---

## Exit codes

| Code | Meaning | Action |
|------|---------|--------|
| `0` | Success | Parse stdout normally |
| `1` | JS error / key not found | Check stderr; fix invocation |
| `2` | Chrome unreachable / tab not found | Tell the user; don't auto-launch Chrome |
| `3` | Snapshot partial failure | Usable output exists; check which manifest keys are missing |
| `130` | Interrupted | Expected after `timeout` or `kill` |

---

## Self-description

```bash
netwatch --describe
```

Returns the current commands, flags, and output schemas as JSON. Authoritative for the installed version.

---

## Things not to do

- Don't run `attach` without `--filter` on a busy page — it will flood your context.
- Don't use `--pretty` when parsing output programmatically.
- Don't treat a missing manifest key as "empty" — it means the capture failed.
- Don't call `wizard` from agent context — it blocks on stdin forever.
- Don't launch Chrome with a temp `--user-data-dir` — you lose the session.

---
> Source: [wailbentafat/netwatch](https://github.com/wailbentafat/netwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
