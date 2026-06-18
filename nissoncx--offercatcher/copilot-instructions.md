## offercatcher

> Use when the user wants recruiting emails turned into native Apple Reminders on macOS/iPhone. OpenClaw should scan and parse the mail, then hand reminder writes to the local native bridge instead of relying on node directly controlling Reminders.app.


# OfferCatcher

## What It Does

Scans Apple Mail for recruiting emails, extracts important events (interviews, assessments, deadlines) with LLM, and syncs them to native Apple Reminders on iPhone/Mac.

## Execution Boundary

- OpenClaw is responsible for orchestration: scan mail, ask the LLM to parse events, and decide whether anything should be written.
- `scripts/apple_reminders_bridge.py` is the only reminder write path.
- The bridge prefers `remindctl` (Swift + EventKit) and only falls back to AppleScript if `remindctl` is unavailable.
- Do not rely on `node -> Reminders.app` Automation as the primary path. On macOS this permission is often less stable than a native Reminders bridge.

## How To Use

### Trigger Phrases

- "Check my recruiting emails"
- "Any interviews coming up?"
- "Sync interview emails to reminders"
- "Don't let me miss my coding test"

### Workflow

```
1. Scan:
   - OpenClaw heartbeat path: direct top-level `osascript -e ...` command
   - Manual/local CLI path: `python3 scripts/recruiting_sync.py --scan-only`
2. Parse: OpenClaw LLM extracts structured recruiting events
3. Apply: `--apply-events` → sends validated events to the native reminders bridge
```

### Step 1: Scan Emails

```bash
osascript \
  -e 'tell application "Mail"' \
  -e 'set acc to account "谷歌"' \
  -e 'set mbx to mailbox "INBOX" of acc' \
  -e 'set output to ""' \
  -e 'set processedCount to 0' \
  -e 'repeat with m in messages of mbx' \
  -e 'if processedCount is greater than or equal to 300 then exit repeat' \
  -e 'set msgDate to date received of m' \
  -e 'if msgDate > ((current date) - (2 * days)) then' \
  -e 'set msgId to (id of m) as string' \
  -e 'set subj to subject of m as string' \
  -e 'set sndr to sender of m as string' \
  -e 'set ts to (date received of m) as string' \
  -e 'set c to content of m as string' \
  -e 'if (length of c) > 2000 then set c to text 1 thru 2000 of c' \
  -e 'set lineText to "谷歌" & (character id 31) & "INBOX" & (character id 31) & msgId & (character id 31) & subj & (character id 31) & sndr & (character id 31) & ts & (character id 31) & c' \
  -e 'set output to output & lineText & (character id 30)' \
  -e 'set processedCount to processedCount + 1' \
  -e 'end if' \
  -e 'end repeat' \
  -e 'return output' \
  -e 'end tell'
```

or for local debugging:

```bash
python3 scripts/recruiting_sync.py --scan-only
```

The heartbeat path returns raw mail records separated by `character id 30` and
`character id 31`. The local debugging path returns JSON.

### Step 2: LLM Parses

For each email, extract:
- `company`: Company name
- `event_type`: interview / ai_interview / written_exam / assessment / authorization / deadline
- `timing`: `{"start": "YYYY-MM-DD HH:MM", "end": "..."}` or `{"deadline": "..."}`
- `role`: Job title
- `link`: Event URL

### Step 3: Apply Events

```bash
python3 scripts/recruiting_sync.py --apply-events /tmp/events.json
```

This does not write Reminders directly from OpenClaw itself. It always routes through `scripts/apple_reminders_bridge.py`.

For Mail reads, prefer the top-level `osascript -e ...` entry when OpenClaw is
the caller. This keeps Mail automation attached to the host process instead of a
Python child process.

## LLM Parsing Prompt

```
Extract recruiting event information from this email. Return JSON.

Email:
{body}

Extract:
- company: Company name
- event_type: interview / ai_interview / written_exam / assessment / authorization / deadline
- timing: {"start": "YYYY-MM-DD HH:MM", "end": "..."} or {"deadline": "..."}
- role: Job title
- link: Event URL
- notes: Additional info
```

## Output Rules

- Reminder title: Company + Event type (e.g., "Google Interview", "Meta Coding Test")
- Include: Time, role, link in notes
- Prefer native bridge writes through `remindctl`; if remindctl is unavailable, let the bridge use its AppleScript fallback
- If no new events: respond `HEARTBEAT_OK`

## Configuration

`~/.openclaw/offercatcher.yaml`:

```yaml
mail_account: "Gmail"    # Apple Mail account name
mailbox: INBOX           # Folder to scan
days: 2                  # Scan last N days
max_results: 60          # Max emails
```

## Supported Languages

The LLM parser works with any language—Chinese, English, Japanese, German, etc. No regex, no language-specific rules.

---
> Source: [NissonCX/offercatcher](https://github.com/NissonCX/offercatcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
