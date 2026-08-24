## giovanni-cars

> Always read `SESSION.md` first and give a briefing:

# Dublin Toyota — Claude Code Project

## Session Start Instructions
Always read `SESSION.md` first and give a briefing:
- **Project:** one sentence
- **Recently completed:** from SESSION.md
- **Currently in progress:** from SESSION.md, or "Nothing in progress — ask Giovanni what to work on."

---

## Who This Is For
Giovanni Galasso, Toyota Sales Specialist at Dublin Toyota (Dublin, CA). Not a developer — builds tools with Claude Code. Plain language, no jargon.

## The Three Claude Contexts
- **Chat** — general Claude conversation
- **Cowork** — Anthropic's agentic desktop mode, used for day-to-day dealership work (leads, inventory, calendar). Also accessible via claude.ai → Dublin Toyota project (web/phone).
- **Code** (this instance) — builds and maintains the tools Cowork uses

---

## Repo & Live URLs
- **Repo:** https://github.com/giogalasso323-png/giovanni-cars
- **Manager app:** https://giogalasso323-png.github.io/giovanni-cars/manager.html
- **Public site:** https://giogalasso323-png.github.io/giovanni-cars/
- **MCP server (Railway):** https://dublin-toyota-inventory-production.up.railway.app

## Files
- `manager.html` — THE internal app. Always edit this. Never touch `inventory-app.html`.
- `apps-script.js` — Google Apps Script backend. Copy/paste into GAS editor to deploy.
- `mcp-server/index.js` — MCP server (HTTP/Express), deployed to Railway.
- `SESSION.md` — tracks in-progress work. Read every session.
- `CLAUDE.md` — this file.

---

## Stack
- **Frontend:** `manager.html` — single HTML file, dark-themed SPA, ~3800+ lines. Pure HTML/CSS/JS, no framework, no build step. SheetJS via CDN for XLS parsing.
- **Backend:** Google Apps Script (`apps-script.js`) — deployed as web app on a separate project Google account
- **Database:** Google Sheets — SPREADSHEET_ID: `1j0UuQEmmctzNPS3mQoLQGv2lNXlDCVb3Pjh3Q0dJQ30`
- **Storage:** Google Drive (car photos)
- **Hosting:** GitHub Pages (auto-deploys on push to main)
- **MCP server:** Railway (~$5/month, port 8080, env var: SCRIPT_URL)

## Deploy
- **Frontend:** `git add manager.html && git commit -m "..." && git push` → live in ~2 min
- **Backend:** copy apps-script.js into GAS editor → Deploy → Manage Deployments → New Version
- **MCP server:** `git push` → Railway auto-deploys from main branch

---

## Apps Script
- Deployed on a **separate project Google account** (not personal Gmail)
- Original sheet owned by personal account, shared with project account
- Current deployment URL (hardcoded in manager.html):
  `https://script.google.com/macros/s/AKfycbzl1VFO7DbmSfMq_tQnZDxygvdC5rpYFgGyCzbHurLdSSAlBtiPgc73WHGiVSK_SM0T/exec`

## MCP Server (Railway)
- Converted from stdio → HTTP (StreamableHTTPServerTransport + Express) on 2026-06-29
- Root directory in Railway: `mcp-server`
- Railway sets PORT automatically (8080)
- Required env var: `SCRIPT_URL` = Apps Script deployment URL above
- No AUTH_TOKEN currently
- Desktop Cowork config: `C:\Users\gioga\AppData\Roaming\Claude\claude_desktop_config.json`
  Points to: `https://dublin-toyota-inventory-production.up.railway.app/mcp`

---

## Code Style Rules
- Always edit `manager.html` for internal app changes — never `inventory-app.html`
- No comments unless the WHY is non-obvious
- No abstractions beyond what's needed
- No new features beyond what's asked
- Navigate with section markers — grep for `// ===== SECTION:` to jump anywhere

---

## Inventory Tabs
All · Upcoming · Live · FB Posted · Not Posted · Sold · Not Listed · Leads · New Cars · Floor

## Upcoming Tab Logic
`appraisedValue > 0 && !color` — cost imported but not yet live. Exits when used car CSV import sets color.

## Key Data Structures

**COLUMNS (used car inventory):**
`vin, year, make, model, trim, color, mileage, price, stock, fbStatus, websiteStatus, websitePrice, fbDescription, carfaxUrl, edmundsLabel, edmundsBelow, vehicleInfo, vehicleHistory, features, certification, addedDate, lastChecked, fbPostedDate, soldDate, websiteUrl, fbPostedPrice, priceDropped, dis, currentFbPrice, originalPrice, drivePhotoFolder, drivePhotoCount, appraisedValue, certCost, appraiser`

**LEADS_COLUMNS:**
`timestamp, firstName, lastName, phone, vehicle, vin, timeframe, source, status, notes, followUpDate, vehicleList, leadType, inFocus, turnedTo, vehicleNotAvailable, vehicleInterest, turnedToFirst, addedBy, calEventId, lastEdited`

## MCP Tools (for Cowork)
`ping, scrape_inventory, search_inventory, get_inventory, get_high_gross_cars, get_stale_inventory, get_car, update_car_field, get_leads, add_lead, update_lead, set_lead_pipeline, delete_lead, get_new_inventory, get_upcoming_inventory, import_cost_data, import_used_cars, import_new_cars`

---

## Gross Calculation
- Total cost = appraisedValue + $2,000 recon + $500 detail + certCost
- Mileage/age adder (capped $2,500) — matrix in mcp-server/index.js `getMileageAdder()`
- topGross = price − totalCost
- bottomGross = topGross − adder
- Commission = max(gross × 25%, $500 mini)
- 🟢 Has Money = bottomGross ≥ $2,000 | 🟡 Saveable = topGross ≥ $2,000 but bottomGross < $2,000 | 🔴 Mini = topGross < $2,000

---

## Giovanni's Brand (for FB content builds)
- Tagline: "Anyone can show you a vehicle. What most salespeople won't do is stay."
- 5.0 stars, 7 FB reviews — all say: honest, patient, no pressure, holds his word, stress-free
- Quote: "Most salespeople want to sell you a car once. I want to sell you your next three."
- Target community: Toyota outdoor/adventure crowd, Tri-Valley/East Bay (Dublin CA)
- Personal: outdoor guy, has a dog (best social content asset), likes to cook, likes to hunt (keep off social in Bay Area)
- Content pillars: social proof · inventory highlights · adventure/lifestyle · behind the scenes · personal/dog

## Next Phase Builds
1. Daily inventory scrape (automated, overnight)
2. Stale lead alerts (9am daily, Cowork message + email to giogalasso323@gmail.com)
3. Calendar ↔ lead sync (scheduled Claude.ai agent)
4. FB Marketplace batch photo processing
5. FB business page content system

---

## Section Map — manager.html
Grep for `// ===== SECTION:` to jump anywhere. Key sections:
- `SECTION: AUTH` — password/session
- `SECTION: INIT/BOOT` — app startup
- `SECTION: API / DATA LAYER` — fetch, load, sync
- `SECTION: FILTERING & RENDERING` — tabs, table, cards
- `SECTION: LEADS / CRM TAB` — lead management
- `SECTION: GROSS ANALYSIS` — cost import, gross calc
- `SECTION: NEW CARS TAB` — new inventory
- `SECTION: FLOOR TAB` — floor traffic log
- `SECTION: DRAWER` — car detail panel
- `SECTION: FACEBOOK DESCRIPTION` — FB post builder

---
> Source: [giogalasso323-png/giovanni-cars](https://github.com/giogalasso323-png/giovanni-cars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
