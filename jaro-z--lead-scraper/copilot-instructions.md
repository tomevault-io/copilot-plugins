## lead-scraper

> Local app for scraping business leads from Google Places API.

# Lead Scraper

Local app for scraping business leads from Google Places API.

## Quick Commands

- **`.p`** → Prep for shipping. Run in this exact order:
  1. Run `/simplify` on all changed files to clean up the code
  2. Run `npm run build` to verify nothing broke (skip if no build step)
  3. Run `/review` on the branch diff to catch logic errors or security issues
  4. Check for sensitive files (`.env`, API keys, secrets, credentials) not in `.gitignore` and add them
  5. List what changed and tell me what to test in the browser
  6. Run `/fewer-permission-prompts` and suggest new permissions to add to my config
- **`.c`** → Ship to production. Assumes `.p` already ran:
  1. Delete any plan files or temporary MD files created during this feature that are no longer needed
  2. Stage changed files, write a fitting commit message, commit, and push the current branch
  3. Switch to `main`, pull latest, merge the feature branch into main (using `--no-ff`), and push main
  4. Switch back to the previous branch and merge main into it so the working branch stays up to date
  5. If already on `main`, skip steps 3-4 and just commit + push main directly.

## Working Style

- **Plans should be ELI10** - Explain what happens in plain language, no code snippets
- Keep things simple and focused
- Show visibility into what's happening (logs, debug info) when things fail

## Quick Start

```bash
npm install
cp .env.example .env.local  # Add your GOOGLE_PLACES_API_KEY
node server.js
# Open http://localhost:3001
```

## Stack

- Frontend: Plain HTML/CSS/JS (no build step)
- Backend: Node.js + Express
- Database: SQLite (better-sqlite3)

## Key Files

- `server.js` - Express server with API routes
- `google-places.js` - Grid-based Google Places scraping
- `db.js` - SQLite database layer
- `index.html` + `app.js` - Frontend UI

## API Endpoints

### Searches
- `GET /api/searches` - List all past searches
- `POST /api/searches` - Start scrape (body: {query, location, gridSize})
- `GET /api/searches/:id` - Get search details
- `DELETE /api/searches/:id` - Delete a search

### Companies
- `GET /api/searches/:id/companies` - Get companies for a search
- `GET /api/companies` - List all companies
- `GET /api/companies/:id` - Get single company (full view)
- `DELETE /api/companies/:id` - Delete a company

### Export
- `GET /api/searches/:id/export` - Export search results as CSV
- `GET /api/companies/export` - Export all companies as CSV

### Usage
- `GET /api/usage` - Get API usage stats

## Google Places API

- Uses Text Search (New) API
- Grid search bypasses 60-result limit (max 60 per query)
- Deduplicates globally by place_id
- Rate limited to prevent cost overruns

## Database Schema

- `searches` - Track each scrape run
- `companies` - Global company records (deduplicated)
- `search_companies` - Many-to-many relationship
- `api_usage` - Track monthly API calls

## Rate Limiting

Default: 20 API requests/month. Configure via `API_MONTHLY_LIMIT` in .env.local.

---

## Waterfall Enrichment API

The waterfall enrichment system provides automated lead enrichment by trying free methods first (web scraping), then falling back to paid APIs (Hunter.io) only when necessary. This reduces costs by ~70%.

### Required Environment Variables

```bash
ANTHROPIC_API_KEY=sk-ant-...   # Required for Claude API (company enrichment and web scraping)
HUNTER_API_KEY=...             # Optional, for Hunter.io fallback in contact discovery
```

### Enrichment Endpoints

#### POST `/api/companies/:id/enrich-full`

Run full waterfall enrichment on a single company. Executes all steps:
1. Extract domain and enrich company (segment, industry, ICO)
2. Validate ICO via ARES (Czech business registry)
3. Discover contacts (web scrape first, Hunter.io fallback)
4. Validate emails via MX lookup
5. Assign outreach templates based on contact roles

**Request Body**: None required (uses company ID from URL)

**Response**:
```json
{
  "company_id": 123,
  "domain": "example.cz",
  "enrichment": {
    "ico": "12345678",
    "segment": "marketing_agency",
    "industry": "Digital Marketing",
    "size": "11-50"
  },
  "ico_validation": {
    "valid": true,
    "company_name": "Example s.r.o.",
    "address": "Praha 1"
  },
  "contacts": [
    {
      "email": "jan@example.cz",
      "firstName": "Jan",
      "lastName": "Novak",
      "title": "CEO",
      "email_valid": true,
      "template_type": "strategic_partnership",
      "source": "web_scrape"
    }
  ],
  "errors": []
}
```

**Example**:
```bash
curl -X POST http://localhost:3003/api/companies/42/enrich-full
```

---

#### POST `/api/companies/enrich-batch`

Batch enrich all unenriched companies (companies with website but no `enrichment_source`).

**Request Body**:
```json
{
  "limit": 10  // Optional, defaults to 10. Max companies to process.
}
```

**Response**:
```json
{
  "total": 50,
  "processed": 10,
  "enriched": 8,
  "contacts_found": 23,
  "errors": [
    {
      "company_id": 15,
      "name": "Broken Corp",
      "error": "Failed to fetch homepage"
    }
  ]
}
```

**Example**:
```bash
curl -X POST http://localhost:3003/api/companies/enrich-batch \
  -H "Content-Type: application/json" \
  -d '{"limit": 25}'
```

---

#### GET `/api/companies/:id/categorize`

Get or refresh company categorization (segment, industry). Uses Claude API to analyze the company website.

**Note**: This endpoint is defined in PRD-WATERFALL-ENRICHMENT.md but not yet implemented. Use `/api/companies/:id/enrich-full` for categorization.

---

#### POST `/api/contacts/:id/validate-email`

Run MX record check on a single contact's email address.

**Request Body**: None required

**Response**:
```json
{
  "contact_id": 456,
  "email": "jan@example.cz",
  "valid": true,
  "reason": "mx_found",
  "mx_host": "mail.example.cz",
  "validated_at": "2026-02-16T12:00:00.000Z"
}
```

**Example**:
```bash
curl -X POST http://localhost:3003/api/contacts/456/validate-email
```

---

#### POST `/api/contacts/validate-batch`

Batch validate all contacts with unchecked emails (where `email_valid` is NULL).

**Request Body**:
```json
{
  "limit": 50  // Optional, defaults to 50. Max contacts to validate.
}
```

**Response**:
```json
{
  "total": 100,
  "processed": 50,
  "valid": 42,
  "invalid": 8,
  "errors": [
    {
      "contact_id": 789,
      "email": "bad@nonexistent.xyz",
      "error": "DNS lookup failed"
    }
  ]
}
```

**Example**:
```bash
curl -X POST http://localhost:3003/api/contacts/validate-batch \
  -H "Content-Type: application/json" \
  -d '{"limit": 100}'
```

---

### Enrichment Module Files

Located in `enrichment/`:

| File | Purpose |
|------|---------|
| `index.js` | Main orchestrator, re-exports all utilities |
| `companyEnricher.js` | Company categorization via Claude API |
| `contactWaterfall.js` | Waterfall contact discovery (scrape -> Hunter) |
| `webScraper.js` | Team/contact page scraping (FREE) |
| `validators.js` | Email MX and phone validation |
| `ares.js` | Czech ICO validation via ARES API |
| `templateRouter.js` | Role to outreach template matching |

---
> Source: [jaro-z/lead-scraper](https://github.com/jaro-z/lead-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
