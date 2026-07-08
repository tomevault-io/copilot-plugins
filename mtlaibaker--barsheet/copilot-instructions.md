## barsheet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A volunteer shift scheduling web app for DCC (Dunany Country Club) bar events. Admins set up events and time slots; volunteers sign up for shifts. Built as a Google Apps Script Web App backed by a Google Sheet.

## File structure

| File | Purpose |
|------|---------|
| `Code.gs` | Apps Script backend — routing (`doGet`), all sheet read/write functions |
| `Volunteer.html` | Public-facing page — browse events, sign up, remove own name |
| `Admin.html` | Admin page (served at `?page=admin`) — create/clone/delete events, manage slots |
| `Style.html` | Shared CSS included by both pages via `<?!= include('Style') ?>` |

## Data model

The Google Sheet has one tab named **"Schedule"** with one row per time slot per event:

```
Date | EventName | Notes | SlotTime | MaxVols | IsOpen | Vol1 | Vol2 | Vol3 | Vol4
```

- `Date` — stored as text `YYYY-MM-DD` for lexicographic sorting
- `MaxVols` — 1–4 (default 2); controls how many of Vol1–Vol4 are active
- `IsOpen` — TRUE/FALSE; closed slots are shown greyed-out on volunteer page
- Multiple rows share the same Date+EventName to form a single event

The `rowIndex` (1-based, including header row) is used as the stable row identifier for all updates and deletes.

## Key architecture notes

- **No separate server** — Apps Script handles everything. Deploy as "Execute as: Me, Access: Anyone."
- **`include()` pattern** — `Style.html` is not a standalone file; it is injected into both HTML templates at render time using `<?!= include('Style') ?>`.
- **Client↔server calls** use `google.script.run` with `withSuccessHandler` / `withFailureHandler`. All server functions return `{ ok: true }` or `{ ok: false, error: '...' }`.
- **Row indices shift on delete** — `deleteSlot` and `deleteEvent` must use the current rowIndex from the last `getSchedule()` call. The UI always reloads after any mutation to keep indices fresh.
- Sorting is done server-side in `getSchedule()` by comparing `YYYY-MM-DD` strings.

## Deployment setup

1. Create a new Google Sheet for 2026 with a tab named **Schedule**
2. Open **Extensions → Apps Script** in that sheet (or create a standalone project)
3. Paste each file into the corresponding Apps Script file (`Code.gs`, `Volunteer.html`, `Admin.html`, `Style.html`)
4. Set `SPREADSHEET_ID` in `Code.gs` to the Sheet's ID (from its URL)
5. **Deploy → New deployment** → Type: Web App, Execute as: Me, Access: Anyone
6. Volunteer URL: `https://script.google.com/macros/s/{ID}/exec`
7. Admin URL:     `https://script.google.com/macros/s/{ID}/exec?page=admin`

After any change to `Code.gs`, create a **new deployment version** — the existing URL keeps serving the old code until a new version is published.

## Standard time slots

The UI presents these defaults (users can also enter custom times):
`11am–1pm · 1pm–3pm · 3pm–5pm · 5pm–7pm · 7pm–9pm · 9pm–11pm · 11pm–Closing`

---
> Source: [mtlaibaker/barsheet](https://github.com/mtlaibaker/barsheet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
