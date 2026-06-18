## updates

> **Skill name:** `updates-cli`

# Updates CLI Skill

**Skill name:** `updates-cli`  
**Version:** 2.0.0  
**Description:** Full-featured CLI for the Updates notification, scheduling, and timer system.

> **When to invoke this skill:**  
> Use when the user or an automated workflow needs to:  
> - Send a notification to any configured channel (ntfy, Discord, Slack, Telegram, email, SMS, webhook, etc.)  
> - Manage notification channels (create, list, test, enable/disable, delete)  
> - Manage schedules (one-time, interval, or cron-based)  
> - Manage timers (create, start, stop, pause, resume, reset)  
> - Browse notification history and retry failed deliveries  
> - Check system health and statistics

---

## Prerequisites

The Updates backend must be running and reachable (default: `http://127.0.0.1:8000`).

```bash
# Start the backend (Windows PowerShell — from the project root)
.\launch.ps1

# Or start directly
cd backend && uv run uvicorn updates.api.main:app --host 127.0.0.1 --port 8000
```

Install the CLI once:

```bash
cd backend
uv pip install -e .
updates-cli --help
```

Set the API URL if the backend is not on localhost:

```bash
export UPDATES_API_URL=http://myserver:8000   # Linux/macOS/WSL
$env:UPDATES_API_URL = "http://myserver:8000" # Windows PowerShell
```

---

## Complete Command Reference

### Notifications

| Command | Description |
|---|---|
| `send <channel> <message> [--title T] [--priority P]` | Send a notification |
| `history [--limit N] [--status sent\|failed\|pending]` | List recent notifications |
| `notification-retry <id>` | Retry a failed notification |
| `notification-delete <id>` | Delete a notification record |

### Channels

| Command | Description |
|---|---|
| `channels` | List all channels |
| `channel-create --name X --type Y [--set K=V ...] [--disabled]` | Create a channel |
| `channel-test <name-or-id>` | Send a test message via a channel |
| `channel-enable <name-or-id>` | Enable a disabled channel |
| `channel-disable <name-or-id>` | Disable a channel |
| `channel-delete <name-or-id>` | Delete a channel |

**Channel types and their `--set` keys:**

| type | required `--set` keys |
|---|---|
| `ntfy` | `ntfy_topic` (and optionally `ntfy_server`, `ntfy_token`) |
| `discord` | `discord_webhook_url` |
| `slack` | `slack_webhook_url` |
| `telegram` | `telegram_bot_token`, `telegram_chat_id` |
| `email` | `smtp_host`, `smtp_port`, `smtp_username`, `smtp_password`, `from_email`, `to_email` |
| `sms` | `twilio_account_sid`, `twilio_auth_token`, `twilio_from_number`, `twilio_to_number` |
| `webhook` | `webhook_url` (optionally `webhook_method`) |
| `phone_call` | `call_from_number`, `call_to_number` |

### Schedules

| Command | Description |
|---|---|
| `schedules` | List all schedules |
| `schedule-create ...` (see below) | Create a schedule |
| `schedule-enable <id>` | Enable a schedule |
| `schedule-disable <id>` | Disable a schedule |
| `schedule-trigger <id>` | Run a schedule immediately |
| `schedule-delete <id>` | Delete a schedule |

**`schedule-create` flags:**
```
--name X          schedule name (required)
--channel X       channel name or id (required)
--type once|interval|cron  (required)
--title T         notification title (required)
--message M       notification message (required)
--at ISO8601      datetime for once-type (e.g. 2026-04-06T09:00:00)
--interval-seconds N  interval in seconds, min 60
--cron EXPR       cron expression (e.g. "0 9 * * *")
--disabled        create in disabled state
```

### Timers

| Command | Description |
|---|---|
| `timers` | List all timers |
| `timer-create --name X --duration N [--channel C] [--start]` | Create (and optionally start) a timer |
| `timer-start <id>` | Start a stopped/paused timer |
| `timer-stop <id>` | Stop a timer |
| `timer-pause <id>` | Pause a running timer |
| `timer-resume <id>` | Resume a paused timer |
| `timer-reset <id>` | Reset a timer to its full duration |
| `timer-delete <id>` | Delete a timer |

### System

| Command | Description |
|---|---|
| `status` | Show backend health, channel/notification/timer stats |

---

## Tool Definitions (Anthropic tool_use format)

### `updates_send`

```json
{
  "name": "updates_send",
  "description": "Send a notification to a configured channel (ntfy, Discord, Slack, Telegram, email, SMS, webhook, etc.).",
  "input_schema": {
    "type": "object",
    "properties": {
      "channel": {"type": "string", "description": "Channel name or numeric id"},
      "message": {"type": "string"},
      "title":   {"type": "string", "description": "Optional title (default: 'Notification')"},
      "priority":{"type": "string", "enum": ["low","normal","high","urgent"], "default": "normal"}
    },
    "required": ["channel", "message"]
  }
}
```

### `updates_channels`

```json
{
  "name": "updates_channels",
  "description": "List all configured notification channels. Use before updates_send to discover channel names.",
  "input_schema": {"type": "object", "properties": {}, "required": []}
}
```

### `updates_channel_create`

```json
{
  "name": "updates_channel_create",
  "description": "Create a new notification channel.",
  "input_schema": {
    "type": "object",
    "properties": {
      "name":     {"type": "string"},
      "type":     {"type": "string", "enum": ["ntfy","discord","slack","telegram","email","sms","webhook","pushover","phone_call"]},
      "config":   {"type": "object", "description": "Channel-specific config fields (e.g. ntfy_topic, discord_webhook_url)"},
      "disabled": {"type": "boolean", "description": "Create in disabled state", "default": false}
    },
    "required": ["name", "type"]
  }
}
```

### `updates_history`

```json
{
  "name": "updates_history",
  "description": "Retrieve recent notification delivery history.",
  "input_schema": {
    "type": "object",
    "properties": {
      "limit":  {"type": "integer", "default": 20},
      "status": {"type": "string", "enum": ["sent","failed","pending","scheduled"]}
    },
    "required": []
  }
}
```

### `updates_schedule_create`

```json
{
  "name": "updates_schedule_create",
  "description": "Create a new notification schedule (once, interval, or cron).",
  "input_schema": {
    "type": "object",
    "properties": {
      "name":             {"type": "string"},
      "channel":          {"type": "string", "description": "Channel name or id"},
      "type":             {"type": "string", "enum": ["once","interval","cron"]},
      "title":            {"type": "string"},
      "message":          {"type": "string"},
      "at":               {"type": "string", "description": "ISO8601 datetime for once-type"},
      "interval_seconds": {"type": "integer", "description": "Interval (min 60) for interval-type"},
      "cron":             {"type": "string",  "description": "Cron expression for cron-type"},
      "disabled":         {"type": "boolean", "default": false}
    },
    "required": ["name","channel","type","title","message"]
  }
}
```

### `updates_timer_create`

```json
{
  "name": "updates_timer_create",
  "description": "Create a countdown timer that can optionally send a notification on completion.",
  "input_schema": {
    "type": "object",
    "properties": {
      "name":     {"type": "string"},
      "duration": {"type": "integer", "description": "Duration in seconds"},
      "channel":  {"type": "string",  "description": "Channel to notify on completion (optional)"},
      "title":    {"type": "string"},
      "message":  {"type": "string"},
      "start":    {"type": "boolean", "description": "Start immediately", "default": false}
    },
    "required": ["name","duration"]
  }
}
```

### `updates_status`

```json
{
  "name": "updates_status",
  "description": "Check Updates backend health and system statistics.",
  "input_schema": {"type": "object", "properties": {}, "required": []}
}
```

---

## Tool Implementation (server-side execution)

```python
import subprocess, json

def run_updates_cli(tool_name: str, tool_input: dict) -> str:
    base = ["updates-cli", "--json"]

    if tool_name == "updates_send":
        cmd = base + ["send", tool_input["channel"], tool_input["message"]]
        if tool_input.get("title"):    cmd += ["--title",    tool_input["title"]]
        if tool_input.get("priority"): cmd += ["--priority", tool_input["priority"]]

    elif tool_name == "updates_channels":
        cmd = base + ["channels"]

    elif tool_name == "updates_channel_create":
        cmd = base + ["channel-create", "--name", tool_input["name"],
                      "--type", tool_input["type"]]
        for k, v in (tool_input.get("config") or {}).items():
            cmd += ["--set", f"{k}={v}"]
        if tool_input.get("disabled"):
            cmd += ["--disabled"]

    elif tool_name == "updates_history":
        cmd = base + ["history", "--limit", str(tool_input.get("limit", 20))]
        if tool_input.get("status"): cmd += ["--status", tool_input["status"]]

    elif tool_name == "updates_schedule_create":
        cmd = base + ["schedule-create",
                      "--name",    tool_input["name"],
                      "--channel", tool_input["channel"],
                      "--type",    tool_input["type"],
                      "--title",   tool_input["title"],
                      "--message", tool_input["message"]]
        if tool_input.get("at"):               cmd += ["--at", tool_input["at"]]
        if tool_input.get("interval_seconds"): cmd += ["--interval-seconds", str(tool_input["interval_seconds"])]
        if tool_input.get("cron"):             cmd += ["--cron", tool_input["cron"]]
        if tool_input.get("disabled"):         cmd += ["--disabled"]

    elif tool_name == "updates_timer_create":
        cmd = base + ["timer-create",
                      "--name",     tool_input["name"],
                      "--duration", str(tool_input["duration"])]
        if tool_input.get("channel"): cmd += ["--channel", tool_input["channel"]]
        if tool_input.get("title"):   cmd += ["--title",   tool_input["title"]]
        if tool_input.get("message"): cmd += ["--message", tool_input["message"]]
        if tool_input.get("start"):   cmd += ["--start"]

    elif tool_name == "updates_status":
        cmd = base + ["status"]

    else:
        return json.dumps({"error": f"unknown tool: {tool_name}"})

    result = subprocess.run(cmd, capture_output=True, text=True)
    return result.stdout if result.returncode == 0 else result.stderr
```

---

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `UPDATES_API_URL` | `http://127.0.0.1:8000` | Backend API base URL |

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Logical error (not found, validation rejected) |
| 2 | Transport error (cannot connect, HTTP error) |
| 130 | Interrupted (Ctrl-C) |

---
> Source: [ephoenix36/Updates](https://github.com/ephoenix36/Updates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
