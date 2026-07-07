## cblanalytics-blog

> - Finance and analytics professional (not IT background)

# Claude Context — Steve Parton's Working Environment

## About Steve
- Finance and analytics professional (not IT background)
- Skills: Excel, VBA, SQL, R (solid), Python (basic)
- Runs Mac Mini (primary) + Dell laptop (secondary)
- Maintains cblanalytics.com blog and sjpconsulting.com website

## Device & Storage Architecture

| Layer | What lives here |
|-------|----------------|
| Kingston 1TB (Mac Mini external) `/Volumes/Kingston/projects/` | All active git repos |
| OneDrive `/Users/stephenparton/Library/CloudStorage/OneDrive-SJPConsulting/` | Documents, Obsidian vault, non-code files |
| GitHub | Source of truth — sync layer between devices |
| Internal SSD | Apps, system only — keep lean |

**Critical rule:** Git repos do NOT go inside OneDrive. OneDrive corrupts `.git` internals.

## Cross-Device Git Workflow
```
# Start of every session
git pull

# End of every session
git add .
git commit -m "meaningful message"
git push
```

## Active Projects
- `brigalow_mc` — Rob Project (Brigalow mulga clearing analysis)
- `brigalow_active_management` — Rob Project (active management methodology)
- `cblanalytics-blog` — this repo, Quarto blog at cblanalytics.com
- `sjpconsulting` — SJP Consulting website

## Blog Stack
- Quarto static site
- GitHub Actions deploys on push to main
- Posts live in `/posts/` as dated folders with `.qmd` files

## Obsidian Vault
Located at: `OneDrive/Obsidian/Projects/`
Full environment setup notes: `6. Wiki/synthesis/Working-Environment-Setup.md`

## Mac Mini Storage Note
Internal SSD is nearly full (250GB). Claude session data writes to 
`~/Library/Application Support/Claude/local-agent-mode-sessions/` — 
clean this up if you see ENOSPC errors.
```bash
find ~/Library/Application\ Support/Claude/local-agent-mode-sessions/ -maxdepth 1 -type d -mtime +7 -exec rm -rf {} +
```

---
> Source: [spinbris/cblanalytics-blog](https://github.com/spinbris/cblanalytics-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
