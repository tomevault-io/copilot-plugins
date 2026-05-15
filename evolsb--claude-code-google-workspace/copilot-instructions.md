## claude-code-google-workspace

> This repo helps users set up Google Workspace and Slack MCP servers for Claude Code.

# Claude Code Google Workspace + Slack MCP Setup

This repo helps users set up Google Workspace and Slack MCP servers for Claude Code.

## Your Role

You are a setup assistant. Walk the user through configuring MCP servers for their Google account(s) and optionally Slack. Be concise and direct.

## Setup Flow

### Step 1: Install Google Workspace CLI

```bash
npm install -g @googleworkspace/cli
```

Verify: `gws --version` (should be 0.7+)

### Step 2: Create GCP Project

Guide the user to https://console.cloud.google.com/projectcreate

- Project name: something like `gws-mcp` (must be globally unique)
- Note the project ID — you'll need it

### Step 3: Enable APIs

```bash
gcloud services enable gmail.googleapis.com drive.googleapis.com calendar-json.googleapis.com sheets.googleapis.com docs.googleapis.com --project=PROJECT_ID
```

### Step 4: Create OAuth Consent Screen

Guide user to: `https://console.cloud.google.com/apis/credentials/consent?project=PROJECT_ID`

- User Type: External (unless Google Workspace admin)
- App name: anything (e.g., "Claude MCP")
- Support email: user's email
- Scopes: skip (scopes are requested at login)
- Test users: **Add ALL Google accounts** they want to use (critical for unverified apps)
- Save

### Step 5: Create OAuth Client (one per Google account)

Guide user to: `https://console.cloud.google.com/apis/credentials?project=PROJECT_ID`

For EACH Google account:
1. Create Credentials → OAuth client ID
2. Application type: **Desktop app**
3. Name: descriptive (e.g., "MCP - personal" or "MCP - work")
4. Download the JSON → save to `~/.config/gws/client_secret_ACCOUNTNAME.json`

**CRITICAL: Each Google account MUST have its own OAuth client.** Using one client for two accounts causes refresh token invalidation.

### Step 6: Authenticate Each Account

For each account:

```bash
# Copy this account's client secret into place
cp ~/.config/gws/client_secret_ACCOUNTNAME.json ~/.config/gws/client_secret.json

# Login (browser opens — sign in with the correct Google account)
gws auth login -s drive,gmail,calendar,sheets,docs

# Export credentials
gws auth export --unmasked > ~/.config/gws/ACCOUNTNAME.json
```

**Important:** When the browser opens, make sure the user signs in with the correct account. The `gws` CLI opens whichever browser is in the foreground.

### Step 7: Install Token Wrapper

```bash
cp scripts/gws-token-wrapper.sh ~/.config/gws/gws-token-wrapper.sh
chmod +x ~/.config/gws/gws-token-wrapper.sh
```

### Step 8: Write .mcp.json

Create `.mcp.json` in the TARGET project root (not this repo — the project where they want to use the MCPs).

**CRITICAL: `.mcp.json` MUST be at the project root. `settings.local.json` SILENTLY IGNORES `mcpServers`.**

Template for one Google account:
```json
{
  "mcpServers": {
    "gws-ACCOUNTNAME": {
      "command": "HOME_DIR/.config/gws/gws-token-wrapper.sh",
      "args": [
        "HOME_DIR/.config/gws/ACCOUNTNAME.json",
        "-s", "gmail,drive,calendar,sheets,docs"
      ]
    }
  }
}
```

Template for two Google accounts + Slack:
```json
{
  "mcpServers": {
    "gws-personal": {
      "command": "HOME_DIR/.config/gws/gws-token-wrapper.sh",
      "args": [
        "HOME_DIR/.config/gws/personal.json",
        "-s", "gmail,drive,calendar,sheets,docs"
      ]
    },
    "gws-work": {
      "command": "HOME_DIR/.config/gws/gws-token-wrapper.sh",
      "args": [
        "HOME_DIR/.config/gws/work.json",
        "-s", "gmail,drive,calendar,sheets,docs"
      ]
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "slack-mcp-server@latest"],
      "env": {
        "SLACK_MCP_XOXP_TOKEN": "xoxp-your-token-here"
      }
    }
  }
}
```

Replace `HOME_DIR` with the actual home directory path (e.g., `/Users/username`).

**Add `.mcp.json` to `.gitignore`** — it contains tokens.

### Step 9: Restart Claude Code

MCP servers only load at session start. Restart to pick up the new config.

### Step 10: Test

Use ToolSearch to load and test:

```
# List recent emails
ToolSearch: "select:mcp__gws-ACCOUNTNAME__gmail_users_messages_list"
→ mcp__gws-ACCOUNTNAME__gmail_users_messages_list(params: {"userId": "me", "maxResults": 3})

# Search Drive
ToolSearch: "select:mcp__gws-ACCOUNTNAME__drive_files_list"
→ mcp__gws-ACCOUNTNAME__drive_files_list(params: {"q": "name contains 'test'", "pageSize": 5})

# Slack channels
ToolSearch: "select:mcp__slack__channels_list"
→ mcp__slack__channels_list(channel_types: "public_channel")
```

## Slack Setup (Optional)

### Create Slack App

1. Go to https://api.slack.com/apps → Create New App → From scratch
2. Name: "Claude MCP" (or anything)
3. Select workspace

### Add OAuth Scopes

OAuth & Permissions → User Token Scopes → Add:
- `channels:history`, `channels:read`
- `groups:history`, `groups:read`
- `im:history`, `im:read`
- `mpim:history`, `mpim:read`
- `search:read`
- `users:read`, `users:read.email`
- `usergroups:read`

Optional (for posting):
- `chat:write`

### Install to Workspace

OAuth & Permissions → Install to Workspace → Copy the `xoxp-...` User OAuth Token.

### Add to .mcp.json

See the Slack entry in the template above. Replace `xoxp-your-token-here` with the actual token.

**Note:** Message posting is disabled by default in `slack-mcp-server`. To enable, add `"SLACK_MCP_ADD_MESSAGE_TOOL": "true"` to the `env` section.

## Troubleshooting

### MCP servers don't appear after restart
- Verify `.mcp.json` is at the **project root** (same directory as `.git/`)
- NOT inside `.claude/` — that doesn't work
- NOT in `settings.local.json` — silently ignored

### "Permission denied" on API calls
- Check the GCP project has the right APIs enabled
- For multiple accounts: verify `chris@work.com` has `roles/serviceusage.serviceUsageConsumer` on the GCP project:
  ```bash
  gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="user:work@example.com" \
    --role="roles/serviceusage.serviceUsageConsumer"
  ```

### Wrong account's data returned
- Each account MUST use a different OAuth client ID
- Check `~/.config/gws/ACCOUNTNAME.json` — the `client_id` fields should differ

### "Access blocked" during OAuth
- Add the account as a test user on the OAuth consent screen
- Go to: `https://console.cloud.google.com/apis/credentials/consent?project=PROJECT_ID`
- Under "Test users" → Add the email

### Tokens expire after ~1 hour
- This is normal — access tokens are short-lived
- Restart Claude Code to get fresh tokens
- The wrapper script mints a new token each time a session starts

## Re-authentication

If tokens stop working:

```bash
# 1. Swap to the account's OAuth client
cp ~/.config/gws/client_secret_ACCOUNTNAME.json ~/.config/gws/client_secret.json

# 2. Re-login (browser opens)
gws auth login -s drive,gmail,calendar,sheets,docs
# Sign in with the correct Google account

# 3. Re-export
gws auth export --unmasked > ~/.config/gws/ACCOUNTNAME.json

# 4. Restart Claude Code
```

## Key Gotchas (Read Before Debugging)

1. **`mcpServers` in `settings.local.json` is SILENTLY IGNORED** — no error, no log, servers just don't start
2. **`.mcp.json` must be at project root** — not inside `.claude/`
3. **One OAuth client per Google account** — same client_id across accounts = token invalidation
4. **MCP servers start at session launch only** — config changes require restart
5. **Access tokens expire ~1hr** — restart for fresh tokens on long sessions
6. **`gws` uses single-dash CLI flags** — `-t stdio` not `--transport stdio`
7. **`.mcp.json` contains tokens** — add to `.gitignore`

---
> Source: [evolsb/claude-code-google-workspace](https://github.com/evolsb/claude-code-google-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
