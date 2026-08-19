## backup

> Local backup location and procedure for this project


# Project backup

When the user asks to **back up** this project (or similar: “backup”, “make a backup”, “sync backup”):

## Destination

Copy to:

`/Users/wolf/Backup_TMP/ComputerScience/`

Source is this repo root (on the Lacie drive).

## How

Use `rsync -a` (macOS-compatible; do not use `--info=progress2`).

Exclude:

- `.env` (API keys — never copy into Backup_TMP)
- `node_modules`
- `_archive/**/.next` (build cache noise)
- `web_site/.next`

Include everything else that matters: `.git`, `deck/images/`, `deck/out/`, `deck/templates/`, `deck/data/`, `web_site/` (minus `.next`), docs, scripts.

Example:

```bash
mkdir -p /Users/wolf/Backup_TMP
rsync -a --progress \
  --exclude 'node_modules' \
  --exclude '.env' \
  --exclude '_archive/**/.next' \
  --exclude 'web_site/.next' \
  "<repo-root>/" \
  "/Users/wolf/Backup_TMP/ComputerScience/"
```

Then confirm size and that key files exist (e.g. `deck/out/print/01-hash-map-info.png` once generated).

## Notes

- `Backup_TMP` is a **local Mac** copy — not off-site. GitHub push is separate if they ask.
- Do **not** commit or push `.env`.
- `out/` is gitignored; the local backup is how print assets are preserved off the Lacie.

---
> Source: [owolfdev/computer-science-study-cards](https://github.com/owolfdev/computer-science-study-cards) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
