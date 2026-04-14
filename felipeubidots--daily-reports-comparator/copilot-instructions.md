## daily-reports-comparator

> You have Slack + Shortcut MCPs globally configured, so no additional setup needed.

# Daily Report Comparator - Claude Code Instructions

## ✅ MCPs Already Configured!

You have Slack + Shortcut MCPs globally configured, so no additional setup needed.

## Quick Start

### From Claude Code (Natural Language)
Simply ask in natural language:

```
Compare gajaguar daily
Compare ricardo.rito@ubidots.com daily
Compare "Gerardo Geronimo" daily
```

Claude will automatically:
1. Search #dev-daily for today's message
2. Parse daily sections: Shipped, In Progress, Today, Blockers, AI Insights
3. Query Shortcut API for the developer's task activity
4. Compare and highlight discrepancies

### From Command Line (Alternative)
```bash
cd /c/Work/daily-reports
node index.js "gajaguar"               # Slack username
node index.js "ricardo.rito@ubidots.com"  # Email
node index.js "Gerardo Geronimo"       # Full name (resolved from Slack)
```

### Identifying Correct Usernames

Three reliable ways to find the correct identifier:

1. **Slack Username** (most reliable):
   - Look at their Slack profile → `@username`
   - Examples: `gajaguar`, `cristianarrieta`, `felipemoreno5879`
   - **This matches their Shortcut mention name too**

2. **Email address** (universal):
   - Works in both Slack and Shortcut
   - Examples: `gerardo@ubidots.com`, `cristian@ubidots.com`
   - Use this if the full name has diacritics or special characters

3. **Full name from Slack**:
   - Display name shown in Slack profile
   - May not work if names have spaces or special characters
   - If it fails, use the Slack username instead

## How It Works

The comparator:
1. **Searches #dev-daily** in Slack for today's report from the developer
2. **Parses the daily** into sections: Shipped, In Progress, Today, Blockers, AI Insights
3. **Queries Shortcut API** for the developer's task activity
4. **Compares the two** and highlights discrepancies:
   - Tasks reported as shipped vs actually completed in Shortcut
   - Tasks in progress in daily vs in Shortcut
   - Missing or extra items

## Expected Daily Format in Slack

```
✅ SHIPPED   [Completed work] — [AI-assisted / Manual] — [link]
⤴️ IN PROGRESS   [Open PRs or tickets] — [status]
🎯 TODAY   [Highest priority items]
🏗️ BLOCKERS  [Blocked tickets] — @[owner] — [🔴🟡🟢]
💡 AI INSIGHT  [Only if AI was used]
```

### Details per section:
- **SHIPPED**: Completed work + how it was done + link (PR/Story)
- **IN PROGRESS**: Current work + current status (% complete, waiting on review, etc.)
- **TODAY**: Top priorities from in-progress or next sprint items
- **BLOCKERS**: What's blocking + owner to follow up + severity (red=critical, yellow=high, green=medium)
- **AI INSIGHT**: Only include if you actually used AI tools (Claude, Copilot, etc.)

Slack adds metadata automatically (name, date, time, channel).

## Architecture

- **index.js**: Main script that:
  - Uses Slack API to fetch #dev-daily messages
  - Parses daily report sections
  - Uses Shortcut API to get completed/in-progress stories
  - Compares and generates comparison report

## Token Setup

### Get Slack Bot Token
1. Go to https://api.slack.com/apps
2. Create a new app or select existing
3. Go to "OAuth & Permissions"
4. Under "Scopes", add:
   - `channels:read`
   - `chat:read`
   - `users:read`
5. Copy "Bot User OAuth Token" (starts with `xoxb-`)

### Get Shortcut API Token
1. Go to https://app.shortcut.com/settings/account/api-tokens
2. Create new token
3. Copy the token

## Team Context

Team structure is defined in two places:

1. **`people-context.json`** — Machine-readable context (used by scripts)
   - Developer identifiers (username, email, full name mapping)
   - Role, seniority, team assignments
   - Focus areas by developer

2. **`TEAM.md`** — Human-readable team documentation
   - Team structure and leads
   - Member details and focus areas
   - For updating: modify both files together

See `TEAM.md` for current team breakdown.

## Future Enhancements

- [ ] Multi-user comparison
- [ ] Time tracking integration
- [ ] Weekly summary reports
- [ ] Automated daily validation checks
- [ ] Role-based insights (comparing expectations vs reality)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/felipeubidots)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/felipeubidots)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
