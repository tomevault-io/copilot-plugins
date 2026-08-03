## cv

> A single-file personal web CV hosted on GitHub Pages at `cv.tysonpriddle.com`.

# CLAUDE.md — tysonpriddle/cv

## What this repo is

A single-file personal web CV hosted on GitHub Pages at `cv.tysonpriddle.com`.

The file is self-contained: the profile photo is embedded as a base64 data URI. No external dependencies, no build step. One HTML file in, one HTML file out.

---

## File structure

```
/
└── index.html        ← the web CV (self-contained, photo embedded)
└── CNAME             ← auto-created by GitHub Pages (cv.tysonpriddle.com)
```

The source files live locally at:
```
C:\Users\tyson\Documents\Tyson_Priddle_WebCV_2026.html   ← edit this
C:\Users\tyson\Documents\Tyson_Priddle_Resume_Alinta_May2026.md  ← master resume (full metrics)
C:\Users\tyson\Documents\Tyson_Priddle_Screening_Answers_Alinta_May2026.md
```

Always edit the source file locally, then push `index.html` to this repo. Do not edit `index.html` directly in GitHub.

---

## How to update

1. Edit `C:\Users\tyson\Documents\Tyson_Priddle_WebCV_2026.html`
2. Run the push script below (requires a repo-scoped PAT)
3. GitHub Pages deploys automatically within ~60 seconds

### Push script

```python
import base64, json, urllib.request, urllib.error

TOKEN = "<repo-scoped PAT>"   # github.com/settings/tokens/new — tick 'repo'
REPO  = "tysonpriddle/cv"
PATH  = "index.html"

with open(r"C:\Users\tyson\Documents\Tyson_Priddle_WebCV_2026.html", "rb") as f:
    content_b64 = base64.b64encode(f.read()).decode("utf-8")

url = f"https://api.github.com/repos/{REPO}/contents/{PATH}"
headers = {
    "Authorization": f"Bearer {TOKEN}",
    "Accept": "application/vnd.github+json",
    "User-Agent": "claude-code",
    "X-GitHub-Api-Version": "2022-11-28"
}

sha = None
try:
    req = urllib.request.Request(url, headers=headers, method="GET")
    with urllib.request.urlopen(req) as r:
        sha = json.loads(r.read()).get("sha")
except urllib.error.HTTPError as e:
    if e.code != 404:
        raise

body = {"message": "Update CV", "content": content_b64}
if sha:
    body["sha"] = sha

data = json.dumps(body).encode("utf-8")
req = urllib.request.Request(url, data=data, headers={**headers, "Content-Type": "application/json"}, method="PUT")
with urllib.request.urlopen(req) as r:
    resp = json.loads(r.read())
    print(f"Done: {resp['content']['html_url']}")
```

---

## Hosting

| Setting | Value |
|---------|-------|
| Platform | GitHub Pages |
| Repo | github.com/tysonpriddle/cv (private) |
| Branch | main / root |
| Custom domain | cv.tysonpriddle.com |
| SSL | Auto-provisioned by GitHub Pages (Let's Encrypt) |
| Indexing | noindex, nofollow (meta tag in HTML — won't appear in Google) |

---

## DNS configuration (Bluehost)

Domain registrar: **Bluehost** — login at bluehost.com

| Type | Host | Points to | TTL |
|------|------|-----------|-----|
| CNAME | cv | tysonpriddle.github.io | 3600 |

**TTL reference:**
- `3600` = 1 hour. Standard for most records. Changes take up to 1 hour to propagate globally.
- `600` = 10 minutes. Use temporarily when making DNS changes so you can test faster, then bump back to 3600.
- `86400` = 24 hours. Avoid for records you might need to change.
- GitHub Pages also requires DNS propagation before it will issue the SSL cert. If HTTPS is not working, wait 15–30 mins after the CNAME is set.

**To verify DNS propagation:**
```
nslookup cv.tysonpriddle.com
# Should return: tysonpriddle.github.io (or a GitHub Pages IP)
```

---

## Auth

A **repo-scoped Personal Access Token** is required to push via API. The Gist token (`GITHUB_GIST_TOKEN` in `C:\Users\tyson\agent-dashboard\.env.local`) does not have repo access.

Generate a new one at: github.com/settings/tokens/new
- Note: `Claude Code CV push`
- Expiration: 7 days (single use is fine)
- Scope: tick `repo`

Do not store long-lived repo tokens. Generate fresh when needed.

---

## Content rules

- **Australian English** throughout (organise, programme, optimise)
- **No em dashes** anywhere. Use a comma to continue a sentence, or a full stop to start a new one
- **No sensitive EY data** in this file: no promotion nominations, no specific headcount figures, no internal NPS numbers, no percentage retention figures
- Run all prose edits through the `/humanizer` skill before publishing
- Photo is embedded as base64 — no external image file needed

---

## Aliases

| Alias | Meaning |
|-------|---------|
| `cv repo` | github.com/tysonpriddle/cv |
| `cv source` | C:\Users\tyson\Documents\Tyson_Priddle_WebCV_2026.html |
| `cv live` | https://cv.tysonpriddle.com |
| `cv dns` | Bluehost CNAME: cv → tysonpriddle.github.io, TTL 3600 |
| `cv push` | Run the push script above with a fresh repo-scoped PAT |

---
> Source: [tysonpriddle/cv](https://github.com/tysonpriddle/cv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
