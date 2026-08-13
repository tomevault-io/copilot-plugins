## group-events

> A Houston-focused events aggregator that scrapes venue websites, uses LLM extraction, and presents events on a public calendar with an internal review dashboard.

# Group Events — Houston Music & Events App

## Project Overview
A Houston-focused events aggregator that scrapes venue websites, uses LLM extraction, and presents events on a public calendar with an internal review dashboard.

**Live URLs:** houeventlist.com (Houston, live) — atxeventlist.com (Austin, in progress)
**Deployed on:** Railway
**Repo:** Big-Junk:group-events

---

## File Structure
```
group-events/
├── universal_scraper.py      # Main scraper — Selenium + LLM extraction
├── review_dashboard.py       # Flask app — review dashboard + public calendar + API
├── templates/
│   ├── review.html           # Review dashboard UI
│   └── calendar.html         # Public calendar UI
├── Dockerfile                # Web service (Flask)
├── Dockerfile.cron           # Scraper service
├── requirements.txt
└── CLAUDE.md
```

**Local paths:**
- Big-Junk: `/Users/drewcollier/Documents/Work/Xcode/Web Data Dev/AI/group-events/`
- iMac: `/Users/drew/Documents/Python/GRClaude1/`

---

## Tech Stack
- **Backend:** Python 3.11, Flask
- **Scraping:** Selenium + ChromeDriver (headless Chrome)
- **LLM:** OpenAI GPT-4o-mini (default), GPT-4o, Claude Sonnet, Groq/Llama 3.1 70B
- **DB:** PostgreSQL on Railway (psycopg2), SQLite fallback locally
- **Frontend:** Vanilla JS, Jinja2 templates
- **Deployment:** Railway (web service + daily cron + weekly cron)

---

## Database Schema

### events
```sql
id, name, date, doors_time, start_time, venue, location, city, state,
price, ticket_url, description, genre, confidence, notes, status,
created_at, approved_at, event_type, visible
```

**status values:** `pending`, `approved`, `rejected`, `canceled`, `possible_duplicate`
**event_type values:** `music`, `comedy`, `open_mic`, `happy_hour`, `private_event`, `other`
**visible:** boolean, default true for music/comedy, false for others

### past_events
Same schema as events plus `archived_at` timestamp.

### venue_cache
```sql
venue_key, content_hash, last_scraped
```
Used for hash-based change detection to skip LLM when page unchanged.

---

## Scraper Architecture

### Scrape Modes
```bash
python3 universal_scraper.py --mode daily       # Hash check, new events only
python3 universal_scraper.py --mode weekly      # Full scrape + canceled event detection
python3 universal_scraper.py --mode onboard     # New venue, skip hash, filter past events
```

### Additional Flags
```bash
--venue <key>        # Scrape single venue (e.g. --venue white_oak)
--llm <provider>     # gpt4o, gpt4o-mini (default), claude, groq
--dry-run            # Extract and print without saving to DB
--auto-approve       # Auto-approve events on insert (use with onboard)
```

### LLM Providers
- `gpt4o-mini` — default, best cost/quality balance
- `gpt4o` — fallback for complex venues
- `claude` — Claude Sonnet 4, good quality, higher cost
- `groq` — Llama 3.1 70B via Groq, free tier available

### Key Functions
- `scrape_page()` — Selenium page fetch with scroll + pagination support
- `get_content_hash()` / `get_stored_hash()` / `update_stored_hash()` — hash caching
- `pre_filter_dates()` — informational date logging (not used for filtering)
- `extract_events_with_llm_raw()` — main LLM extraction function
- `parse_white_oak_html()` — custom BeautifulSoup parser for White Oak
- `save_to_database()` — insert with exact + partial duplicate detection
- `check_canceled_events()` — weekly mode canceled event flagging

### Duplicate Detection
- **Exact match** (name + date + venue + start_time) → auto-skip
- **URL match** (`ticket_url` and/or `event_url` identifies the same event) → update in place (title/time/openers/price/URL drift all get refreshed on the existing row). Whether each URL field requires `start_date` to also match is controlled per-venue by three toggles stamped onto each event dict in `scrape_venue` — `dupe_use_ticket_url`, `dupe_use_event_url`, `dupe_use_start_date` (all default `True`, i.e. today's safe default: a URL match alone isn't enough, the date must agree too). A venue only gets `dupe_use_start_date: False` after confirming its URL is genuinely unique per occurrence (not a reused comic-profile or generic template page — see Improv Houston and Dan Electro's below) — only then does a URL match with a *different* date get treated as a correction (updates `start_date` in place) rather than falling through to a duplicate insert. If the corrected row was already `approved` and the venue isn't `auto_approve`, the correction also reverts it to `pending` so a human reviews the new date before it's public again.
- **Partial match** (same venue + date, fuzzy name similarity ≥ per-venue `duplicate_threshold`, default 0.5) → insert as `possible_duplicate`
- **Weekly automated sweep** (`--mode weekly` only) — `detect_existing_duplicates()` (same-date fuzzy-name pairs) and `detect_url_date_conflicts()` (same ticket_url/event_url, different start_date — the backstop for venues that don't have `dupe_use_start_date: False`) both run automatically, flagging matches as `possible_duplicate` for review. Added 2026-07-30 after a Heights Theater date-drift bug (LLM occasionally mis-extracted the wrong date for an already-known show on re-scrape) let 51 duplicate rows accumulate over ~6 weeks undetected — neither existing check compares URLs across *different* dates, so nothing ever flagged them until a manual review caught it.
- Scraping stats (`scrape_stats` table, surfaced on `/venue_health`) track `updated_events` and `date_corrections` per venue per run — a venue with a recurring `date_corrections` count is a sign its extraction is unstable, not that its shows keep genuinely moving. Feeds into that venue's health flag (yellow at ≥3/30d, red at ≥10/30d).

---

## Venues
```python
bad_astronaut       # Bad Astronaut Brewing Co.
white_oak           # White Oak Music Hall (custom BS4 parser, locations: Downstairs/Upstairs/Lawn)
heights_theater     # Heights Theater
last_concert        # Last Concert Cafe
dan_electros        # Dan Electro's Guitar Bar
713_music_hall      # 713 Music Hall
continental_club    # Continental Club Houston + Big Top Charlies (shared Timely page)
woodland_pavilion   # Cynthia Woods Mitchell Pavilion
house_of_blues      # House of Blues Houston (consolidated, location field used for rooms)
smart_financial     # Smart Financial Centre
improv_tx           # Improv Houston (paginated, filters Houston-only from TX page)
riot_comedy         # The Riot Comedy Club
bayou_music_center  # Bayou Music Center
secret_group        # The Secret Group (comedy/music, EaDo)
hotel_lucine        # Hotel Lucine (Galveston)
echoes_htx          # Echoes (Montrose, site posts same-day ephemeral events — location on-site TBD, under active observation)
scout_bar           # Scout Bar
armadillo_palace    # Goode Company Armadillo Palace
moth                # MOTH Presents
numbers             # Numbers (Montrose nightclub)
miller_outdoor      # Miller Outdoor Theatre (own site — previously only a byproduct of the city ICS feed)
docs_jazz           # Doc's Jazz Club
arena_theatre       # Arena Theatre (event names/dates parsed from URL slugs — no visible listing text on site)
dosey_doe           # Dosey Doe - The Big Barn (The Woodlands)

# Austin
emos_austin           # Emo's Austin
stubbs_austin         # Stubb's Bar-B-Q
acl_live              # ACL Live at The Moody Theater
moody_center          # Moody Center
mohawk_austin         # The Mohawk (url is the site root, not /events/ — see VENUES.md)
cactus_cafe           # Cactus Cafe (UT Austin) — pending events need manual cleanup, see VENUES.md Open Items
come_and_take_it_live # Come and Take It Live
scoot_inn_austin      # Historic Scoot Inn
vulcan_gas_company    # Vulcan Gas Company (mixed per-show ticketing platforms)
broken_spoke          # Broken Spoke (no ticketing, free/cover shows)
cap_city_comedy       # Cap City Comedy Club (Seat Engine)
velveeta_room         # The Velveeta Room (Seat Engine, partial capture)
fallout_theater       # Fallout Theater
ballroom_29th_street  # 29th Street Ballroom (formerly Spider House Ballroom)
esthers_follies       # Esther's Follies (weekly revue, not a touring-act calendar)
cheer_up_charlies     # Cheer Up Charlie's (mechanism works, currently 0 events on site)
```
See [VENUES.md](VENUES.md) for platform/config detail and capture-quality notes per venue. Six more Austin venues (`antones_austin`, `continental_club_austin`, `cboys`, `white_horse_austin`, `hole_in_the_wall`, `saxon_pub`) have config in place but aren't onboarded yet — see VENUES.md's "Austin — needs follow-up work" section for each one's specific blocker.

---

## Railway Deployment
- **Web service:** `python review_dashboard.py` — Flask app
- **Daily cron:** `python universal_scraper.py --mode daily` — runs Tue-Sun at 6am
- **Weekly cron:** `python universal_scraper.py --mode weekly` — runs every Monday at 6am

**Environment variables needed:**
- `DATABASE_URL` — Railway PostgreSQL public connection string
- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY` (optional, for --llm claude)
- `GROQ_API_KEY` (optional, for --llm groq)

---

## Flask Routes
```
GET  /                    # Review dashboard (pending events)
GET  /calendar            # Public calendar (approved + visible events)
POST /approve             # Single event approve
POST /batch_approve       # Batch approve
POST /reject              # Single event reject
POST /batch_reject        # Batch reject
POST /update              # Update event fields
POST /api/filter_events   # Dashboard filter endpoint
GET  /api/venues          # Venue list for filters
GET  /api/genres          # Genre list for filters
GET  /stats               # DB stats
GET  /debug               # Debug info
```

---

## Known Issues / To-Do

### In Progress
- [ ] Event title cleanup — strip `(SOLD OUT)`, `(NEW DATE)`, venue references, Riot suffixes
- [ ] Add `sold_out`, `date_changed`, `openers` fields to DB and LLM prompt
- [ ] Archive logic — move approved past events to `past_events` table (weekly job)
- [ ] Location filter in review dashboard
- [ ] Multi-night event expansion for Improv (e.g. "Apr 17-18" → two events)
- [ ] Toyota Center needs a dedicated HTML parser (like White Oak's) — every event's ticket button renders identical anchor text ("More Info & Ticket Options") with no distinguishing per-event label, so the LLM can't reliably match hrefs to events from the flattened page text + link list. This causes a systematic off-by-one misattribution of `ticket_url`/`event_url` between neighboring events (confirmed both in fresh scrapes and in historical data — e.g. two different past events shared an identical AXS ticket link). DB is intentionally left empty for this venue until fixed rather than serve wrong ticket links.
- [ ] Review the city-calendar-sourced venues (`houcalendar`, `houston_city_calendar`) for cleanup — large unreviewed pending backlogs (e.g. Houston Museum of Natural Science, Sabine Street Studios, City Hall each in the 70-100 pending range). Focus first on parks with performing-arts stages (Miller Outdoor Theatre, Memorial Park, etc.) since those are the most likely to have real music/performing-arts events worth surfacing on the public calendar, versus generic city/civic listings.
- [ ] Echoes HTX (`echoes_htx`) has never captured a single real event since onboarding (confirmed 0 rows, ever, in both `events` and `past_events` as of 2026-07-26). The `/event-calendar` page's Wix calendar widget (`calendar.boomte.ch`, a cross-origin iframe — `driver.page_source` never sees inside one, which is why the content-hash check was silently reporting "no changes" every run) is confirmed genuinely empty, not just unreachable: checked both its Month and Agenda views directly and its underlying API (`calendar.apiboomtech.com/api/published_calendar`) returns 0 future events. So this isn't a scraping-technique bug — the venue's real event announcements live somewhere else on their site as same-day, short-lived posts (confirmed: a real event was visible on-site earlier 2026-07-26 and gone by the next check). Facebook is ruled out as a source. Plan: manually check the site daily this week to find where on-site the posts appear and how long they stay up, before designing any automated approach. Until then, the afternoon `schedule_group` cron scraping the `/event-calendar` page is a harmless no-op.

### Architecture / Scale
- [ ] Venues table — migrate VENUES dict from scraper to DB
- [ ] Redis + RQ job queue for parallel scraping
- [ ] Scheduler service to replace cron
- [ ] LLM worker separation from scraper workers
- [ ] Target: 100 venues architecture, then scale to 1000+

### Future Features
- [ ] Recurring events engine (open mic, weekly shows)
- [ ] Day-of scrape for time/venue changes
- [ ] `tour_name` field
- [ ] Artist/headliner + openers fields with set times
- [ ] Event detail view
- [ ] Venue discovery automation (Google Places API, Songkick, etc.)
- [ ] User-submitted venue suggestions
- [ ] `possible_duplicate` status filter in dashboard (added to filter dropdown)

### ML Classification (exploratory, not scheduled)
Classical ML (not LLM) is a fit for the specific sub-tasks in the pipeline that are genuinely fixed-label classification problems — not for event extraction itself. Extraction (name/date/ticket_url identification, matching links to the right event) should stay LLM- or parser-based: every venue's page layout differs and changes over time, and a trained classifier only generalizes to layouts resembling its training data, unlike an LLM which needs zero venue-specific training. That tradeoff gets worse, not better, at the 100→1000 venue target.
- [ ] Duplicate-detection classifier — replace the fixed fuzzy-name-ratio threshold (`SequenceMatcher`, hand-tuned per venue via `duplicate_threshold`) with a learned model over engineered features (name similarity, date/time delta, venue match, shared ticket_url/event_url). Label source: dashboard approve/reject actions on `possible_duplicate` rows — though this is a noisy signal (a reject doesn't always mean "confirmed duplicate"), and there isn't enough resolved volume yet to train on. Motivated by concrete false positives seen in practice (e.g. "Bob Schneider" vs "Bob Schneider – Early Show", "Texas Elite Auto Showcase" vs "Texas Trucking Show").
- [ ] Canceled-event detection — `check_canceled_events` also uses a fixed fuzzy-match threshold (0.85); same category of candidate as duplicate detection, same caveat about needing more labeled history first.

### Ground-Truth Date Verification (exploratory, not scheduled)
Idea from the 2026-07-30 Heights Theater date-drift cleanup: some ticket platforms (confirmed on Prekindle) expose an authoritative `startDate` via JSON-LD directly on the event page — that's how the correct dates were verified by hand during that cleanup. A venue could fetch that per-event page to confirm/correct the scraped date rather than trusting the flattened listing text alone. Deferred pending an eval of the LLM/compute cost impact (N extra HTTP fetches per scrape, one per event with a `ticket_url`) before committing to it — the weekly `detect_url_date_conflicts()` sweep added the same day is the current mitigation and doesn't have that cost.

---

## Important Notes

### DB Quirks
- Railway PostgreSQL uses `%s` placeholders, SQLite uses `?`
- `DATABASE_URL` uses `postgres://` prefix — must replace with `postgresql://` for psycopg2
- Railway console does not support `LIMIT` in queries
- Always use public connection string locally, not `postgres.railway.internal`

### White Oak Parser
- Uses custom BeautifulSoup parser (`parse_white_oak_html`) instead of LLM — too many events for token limits
- Extracts location from `tw-venue-name` span (Downstairs/Upstairs/Lawn)
- Extracts year from ticket URL href (handles 2027 shows correctly)

### Continental Club / Big Top
- Both venues share one Timely calendar URL
- Bypasses the LLM entirely — `timely_calendar_id` in VENUES config routes this venue straight to `scrape_timely_api`, which hits Timely's API directly
- Timely's own API response tags each event with a `taxonomy_venue` field, which `scrape_timely_api` reads directly to set `venue` per event (falls back to the VENUES config's `name` only if that taxonomy is missing) — the split is Timely's own structured data, not LLM inference

### Improv Houston
- Uses `improv_tx` key, scrapes the all-TX-venues page
- Paginated via `#moreshowsbtn` link detection
- Filters Houston-only events via `venue_instruction`
- Multi-night events (e.g. "Apr 17-18") not fully expanded — known limitation

### HOB Consolidation
- Three HOB rooms consolidated into one venue key (`house_of_blues`)
- Room stored in `location` field (Main Stage, Bronze Peacock, Foundation Room)

### Security
- Never commit API keys or DATABASE_URL to git
- Store in `~/.zshrc` for local dev, Railway env vars for production
- GitHub push protection will block commits containing secrets

---
> Source: [drouston/group-events](https://github.com/drouston/group-events) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
