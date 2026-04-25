## epdchecker

> **epdchecker** is a Node.js tool that periodically checks the availability of Dutch hospital patient portals (EPD-portalen). It performs HTTP reachability checks, DNS lookups, and takes screenshots of each portal, then generates a static HTML dashboard that is published to GitHub Pages.

# CLAUDE.md

## Project overview

**epdchecker** is a Node.js tool that periodically checks the availability of Dutch hospital patient portals (EPD-portalen). It performs HTTP reachability checks, DNS lookups, and takes screenshots of each portal, then generates a static HTML dashboard that is published to GitHub Pages.

Live dashboard: https://schellevis.github.io/epdchecker

---

## Key commands

```bash
# Run the checker (generates dist/)
npm run check
# or directly:
node check.js

# Run checker and deploy dist/ to gh-pages branch
./deploy.sh

# Dry-run (check but do not push)
./deploy.sh --dry-run
```

There are no test or lint scripts in this project.

---

## Repository structure

| File/dir | Purpose |
|---|---|
| `check.js` | Main script – checks hospitals, takes screenshots, generates HTML/JSON output |
| `hospitals.json` | List of hospital domains and names to monitor |
| `deploy.sh` | Runs `check.js`, then force-pushes `dist/` to the `gh-pages` branch |
| `package.json` | npm metadata; `playwright` is the only runtime dependency |
| `dist/` | Generated output directory (gitignored) |
| `dist/index.html` | Generated dashboard page |
| `dist/status.json` | Machine-readable JSON summary of the last run |
| `dist/screenshots/` | JPEG screenshots per hospital (`<slug>.jpg`) |

---

## Architecture notes

### check.js

- **Concurrency**: 5 hospitals are checked simultaneously (`CONCURRENCY = 5`).
- **HTTP check** (`checkHttp`): plain `fetch` with 12 s timeout (`HTTP_TIMEOUT`), follows redirects, returns `{ status, ok }`.
- **DNS lookup** (`resolveAddresses`): `dns.lookup` with `all: true`; returns unique IP addresses.
- **HIX365 detection**: if a hospital resolves to `20.86.217.65` it is tagged as a HIX365 portal.
- **Screenshots** (`takeScreenshot`): uses Playwright Chromium (`playwright` package). If `playwright` is not installed the script still runs, but screenshots are skipped.
- **Output sorting**: offline hospitals appear first, then online; within each group sorted alphabetically by name (Dutch locale).
- **HTML generation**: fully inline CSS with CSS custom properties for dark/light mode; lightbox for enlarged screenshots; auto-refreshes every 10 minutes (`<meta http-equiv="refresh" content="600">`).

### hospitals.json

Array of `{ "domain": "...", "name": "..." }` objects. Add or remove entries here to change which portals are monitored.

### deploy.sh

Creates a throwaway Git repo inside `dist/`, commits everything, and force-pushes to the `gh-pages` branch on GitHub. GitHub Pages serves that branch at https://schellevis.github.io/epdchecker.

---

## Important constants (check.js)

| Constant | Value | Meaning |
|---|---|---|
| `CONCURRENCY` | `5` | Parallel hospital checks |
| `HTTP_TIMEOUT` | `12000` ms | Fetch abort timeout |
| `NAV_TIMEOUT` | `20000` ms | Playwright navigation timeout |
| `HIX365_IP` | `20.86.217.65` | IP address that identifies HIX365-hosted portals |

---

## Status badge mapping

| HTTP status | Label | CSS class |
|---|---|---|
| `200` | Online | `ok` |
| `503` | 503 – Niet beschikbaar | `err503` |
| `0` | Geen verbinding | `err000` |
| other | HTTP `<status>` | `errOther` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/schellevis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
