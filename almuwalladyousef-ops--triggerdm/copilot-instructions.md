## triggerdm

> Instagram DM automation. When someone comments a trigger keyword on a watched reel, the app automatically sends them a DM sequence. Rules control which keywords, which messages, and which reels each automation applies to.

# TriggerDM — Claude Reference

## What This App Does

Instagram DM automation. When someone comments a trigger keyword on a watched reel, the app automatically sends them a DM sequence. Rules control which keywords, which messages, and which reels each automation applies to.

---

## Stack

- **Framework:** Next.js 14 (App Router), deployed on Vercel
- **Database:** Single JSON file (`db.json`) stored in Google Drive, accessed via Google Drive API
- **Instagram:** Meta Graph API v18.0 — send DMs, fetch reels
- **Webhooks:** Meta Webhooks POST to `/api/webhook`

---

## Project Map

```
app/
  layout.js                  — sidebar layout
  page.js                    — dashboard (stats + active rules)
  rules/page.js              — all rules list
  rules/new/page.js          — create rule
  rules/[id]/page.js         — edit rule
  api/webhook/route.js       — Meta webhook (GET verify + POST process)
  api/refresh-token/route.js — monthly token refresh (Vercel cron)
  api/rules/route.js         — GET list / POST create
  api/rules/[id]/route.js    — GET / PUT / DELETE single rule
  api/reels/route.js         — GET user's reels from Instagram

components/
  RuleEditor.js              — full rule form (name, toggle, keywords, messages, reels)
  MessageBuilder.js          — add/remove/reorder text + link blocks
  ReelPicker.js              — reel thumbnail grid with select/deselect

lib/
  driveDB.js                 — all database operations (read/write Drive JSON)
  instagram.js               — sendDM(), getReels(), getReelInfo()
  tokenManager.js            — refreshToken(), logs to tokenLog in Drive JSON
  verify.js                  — verifySignature() for X-Hub-Signature-256

styles/globals.css           — all styles, dark sidebar + clean white content
```

---

## Database Schema

Location: Google Drive file at `GOOGLE_DRIVE_FILE_ID`

```json
{
  "rules": [
    {
      "id": "uuid",
      "name": "Rule name",
      "active": true,
      "applyToAll": false,
      "targetReels": ["reel_id_1"],
      "keywords": ["link", "price"],
      "messages": [
        { "type": "text", "content": "Hey! Here's the link:" },
        { "type": "link", "url": "https://example.com" }
      ],
      "createdAt": "ISO timestamp"
    }
  ],
  "dmedLog": {
    "ruleId:userId": "ISO timestamp"
  },
  "tokenLog": [
    { "token": "EAAxx", "expiryDate": "2024-03-15", "refreshedAt": "ISO timestamp" }
  ]
}
```

Deduplication key: `ruleId:userId` — same user can get DMs from different rules but never the same rule twice.

---

## Environment Variables

| Variable | Description |
|---|---|
| `PAGE_ACCESS_TOKEN` | Meta long-lived page access token |
| `PAGE_ID` | Instagram Page ID |
| `APP_SECRET` | Meta app secret (webhook signature verification) |
| `VERIFY_TOKEN` | Any string you set — used in Meta webhook setup |
| `GOOGLE_DRIVE_FILE_ID` | ID of `db.json` in Google Drive |
| `GOOGLE_SERVICE_ACCOUNT_EMAIL` | Service account email |
| `GOOGLE_PRIVATE_KEY` | Full private key with `\n` characters |
| `CRON_SECRET` | Protects `/api/refresh-token` endpoint |
| `ADMIN_SECRET` | Dashboard password gate (optional) |

---

## Common Tasks

### Add or change trigger keywords
Go to **Rules → [rule name] → Trigger Keywords**. Add/remove words in the UI. No code change needed.

### Change the DM message
Go to **Rules → [rule name] → DM Message**. Edit blocks inline. Click Save.

### Add a new reel to an existing rule
Go to **Rules → [rule name] → Target Reels**. Select the new reel thumbnail. Click Save.

### Create a global rule (all reels)
When creating/editing a rule, toggle **"Apply to all reels"** on. All current and future reels will be covered.

### Pause a rule without deleting it
Toggle **Active** off in the rule editor. The rule stays saved but won't fire.

### Delete a rule
Open the rule, scroll to the bottom, click **Delete**.

---

## Webhook Flow

```
Meta → POST /api/webhook
  → verify X-Hub-Signature-256
  → parse commentText, commenterId, mediaId
  → load rules from Drive JSON
  → for each rule:
      if rule.active
      && (rule.applyToAll || rule.targetReels includes mediaId)
      && comment contains a keyword
      && user not in dmedLog for this rule:
        sendDM(commenterId, rule.messages)
        logDM(ruleId, commenterId)
```

---

## Deploy Steps

1. Push to GitHub: `git push origin main`
2. Go to [vercel.com](https://vercel.com) → Import repo → Select `triggerdm`
3. Add all env vars in Vercel dashboard → Settings → Environment Variables
4. Deploy
5. Copy your Vercel URL → paste into Meta developer dashboard as webhook URL (`https://your-app.vercel.app/api/webhook`)

---

## Google Service Account Setup

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a project → Enable **Google Drive API**
3. Create a Service Account → Download the JSON key
4. Copy `client_email` → `GOOGLE_SERVICE_ACCOUNT_EMAIL`
5. Copy `private_key` → `GOOGLE_PRIVATE_KEY`
6. In Google Drive: create a file called `db.json`, paste `{}` as content
7. Share the file with the service account email (Editor access)
8. Copy the file ID from the URL → `GOOGLE_DRIVE_FILE_ID`

---

## Meta Webhook Setup

1. Go to [developers.facebook.com](https://developers.facebook.com) → Your App → Webhooks
2. Subscribe to **Instagram** → field: `comments`
3. Callback URL: `https://your-app.vercel.app/api/webhook`
4. Verify token: whatever you set as `VERIFY_TOKEN`
5. Subscribe and verify

---

## GitHub Auto-Push (OFF BY DEFAULT)

**Only activate when you say: "activate github push"**

When activated: every file edit auto-commits and pushes to GitHub immediately. Global — applies to all projects.

To activate: tell Claude "activate github push" → Claude runs `/update-config` to add the hook.
To deactivate: tell Claude "deactivate github push" → Claude removes the hook.

---

## First-Time GitHub Push

When you say "start" for the first time:
1. `git init`
2. `gh repo create triggerdm --private --source=. --push`

From then on, every fix goes to GitHub immediately (once auto-push is activated).

---
> Source: [almuwalladyousef-ops/triggerdm](https://github.com/almuwalladyousef-ops/triggerdm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
