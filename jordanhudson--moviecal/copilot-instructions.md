## moviecal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow

- Only commit or push when the user explicitly asks in the current message (e.g., "commit", "push", "commit and push"). Never carry forward commit/push intent from earlier messages.

## Project Overview

MovieCal is a TypeScript web scraper that collects movie screening times from Vancouver cinema websites, stores them in PostgreSQL, and displays them in a timeline web interface.

## Commands

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript to dist/
npm start            # Run compiled JavaScript from dist/
npm run scrape       # Run full scraping job (all venues + TMDB + DB save)
npm run migrate      # Run database migrations
npm run repair       # Re-clean titles + backfill missing TMDB data
npm run clear        # Clear all data from database
npm run drop         # Drop all tables from database
npm run server       # Start web server on http://localhost:3000
```

## Deployment

Deployed on Fly.io as `movieclock`.

**Auto-deploy**: Pushing to `main` triggers GitHub Actions (`.github/workflows/deploy.yml`) which runs `flyctl deploy --remote-only`. Do NOT run `fly deploy` manually — just push and GHA handles it.

**Manual commands** (for debugging/repair only):
```bash
fly status -a movieclock                      # Check app status
fly ssh console -a movieclock -C "command"   # Run command on server
fly logs -a movieclock                        # View logs
```

**Important**: `tsx` is not available on the production image (it's a dev dependency). To run scripts on prod, use the compiled JS: `fly ssh console -a movieclock -C "node dist/db/repair.js"` (NOT `npm run repair`).

**Configuration** (`fly.toml`):
- `min_machines_running = 1` - Keeps one machine always running for cron jobs
- Release command runs migrations on deploy (`node dist/db/migrate.js`)

## Architecture

### ES Modules Configuration

- Project uses `"type": "module"` in package.json
- **CRITICAL**: All imports must include `.js` extension, even when importing `.ts` files
  - Example: `import { Movie } from './models.js'` (NOT `'./models'` or `'./models.ts'`)
- TypeScript compiles `.ts` → `.js` but import statements must already reference `.js`

### Entry Points

1. **`src/scrape.ts`** - Production scraping job
   - Runs all scrapers in parallel (VIFF, Rio, Cinematheque, Park, Hollywood, Cineplex)
   - Re-cleans existing movie titles (see Title Cleaning below)
   - Enriches new movies with TMDB and Letterboxd data
   - Saves to PostgreSQL using delete-and-reinsert per scraper
   - Use `npm run scrape` to run, or `npm run scrape {name}` for a single scraper

2. **`src/server.ts`** - Hono web server
   - Routes requests to page renderers
   - Hosts admin API endpoints for TMDB/Letterboxd fix-match
   - Runs cron job every 2 hours to scrape
   - After 10pm Pacific, home page auto-shows tomorrow's screenings
   - Runs on port 3000

### Web Pages

Page rendering is in `src/pages/`:

- **`src/pages/index.ts`** - Home page (`/`)
  - Desktop: Timeline view with theatre rows and time-positioned screening blocks
  - Mobile (<800px): Listing view with theatre sections listing movies
  - Cineplex venues collapse multiple auditoriums into one group
  - Query by date: `/?date=YYYY-MM-DD`

- **`src/pages/movie.ts`** - Movie detail page (`/movie/:id`)
  - Shows poster, title, year, runtime, director, TMDB + Letterboxd links
  - Chronological list of future screenings with notes displayed
  - Hidden TMDB fix-match modal (10 clicks on poster to activate, requires `ADMIN_TOKEN`)

- **`src/pages/theatre.ts`** - Theatre detail page (`/theatre/:name`)
  - Lists all future screenings at this theatre with notes displayed

- **`src/pages/movies.ts`** - Movies page (`/movies`)
  - All movies with upcoming screenings, grouped by movie
  - Client-side sort: Date Added, Name, Popularity (TMDB)
  - Client-side theatre filtering (persisted in localStorage)

- **`src/pages/all-movies.ts`** - Internal movies page (`/internal-movies`)
  - Admin-oriented movie list with TMDB fix-match modal

- **`src/pages/layout.ts`** - Shared page layout/shell
  - Nav bar with search (searches movies with upcoming screenings)
  - Cloudflare Web Analytics

- **`src/pages/tmdb-modal.ts`** - Shared TMDB fix-match modal component

### API Endpoints

- `GET /api/movie/:id/tmdb-search` - Search TMDB (requires `ADMIN_TOKEN`)
- `POST /api/movie/:id/tmdb-update` - Fix TMDB match for a movie (requires `ADMIN_TOKEN`)
- `POST /api/movie/:id/letterboxd-update` - Fix Letterboxd URL for a movie (requires `ADMIN_TOKEN`)
- `GET /robots.txt` - Robots file with sitemap reference
- `GET /sitemap.xml` - Dynamic sitemap of movies and theatres with future screenings

### Database Layer

**Stack**: PostgreSQL + Kysely (type-safe SQL query builder) + pg driver

**Schema** (defined in `migrations/*.sql`, types in `src/db/schema.ts`):

`movie` table:
- `id`, `title`, `year`, `director`, `runtime`
- `tmdb_id`, `tmdb_url`, `poster_url`, `tmdb_popularity` — from TMDB
- `letterboxd_url` — from Letterboxd (`'MISS'` = searched but not found, `null` = not yet searched)
- `created_at`, `updated_at`

`screening` table:
- `id`, `movie_id`, `datetime`, `theatre_name`, `booking_url`
- `note` — extracted from title cleaning (e.g. "Advance Screening", "4K Restoration")
- `created_at`, `updated_at`

**Files**:
- `src/db/connection.ts` - Database connection and config (reads from `.env`)
- `src/db/schema.ts` - TypeScript types for Kysely matching the DB tables
- `src/db/migrate.ts` - Migration runner (runs SQL files in `migrations/`)
- `src/db/repair.ts` - Re-cleans existing titles (shared logic with scrape), then backfills missing TMDB data
- `src/utils/reclean.ts` - Shared re-clean logic used by both scrape and repair
- `src/db/clear.ts` - Deletes all data
- `src/db/drop.ts` - Drops all tables
- Migrations stored in `migrations/*.sql` (use `IF NOT EXISTS`/`IF EXISTS` for idempotency)

### Global Data Models

All scrapers must return data using standardized models (`src/models.ts`):

```typescript
interface Movie {
  id: number | null;
  title: string;
  year: number | null;
  director: string | null;
  runtime: number | null;
}

interface Screening {
  id: number | null;
  datetime: Date;
  theatreName: string;
  bookingUrl: string;
  note: string | null;    // Extracted by title cleaner (e.g. "4K Restoration")
  movie: Movie;
}
```

### Timezone Handling

**Important**: The system stores times as naive timestamps representing Pacific time. The server runs in UTC on Fly.io.

- **VIFF, Cinematheque, Park, Hollywood**: Return times without timezone info - these are parsed as local time (UTC on server), which works correctly because the times are stored/displayed as-is
- **Rio**: Returns UTC times with `+00:00` offset - these must be converted to Pacific-naive format using `utcToPacificNaive()` in `rio-scraper.ts`
- **Cineplex**: Returns local Pacific time without timezone info - parsed with `parsePacificNaive()`

If adding a new scraper that returns UTC times, use the same pattern as Rio to convert to Pacific-naive format.

### Title Cleaning

**`src/utils/title-cleaner.ts`** strips parenthesized annotations (5+ chars at end of title) from movie titles and returns them as screening notes. Also decodes HTML entities (e.g. `&#8217;` → `'`).

`cleanMovieTitle("Backrooms (Advance Screening)")` returns `{ title: "Backrooms", note: "Advance Screening" }`.

**TMDB verification** (`verifyTitleCleaning()` in `src/utils/tmdb.ts`): The catch-all pattern can false-positive on foreign films with English subtitles in parens (e.g. "Él (This Strange Passion)"). `verifyTitleCleaning()` wraps `cleanMovieTitle()` and checks TMDB before stripping:
1. If the movie already has a `tmdb_id`: searches TMDB for the note content and checks if any result matches the existing `tmdb_id`. If so, the parens are an alternate title — keep them.
2. If no `tmdb_id`: searches TMDB for the full uncleaned title, checks for exact title match OR whether searching the note content returns the same movie as the uncleaned title search.

This is the **single verification function** used by both scrape and repair.

**Re-cleaning** (`recleanExistingTitles()` in `src/utils/reclean.ts`): Both scrape and repair runs re-clean all existing movie titles in the DB using the shared re-clean logic:
1. Calls `verifyTitleCleaning()` for each existing title
2. If the cleaned title matches another existing movie, screenings are merged and the stale record is deleted
3. If TMDB or Letterboxd data is missing on the renamed movie, it retries the lookup with the cleaned title

This means adding a new pattern to the title cleaner is safe — just add the regex and the next scrape or repair run fixes everything up.

### TMDB Integration

**Shared module**: `src/utils/tmdb.ts` contains `getTMDBMovieDetails()`, `tmdbDetailsToMovieFields()`, and `verifyTitleCleaning()`. `tmdbDetailsToMovieFields()` is the **single place** that maps a TMDB API response to DB column values (`tmdb_id`, `tmdb_url`, `poster_url`, `runtime`, `year`, `tmdb_popularity`). When adding a new TMDB field, update `tmdbDetailsToMovieFields` and all code paths pick it up:
- New movie insert in `scrape.ts`
- Re-clean retry in `scrape.ts`
- TMDB fix-match endpoint in `server.ts`
- Repair script in `db/repair.ts`

**Search**: `scrape.ts` also has `searchTMDB()` which searches TMDB by title+year, filters out shorts (<60 min), and picks the best runtime match.

**Letterboxd**: `scrape.ts` has `searchLetterboxd()` which tries slug-based URL lookups on letterboxd.com. Stores `'MISS'` for not-found to distinguish from not-yet-searched (`null`).

Requires `TMDB_API_TOKEN` in `.env` (optional - scraper works without it).

### Scraper Patterns

Scrapers use one of two approaches depending on whether the venue has an API:

#### API-Based Scrapers (Preferred)

**When to use**: If the venue has a public API or WordPress REST API endpoint

**Pattern**:
1. **Define venue-specific interfaces** matching the API response structure
2. **Export async function**: `export async function scrape{Venue}(): Promise<Screening[]>`
3. **Fetch from API**: Use `fetch()` to call the API endpoint
4. **Parse response**: Extract film title, datetime (usually ISO 8601), booking URL, venue
5. **Clean title**: Use `cleanMovieTitle()` to extract title and note
6. **Convert to global models**: Map API data to `Screening[]`

**Example (API-based)**:
```typescript
import { Movie, Screening } from '../models.js';
import { cleanMovieTitle } from '../utils/title-cleaner.js';

export async function scrapeVenue(): Promise<Screening[]> {
  const response = await fetch('https://venue.com/api/events');
  const events = await response.json();

  return events.map(event => {
    const { title, note } = cleanMovieTitle(event.title);
    return {
      id: null,
      datetime: new Date(event.start_time),
      theatreName: 'Venue Name',
      bookingUrl: event.booking_url,
      note,
      movie: { id: null, title, year: null, director: null, runtime: null }
    };
  });
}
```

#### Puppeteer-Based Scrapers (Fallback)

**When to use**: If no API is available (must scrape HTML)

**Pattern**:
1. **Define venue-specific interfaces** matching venue's HTML structure
2. **Export async function**: `export async function scrape{Venue}(): Promise<Screening[]>`
3. **Use Puppeteer**:
   - Launch headless browser: `puppeteer.launch({ headless: true })`
   - Navigate: `page.goto(url, { waitUntil: 'networkidle2', timeout: 30000 })`
   - **Wait 3 seconds** after page load: `await new Promise(resolve => setTimeout(resolve, 3000))`
     - Required for JavaScript-rendered content to hydrate
   - Extract data via `page.evaluate()` using CSS selectors
4. **Parse dates with custom function**: Each venue has unique date/time format
5. **Clean title**: Use `cleanMovieTitle()` to extract title and note
6. **Convert to global models**: Transform venue-specific data to `Screening[]`
7. **Always close browser**: Use try/finally to ensure `browser.close()`

### Current Scrapers

**API-Based** (fast, reliable):
- **`viff-scraper.ts`** - VIFF Centre
  - API: `https://viff.org/wp-json/v1/attendable/calendar/instances`
  - WordPress custom endpoint with calendar events
  - Returns HTML in JSON (extract title/URL via regex)
  - Venue mapping: `VENUE_NAMES` lookup with fallback formatter

- **`rio-scraper.ts`** - Rio Theatre
  - API: `https://riotheatre.ca/wp-json/barker/v1/listings`
  - WordPress custom plugin endpoint
  - Clean JSON with `tickets_link` for booking URLs
  - Date range: 1 month back, 2 months forward
  - **Note**: Uses `utcToPacificNaive()` to convert UTC times to Pacific-naive format

- **`cineplex-scraper.ts`** - Cineplex Theatres (Fifth Avenue, International Village, Scotiabank, Langley)
  - API: `https://apis.cineplex.com/prod/cpx/theatrical/api/v1/showtimes?language=en&locationId={theatreId}&date={M/D/YYYY}`
  - Azure API Management endpoint, requires header: `ocp-apim-subscription-key: 477f072109904a55927ba2c3bf9f77e3` (public key embedded in cineplex.com frontend)
  - Returns movies with sessions (showtimes) per theatre per day; fetches 7 days
  - `showStartDateTime` is local Pacific time — parsed with `parsePacificNaive()`
  - Theatre name format: `"{venue} {auditorium}"` e.g. `"Fifth Ave Aud #3"`, `"Intl Village Aud 05"`
  - Uses `deeplinkUrl` for booking URLs
  - **To find theatre IDs**: `GET https://apis.cineplex.com/prod/cpx/theatrical/api/v1/theatres?language=en&city=Vancouver&latitude=49.2514&longitude=-123.0972` (same API key header). Returns `theatreId` and `theatreName` for nearby theatres. This is not used by the scraper but is useful for adding new Cineplex locations.

- **`hollywood-scraper.ts`** - Hollywood Theatre
  - Uses cheerio to scrape event pages
  - Crawls individual event detail pages with a delay between requests

**Puppeteer-Based** (slower, more brittle):
- **`cinematheque-scraper.ts`** - Cinematheque
  - URL: `https://thecinematheque.ca/films/calendar`
  - Custom PHP application, no API available
  - Scrapes `#eventCalendar` DOM structure

- **`park-scraper.ts`** - Park Theatre
  - URL: `https://tickets.theparktheatre.ca/`
  - Igniter ticketing system, no API available
  - Scrapes event listings from HTML

All scrapers run in parallel via `scrape.ts` with error handling (failed scrapers don't break the job).

## Adding New Scrapers

### Step 1: Investigate for APIs

**Always try to find an API first** - they're faster, more reliable, and easier to maintain.

**Investigation process**:
1. **Check for WordPress**: Try `https://venue.com/wp-json/` to see available endpoints
   - Look for custom namespaces beyond `wp/v2`
   - Example: VIFF uses `/v1/attendable/calendar/instances`
   - Example: Rio uses `/barker/v1/listings`

2. **Monitor network traffic**: Use Puppeteer debug script to capture API calls:
   ```typescript
   page.on('response', async response => {
     const url = response.url();
     if (url.includes('json') || url.includes('api')) {
       console.log(url, await response.text());
     }
   });
   ```

3. **Check for React/Vue data**: Look in browser DevTools for:
   - Global variables: `window.__INITIAL_STATE__`, `window.eventData`, etc.
   - Script tags with `type="application/json"`
   - React fiber props via DOM inspection

4. **Try common API patterns**:
   - `/api/events`
   - `/api/screenings`
   - `?format=json` (Squarespace, Jekyll)
   - GraphQL endpoints

### Step 2: Implement Scraper

1. Create `src/scrapers/{venue}-scraper.ts`
2. Use **API pattern** if API found, otherwise **Puppeteer pattern**
3. Follow patterns documented in "Scraper Patterns" section above
4. Import in `src/scrape.ts` and add to the `scrapers` registry object
5. Run `npm run scrape {scraper-name}` to test the new scraper

**Note**: If a venue later adds an API, refactor from Puppeteer to API-based approach for better performance.

### Step 3: Add Theatre to UI

Add the new theatre name(s) to `THEATRE_ORDER` in `src/server.ts`. This controls which theatres appear on the home page timeline and their display order.

**Note**: If a venue has multiple screens/auditoriums with separate `theatreName` values (e.g., "Fifth Ave Aud #1", "Fifth Ave Aud #2"), add each one to the list. Cineplex venues also need an entry in `CINEPLEX_VENUES` to be collapsed into a single listing group.

## Environment Variables

Create `.env` file (see `.env.example`):
```
DB_HOST=localhost
DB_PORT=5432
DB_NAME=moviecal
DB_USER=postgres
DB_PASSWORD=postgres
TMDB_API_TOKEN=your_token_here  # Optional — scraper works without it
ADMIN_TOKEN=your_token_here     # Required for TMDB/Letterboxd fix-match endpoints
```

## Tech Stack

- **TypeScript**: Strict mode, ES2022 target
- **Puppeteer**: Headless browser automation for scraping
- **Cheerio**: HTML parsing (Hollywood scraper)
- **Kysely**: Type-safe SQL query builder
- **PostgreSQL**: Database
- **Hono**: Web framework for timeline UI
- **node-cron**: Scheduled scraping (every 2 hours)
- **tsx**: TypeScript execution for development
- **dotenv**: Environment variable management

## UI Design

**Color scheme** (dark mode):
- Background: `#1e1e1e`
- Content areas: `#262626`
- Text: `#c5c5c5`
- Muted text: `#707070` - `#888`
- Borders: `#353535`
- Accent (screening blocks, buttons): `#4a7c7c` (muted teal)
- Links: `#6a9a9a`

---
> Source: [jordanhudson/moviecal](https://github.com/jordanhudson/moviecal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
