## amux

> Single-file project: everything lives in `amux-server.py` (Python server + inline HTML/CSS/JS dashboard).

# amux

Single-file project: everything lives in `amux-server.py` (Python server + inline HTML/CSS/JS dashboard).

## Structure

- `amux-server.py` — the server + dashboard (single file)
- `mcp.json` — centralized MCP server config (shared by local and cloud)
- `cloud/` — GCP VM provisioning (Terraform + setup script)

## Workflow

- **Commit after every completed task.** When you finish a piece of work (bug fix, feature, refactor), immediately `git add amux-server.py && git commit` with a concise message. Don't batch multiple tasks into one commit.
- The server auto-restarts on file save (watches its own mtime), so changes are live immediately.
- Always verify Python syntax after edits: `python3 -c "import ast; ast.parse(open('amux-server.py').read())"`

## Deploy

When the user says **"deploy"**, run the full pipeline:
1. `git add` changed files (typically `amux-server.py`)
2. `git commit` with a concise message
3. `git push origin main`

## Single-codebase rule (CRITICAL)

**`amux-server.py` is identical for both local (OSS) and cloud deployments — no exceptions.**

- Never add cloud-only or OSS-only code branches (no `if IS_CLOUD`, no `if os.environ.get('CLOUD')`).
- Features that differ between environments must be driven by headers/env vars injected by the gateway (e.g., `X-Amux-User-Email`) or by presence/absence of configuration, not by build-time flags.
- `cloud/docker/amux-server.py` must never be committed — it is auto-generated during deploy. It is in `.gitignore`.

## Server config — `~/.amux/server.env`

Persistent env vars for the server. Loaded at startup via `os.environ.setdefault` so process-level env always wins. Survives `os.execv` auto-restarts.

Example `~/.amux/server.env`:
```
AMUX_S3_BUCKET=ethan-personal
AMUX_S3_KEY=amux/calendar.ics
AMUX_S3_REGION=us-east-2
```

After creating/editing server.env, `touch amux-server.py` to trigger a reload.

## iCal / Google Calendar sync

Board items with `due` dates are exported as an iCal feed:
- Local: `GET /api/calendar.ics`
- Public S3 (for Google/Apple Calendar subscriptions): set `AMUX_S3_BUCKET` in `server.env`

S3 bucket config (one-time, already done on `ethan-personal`):
- Public access block: `BlockPublicAcls=true, IgnorePublicAcls=true, BlockPublicPolicy=false, RestrictPublicBuckets=false`
- Bucket policy grants `s3:GetObject` on `arn:aws:s3:::ethan-personal/amux/calendar.ics` only
- Public URL: `https://ethan-personal.s3.us-east-2.amazonaws.com/amux/calendar.ics`

The feed auto-uploads to S3 on every board write (POST/PATCH/DELETE). The dashboard's calendar subscription button shows the S3 URL directly when configured.

## Browser Automation

Use `/chrome-cdp` for browser tasks. It connects directly to the user's live Chrome via CDP — real tabs, real cookies, no fresh browser.

```bash
node skills/chrome-cdp/scripts/cdp.mjs list           # list open tabs
node skills/chrome-cdp/scripts/cdp.mjs snap <target>   # accessibility tree
node skills/chrome-cdp/scripts/cdp.mjs shot <target>   # screenshot
node skills/chrome-cdp/scripts/cdp.mjs click <target> <selector>
node skills/chrome-cdp/scripts/cdp.mjs type <target> <text>
node skills/chrome-cdp/scripts/cdp.mjs eval <target> <js>
node skills/chrome-cdp/scripts/cdp.mjs nav <target> <url>
```

Requires Chrome remote debugging enabled (`chrome://inspect/#remote-debugging`) and Node.js 22+.

Claude Code, the amux server, and Chrome all run on the same desktop machine. Use `https://localhost:8822` for amux dashboard URLs.

---
> Source: [mixpeek/amux](https://github.com/mixpeek/amux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
