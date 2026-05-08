## firstproject

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Automated concall tracker that scrapes upcoming investor conference calls from Screener.in, extracts dial-in numbers from PDF announcements, and syncs to Google Sheets + Calendar. Runs daily at 7 AM IST via GitHub Actions.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the scraper locally (requires env vars)
SCREENER_USERNAME=xxx SCREENER_PASSWORD=xxx python screener_login.py

# Run manually via GitHub Actions
# Go to Actions tab → "Update Screener Concalls" → Run workflow
```

## Required Environment Variables

- `SCREENER_USERNAME` / `SCREENER_PASSWORD` - Screener.in login credentials
- `GOOGLE_CREDENTIALS_BASE64` - Base64-encoded Google service account JSON
- `EMAIL_USERNAME` / `EMAIL_PASSWORD` - Gmail credentials for notifications (GitHub Actions only)

## Architecture

### Main Script: `screener_login.py`

Single-file architecture with clear function separation:

1. **Configuration** (lines 35-54) - All constants at top: `TARGET_CONCALL_COUNT`, `CALENDAR_ID`, timeouts
2. **Google Auth** - `get_google_credentials()` handles both local file and base64 env var
3. **Scraping Flow**:
   - `login_to_screener()` → `scrape_all_concalls()` → `extract_all_phone_numbers()`
   - Uses Selenium with WebDriverWait for reliability
4. **Output**:
   - `write_to_google_sheets()` - Creates/updates sheet with formatting
   - `sync_to_google_calendar()` - Smart sync with duplicate detection via `concall_id` hash
   - Color-codes events based on watchlist membership and overlaps
5. **Watchlist Integration**:
   - `scrape_watchlists()` - Fetches company lists from user's Screener.in watchlists
   - `get_watchlist_color()` - Assigns colors based on watchlist membership
   - `is_my_stonks_company()` - Checks if company is in My Stonks watchlist

### Watchlist Color Coding

Events are color-coded based on watchlist membership:
- **My Stonks** → Tomato (color ID 11) - also copied to main calendar
- **Core Watchlist** → Cycles through Flamingo, Tangerine, Banana (IDs 4, 6, 5)
- **Overlapping non-watchlist events** → Lavender, Sage, Grape, Peacock, Graphite, Blueberry, Basil

### Main Calendar Sync

My Stonks events are automatically copied to the main calendar (`moonkanish@gmail.com`) if they don't already exist there. Duplicate detection uses the same `concall_id` hash mechanism.

### Key Design Decisions

- **Duplicate detection**: MD5 hash of `company_date_time` stored in calendar event's `extendedProperties`
- **Retry logic**: HTTP requests use `urllib3.Retry` with exponential backoff
- **Past events**: Calendar sync skips events where `start_dt < datetime.now()`
- **Rate limiting**: 0.3s delay between PDF downloads to avoid throttling
- **Always use IST timezone** (`Asia/Kolkata`) for calendar events

### GitHub Actions Workflow

`.github/workflows/update-concalls.yml`:
- Cron: `30 1 * * *` (7 AM IST = 1:30 AM UTC)
- Sends success/failure emails to `moonkanish@gmail.com`
- Creates GitHub issue on failure with `bug` and `automation` labels

## Google API Setup

Service account needs these scopes:
- `spreadsheets` - Read/write sheets
- `drive` - Create new sheets
- `calendar` - Create/update events

Calendar must be shared with service account email with "Make changes to events" permission.

---
> Source: [AM1403x/firstproject](https://github.com/AM1403x/firstproject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
