## skill-browser-use

> Run a task on a real cloud-hosted browser via Browser Use Cloud. Use whenever the user wants to look something up, click around, fill a form, scrape a logged-in page, post a message, or do anything on a website — including authenticated sites (LinkedIn, Gmail, internal tools) when a saved browser profile exists. The Browser Use agent does the clicking; this skill just dispatches the natural-language task, shares a live-watch URL with the user, and returns the result.


# Browser Use Cloud bridge

This skill turns any natural-language web task into a real run on a cloud-hosted, stealth browser at [Browser Use Cloud](https://cloud.browser-use.com). The user gets a live URL they can watch in real time, and you get a structured result back.

**When to activate**
- The user asks to do *anything* on a website ("check my LinkedIn DMs", "find a fund manager in my 1st-degree connections", "buy this thing", "fill out this form", "what's the top post on HN right now").
- The request needs a real browser (JavaScript, login state, clicks, multi-page navigation), not just a static fetch.
- For static read-only fetches of public pages, plain `curl` is fine — don't waste a browser session.

**When NOT to activate**
- The user asks a question you can answer from memory or the local filesystem.
- The user wants you to run code, edit files, or use other local tools.

---

## First-run setup (self-bootstrap)

Run these checks **before** dispatching any task. They run once per machine.

### 1. API key — required

```bash
if [ -z "${BROWSER_USE_API_KEY:-}" ]; then
  # Try loading from the OpenClaw env file
  [ -f "$HOME/.openclaw/.env" ] && set -a && . "$HOME/.openclaw/.env" && set +a
fi
```

If `$BROWSER_USE_API_KEY` is still empty, reply to the user **in whichever channel they're chatting on** (Telegram, terminal, web) with:

> I need a Browser Use API key. Create one at <https://cloud.browser-use.com/settings?tab=api-keys&new=1>, then paste it here (it starts with `bu_`). I'll store it locally and won't ask again.

When they reply with the key, validate (`^bu_[A-Za-z0-9_-]+$`), then persist:

```bash
mkdir -p "$HOME/.openclaw"
touch "$HOME/.openclaw/.env"
chmod 600 "$HOME/.openclaw/.env"
# Replace or append the line
grep -v '^BROWSER_USE_API_KEY=' "$HOME/.openclaw/.env" > "$HOME/.openclaw/.env.tmp" || true
echo "BROWSER_USE_API_KEY=$KEY" >> "$HOME/.openclaw/.env.tmp"
mv "$HOME/.openclaw/.env.tmp" "$HOME/.openclaw/.env"
export BROWSER_USE_API_KEY="$KEY"
```

### 2. Browser profile — optional, but required for logged-in tasks

A *profile* in Browser Use Cloud is a persistent browser state (cookies, localStorage, saved passwords). The user logs into a site **once** via the Browser Use Cloud dashboard, gets a `profile_id` (UUID), and every future session that passes that `profileId` is already logged in.

After loading `~/.openclaw/.env` above, also check `$BROWSER_USE_PROFILE_ID`. If the task obviously requires a login (LinkedIn, Gmail, X, banking, anything that says "my account", "my inbox", "my contacts") **and** the env var is missing, reply:

> This needs you to be logged in. One-time setup:
> 1. Go to <https://cloud.browser-use.com/profiles> and click **New profile**.
> 2. Open the live browser they give you, log in to the site(s) you want me to use (LinkedIn, Gmail, etc.), then close it.
> 3. Copy the profile ID (a UUID) and paste it here. I'll store it for next time.

When they reply with a UUID, validate (`^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$`), then persist the same way as the API key (line `BROWSER_USE_PROFILE_ID=...` in `~/.openclaw/.env`).

For tasks that don't need a login (public site, HN, weather, news), skip the profile prompt and dispatch without `profileId`.

---

## Dispatching the task

Browser Use Cloud's v3 REST API:

- Base URL: `https://api.browser-use.com/api/v3`
- Auth header: `X-Browser-Use-API-Key: bu_...`

### Start the session

```bash
TASK='<the user'\''s request, verbatim or lightly cleaned up>'

REQ=$(jq -n --arg task "$TASK" --arg profile "${BROWSER_USE_PROFILE_ID:-}" '
  {task: $task}
  + (if $profile == "" then {} else {profileId: $profile} end)
')

RESP=$(curl -sS -X POST https://api.browser-use.com/api/v3/sessions \
  -H "X-Browser-Use-API-Key: $BROWSER_USE_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$REQ")

SESSION_ID=$(echo "$RESP" | jq -r '.id')
LIVE_URL=$(echo "$RESP" | jq -r '.liveUrl // empty')
```

**Immediately** send `$LIVE_URL` to the user — they can watch the agent work in real time. Example reply:

> Working on it. Watch live: <LIVE_URL>

### Poll for completion

```bash
DEADLINE=$(( $(date +%s) + 180 ))   # 180s default; bump for heavier tasks
while [ "$(date +%s)" -lt "$DEADLINE" ]; do
  sleep 3
  STATE=$(curl -sS https://api.browser-use.com/api/v3/sessions/$SESSION_ID \
    -H "X-Browser-Use-API-Key: $BROWSER_USE_API_KEY")
  STATUS=$(echo "$STATE" | jq -r '.status')
  case "$STATUS" in
    stopped|timed_out|error) break ;;
  esac
done
```

### Return the result

- If `status == "stopped"` and `isTaskSuccessful == true` → reply to the user with `.output` (it may be a string or a structured object).
- If `status == "stopped"` and `isTaskSuccessful == false` → reply with `.output` plus a brief "the agent gave up — here's what it found" framing.
- If `status == "error"` or `"timed_out"` → tell the user, include `.lastStepSummary` and `$LIVE_URL` so they can inspect.
- If the 180s deadline hits while still `running` → tell the user it's still running, share `$LIVE_URL`, and offer to poll again later. **Do not** stop the session — it'll finish on its own.

---

## Behaviour rules

- **Always relay the `liveUrl` to the user as soon as you get it.** That's the killer feature — they should see the browser working.
- **Don't add CoT or planning before dispatch.** The Browser Use agent has its own planner; double-planning wastes tokens and slows things down. Pass the task through with minimal rewriting.
- **Don't pre-validate.** If the user asks to do something on a site that needs login and `$BROWSER_USE_PROFILE_ID` isn't set, dispatch anyway — the agent will tell you it can't log in, and *then* you prompt for profile setup. Cheaper than guessing wrong.
- **One task per skill invocation.** If the user has follow-ups, dispatch a new task (or, advanced: pass `sessionId` + `keepAlive: true` to reuse the same browser).
- **Never log or echo `$BROWSER_USE_API_KEY`.**

---

## Quick reference

| Need | Endpoint |
| --- | --- |
| Start task | `POST /api/v3/sessions` with `{task, profileId?}` |
| Poll status | `GET /api/v3/sessions/{id}` |
| Stop early | `POST /api/v3/sessions/{id}/stop` |
| List profiles | `GET /api/v3/profiles` |
| Live messages stream | `GET /api/v3/sessions/{id}/messages` |

Full reference: <https://docs.browser-use.com/cloud/api-reference> · OpenAPI: <https://docs.browser-use.com/cloud/openapi/v3.json>

---
> Source: [A-I-M-S/skill-browser-use](https://github.com/A-I-M-S/skill-browser-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
