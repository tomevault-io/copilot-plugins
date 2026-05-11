## dessertable

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**DessertAble** is a Flask web application that finds and recommends dessert spots using Google Places API and DeepSeek AI. It features user authentication, favorites management, and an AI-powered ranking system that balances rating quality, distance, and newness.

## Core Commands

### Development
```bash
# Run the application locally
python run.py

# Or use the platform-specific scripts
# Windows: START_APP.bat
# Mac/Linux: ./START_APP.sh

# Run tests
pytest tests/

# Run specific test file
pytest tests/test_scoring_algorithm.py

# Run with verbose output
pytest -v
```

### Environment Setup
```bash
# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Mac/Linux)
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Production Deployment
```bash
# Run with Gunicorn (production)
gunicorn --workers 4 run:app

# Generate production SECRET_KEY
python -c "import secrets; print(secrets.token_hex(32))"
```

## Architecture Overview

### Application Factory Pattern
- `app/__init__.py` - Flask application factory using `create_app()`
- Configuration loaded from `config.py` based on `FLASK_ENV` environment variable
- Blueprint-based routing with single main blueprint (`main_bp`)

### Service Layer Architecture
The application uses a **service-oriented architecture** separating concerns:

1. **PlacesAPI** (`app/services/places_api.py`)
   - Wrapper around Google Maps Python client
   - Handles geocoding, nearby search, and place details
   - Built-in caching system (60-second TTL)
   - Uses Haversine formula for distance calculations

2. **DeepSeekService** (`app/services/deepseek_service.py`)
   - Generates 10-15 word AI descriptions from reviews
   - OpenAI-compatible API client pointing to DeepSeek
   - Gracefully degrades if API key not available

3. **RestaurantService** (`app/services/restaurant_service.py`)
   - Core business logic for restaurant search and ranking
   - **Progressive filter relaxation**: automatically widens search criteria if insufficient results
   - Composite scoring algorithm (see below)

### Database Layer
- `Database` class (`app/models/database.py`) supports **both SQLite and PostgreSQL**
- Auto-detects via `DATABASE_URL` env var: PostgreSQL in production, SQLite in development
- Database location (SQLite): `data/restaurants.db` (created automatically)
- Four main tables:
  - `searches` - Search history with parameters
  - `restaurants` - Cached restaurant data linked to searches
  - `users` - User accounts with password hashing
  - `favorites` - User-restaurant many-to-many with notes
- Key DB helper methods for cross-DB compatibility:
  - `get_connection()` - returns connection with proper row factory
  - `placeholder()` - returns `?` (SQLite) or `%s` (PostgreSQL)
  - `execute_insert()` - INSERT returning last row ID for either DB
  - `date_interval(days)` - DB-specific date arithmetic

### Authentication
- Flask-Login integration in `app/__init__.py`
- User model implements `UserMixin` interface
- Password hashing with werkzeug security utilities
- Session-based authentication with 7-day persistence
- **Invite-code-gated registration**: `INVITE_CODE` env var required to create accounts

### Admin Panel
- Separate session-based admin auth at `/admin/login` (not Flask-Login)
- Protected by `ADMIN_PASSWORD` env var; session key `is_admin`
- Dashboard at `/admin` (also `/admin/dashboard`) powered by `db.get_admin_analytics()` and `db.get_all_users(limit=50)`
- **Stat cards**: Total Users, Total Searches, Total Favorites, Avg Results — each with a +N last-7-days badge
- **Top Locations** (last 30 days) and **Popular Cuisines** panels with hover-slide rows
- **Recent Activity** table: last 10 searches with location, date, result count
- **User Management** table: all registered users with join date and favorite count
- **Search Activity** and **User Registrations** trend charts: bar-style rows for each of the last 30 days, normalized to max
- **Database Information** strip: cached restaurant count, total searches, total favorites
- **Export Data** button: downloads a CSV of the key metrics via `Blob` URL (client-side, no server call)
- Templates: `admin_login.html`, `admin_dashboard.html`

## Critical Algorithm: Composite Scoring

The **composite scoring algorithm** in `RestaurantService.calculate_composite_score()` is the heart of the ranking system:

**Weight Distribution:**
- **60%** - User's sort preference (rating OR distance)
- **15%** - Cuisine match bonus
- **20%** - Newness score (logarithmic decay based on review count)
- **5%** - Proximity bonus (always applied regardless of sort preference)

**Score Calculation:**
```
Total Score = Base Score (60) + Cuisine Bonus (0-15) + Newness (0-20) + Proximity (0-5)
Maximum Possible: 100 points
```

**Newness Formula:**
```
newness = max(0, 20 - (log10(review_count) * 8))
```
This creates logarithmic decay:
- 25 reviews ≈ 9 pts (newer establishments)
- 100 reviews ≈ 4 pts
- 500+ reviews ≈ 0 pts (established venues)

**Progressive Filter Relaxation:**
The search applies filters in multiple passes to guarantee at least 5 results:
1. Try: 25+ reviews, 5-mile radius
2. Try: 15+ reviews, 7-mile radius
3. Try: 10+ reviews, 10-mile radius
4. Try: 5+ reviews, 15-mile radius
5. Final: 0+ reviews, 20-mile radius

This ensures users always get results while preferring well-reviewed nearby options.

## Data Flow: Search Request

1. User submits address via `/search` POST route
2. `PlacesAPI.geocode_address()` converts address → (lat, lng)
3. `PlacesAPI.search_restaurants()` fetches nearby dessert places with pagination
4. `RestaurantService.apply_filters()` applies progressive filtering
5. `PlacesAPI.get_place_details()` enriches top 20 results with reviews, hours, etc.
6. `DeepSeekService.generate_dessert_description()` creates AI summaries from reviews
7. `RestaurantService.calculate_composite_score()` scores each restaurant
8. Results sorted by composite score descending
9. Database saves search + top 3 results
10. Route passes top 3 (`restaurants[:3]`) to template; results template renders up to 5 in a 3-per-page carousel

## Key Design Decisions

### Why Dual-Database (SQLite + PostgreSQL)?
- SQLite for zero-config local development; PostgreSQL for production (Render)
- `Database` class auto-detects via `DATABASE_URL` env var — no code changes needed
- `render.yaml` provisions a free PostgreSQL instance and injects `DATABASE_URL` automatically
- See `infrastructure/ARCHITECTURE.md` for deployment architecture details

### Why Progressive Filtering?
- Guarantees non-empty results (critical UX requirement)
- Prefers quality (high reviews, close distance) but adapts to reality
- Avoids the "no results found" problem in sparse areas

### Why Composite Scoring Instead of Simple Rating Sort?
- Pure rating sort favors established restaurants with hundreds of reviews
- Pure distance sort ignores quality entirely
- Composite approach surfaces hidden gems (new places with high ratings)
- Newness component gives newer businesses a chance to compete

### API Cost Optimization
- Caching layer in PlacesAPI (60s TTL)
- Limits detailed fetches to top 20 candidates only
- DeepSeek gracefully degrades without breaking functionality

### Visual Design — Washi Paper / Paper Collage Aesthetic
- **Color palette**: Warm linen variables — `--ink` (#1a1815), `--paper` (#e9e4d9), `--paper-warm` (#f0ebe2), `--paper-mid` (#d4ccc0), `--ink-block` (#1e1b17), `--fog` (#cac4ba). No blue accents except `--slate` for focus rings.
- **Body background**: `washi_bg.png` (1536×1024, served from `/static/`) tiled at 768×512 with `background-blend-mode: multiply` over the warm linen base — gives a subtle paper-fiber texture across all pages.
- **Typography**: Shippori Mincho (wght 400/500/700) as the primary serif; JetBrains Mono for meta/mono labels.
- **Home page**: Full-viewport collage of 5 CSS-drawn paper blocks (`.collage-tan-tl`, `.collage-dark-tr`, `.collage-dark-bl`, `.collage-tan-br`, `.collage-dark-shadow`) with diagonal multi-stop gradients, SVG `feTurbulence` fiber overlays on `::before`, and subtle rotations (−1.3° to +1.5°). Center `.home-card` floats above with a drop shadow.
- **Home card (single-screen)**: "Go" heading + address input (always visible) + circular 54px ▶ play/submit button. No two-screen toggle — input is immediately visible on load.
- **Results page frame**: `body:has(.results-frame)` hides the global navbar/footer and sets a washi-textured gray background (#c8c3bc). The `.results-frame` container is semi-transparent (`rgba(233,228,217,0.82)`) with a 1px `--fog` border so the washi shows through faintly.
- **Results frame nav**: Brand link (left) · timestamp (center-right) · auth links (far right) · 1px `--fog` separator.
- **Results cards**: 3-column grid with 1.25rem gap; each card fully bordered in `--fog`; emoji header row (🍵/🏮/🪭 cycling) with name + cuisine·rating; `card-rule` dividers between hours / description / distance.
- **Pagination bar**: Dark `--ink-block` bar spanning the full frame width. Left group: `< Previous | Page N of M | Next >` (thin vertical separators). Right: `Next >`.

### UI / UX Design Decisions
- **Full-card click**: result cards use an overlay `<a>` link to the restaurant website (no nested links)
- **Toast notifications**: `alert()` replaced with toast messages across all pages for non-blocking feedback; defined globally in `base.html` as `showToast(message, duration)` with slide-in / fade-out CSS animations
- **Search loading state**: button disables and an animated dot-ellipsis loading message appears during API calls to prevent double-submit
- **Search input clear button**: an `×` clear button (`.input-clear-btn`) appears inside the address field when text is present
- **Results carousel**: results page shows up to 9 results, 3 per page, with Previous/Next pagination in the dark bar; page counter shows "Page N of M"
- **Hamburger nav**: mobile (≤480px) replaces desktop nav with a slide-out drawer (`mobile-menu`), a dark overlay (`mobile-nav-overlay`), and closes on link click / overlay tap / Escape key / window resize above 480px
- **Mobile-first CSS**: 44×44px minimum touch targets, `safe-area-inset` support for notched phones; breakpoints at 375px (iPhone SE), 480px (phone), 481–768px (tablet, 2-col grid), 1024px (stack cards to 1-col)
- **Touch device block**: `@media (hover: none) and (pointer: coarse)` removes hover effects and adds active-state scale feedback for touch devices
- **CSS animations**: `fadeIn` (page entrance), `toastSlideIn` / `toastFadeOut`, ink-drop click burst via `retro.js`
- **AI descriptions**: DeepSeek generates ≤10-word personal sentences (e.g., "The croissants here changed my mornings.")

## Environment Variables

Required in `.env` file:

```bash
# Required for core functionality
GOOGLE_PLACES_API_KEY=your_google_api_key

# Required for production
SECRET_KEY=generate_with_secrets_token_hex_32
FLASK_ENV=production

# Optional - AI descriptions
DEEPSEEK_API_KEY=your_deepseek_api_key

# Optional - controls who can register
INVITE_CODE=your_invite_code

# Optional - admin dashboard access
ADMIN_PASSWORD=your_admin_password

# Set automatically by Render (PostgreSQL); omit for local SQLite
DATABASE_URL=postgresql://...
```

**Getting API Keys:**
- Google Places: https://console.cloud.google.com/ (enable Places API, create credentials)
- DeepSeek: https://platform.deepseek.com/

## Testing Philosophy

- **33 unit tests** for composite scoring algorithm (`tests/test_scoring_algorithm.py`)
- Tests use MockPlacesAPI to avoid real API calls
- Fixtures provide reusable restaurant objects
- `pytest.approx()` for floating-point comparisons (±10% tolerance)
- **Known quirk**: `distance=0` evaluated as falsy in Python; documented in tests

## Common Modifications

### Adding a New Filter
1. Add parameter to `RestaurantService.find_restaurants()`
2. Update `apply_filters()` method with new filter logic
3. Update route parameter extraction in `routes.py`
4. Update HTML form in templates

### Changing Scoring Weights
Modify constants in `RestaurantService.calculate_composite_score()`:
```python
# Current: 60% base, 15% cuisine, 20% newness, 5% proximity
score += (restaurant.rating / 5.0) * 60  # Adjust 60
# ... etc
```
**Important**: Update tests in `test_scoring_algorithm.py` to match new weights.

### Adding New Database Fields
1. Add column in **both** `Database._init_database_sqlite()` and `Database._init_database_postgres()` CREATE TABLE statements
2. Update corresponding model class (`User`, `Restaurant`)
3. Handle migration (SQLite doesn't support ALTER TABLE well; recommend recreating DB in dev)
4. Update insert/select queries in database methods — use `self.placeholder()` for parameterized values

## File Structure

```
app/
  __init__.py           # Application factory, Flask-Login setup
  routes.py             # All routes (main blueprint + admin routes); _is_safe_redirect helper
  models/
    database.py         # Dual-database manager (SQLite/PostgreSQL)
    restaurant.py       # Restaurant data model; get_google_maps_url uses quote_plus
    user.py             # User model with authentication
  services/
    places_api.py       # Google Places API wrapper
    deepseek_service.py # DeepSeek AI integration (graceful degradation; structured logging)
    restaurant_service.py # Core business logic & scoring
  utils/
    filters.py          # Jinja2 template filters
  static/
    css/style.css       # All styles — washi palette, collage blocks, results frame, cards
    js/retro.js         # Ink-drop click burst animation
    washi_bg.png        # Washi paper texture tile (1536×1024, served at /static/)
    sprites/            # Legacy sprite images (not used on results page)
  templates/
    base.html             # Base layout: navbar, toast JS, hamburger menu JS
    index.html            # Single-screen search: "Go" heading + input + circular ▶ button
    results.html          # Results frame: frame nav, emoji cards (🍵/🏮/🪭), dark pagination bar
    favorites.html        # Favorites grid with inline note editing (debounced)
    history.html          # Past searches list with links to view_search
    login.html            # Login form
    register.html         # Invite-code-gated registration form
    admin_dashboard.html  # Admin analytics dashboard (stats, trends, users, export)
    admin_login.html      # Admin session login

config.py              # Config classes (Dev, Prod, Test); warns if SECRET_KEY unset
run.py                 # Application entry point
render.yaml            # Render deployment config (web service + PostgreSQL DB)
requirements.txt       # Python dependencies
tests/                 # Pytest test suite
data/                  # SQLite database location (auto-created, dev only)
design/                # Design mockups (main_page.png, results.png, washi_bg.png source)
infrastructure/
  ARCHITECTURE.md      # Deployment architecture documentation
```

## Security Considerations

- Passwords hashed with werkzeug security (PBKDF2)
- Session cookies: HttpOnly + SameSite=Lax
- SECRET_KEY required for session security; `config.py` emits a `warnings.warn` at startup if unset
- API keys loaded from environment variables (never committed)
- SQL injection prevented via parameterized queries (`self.placeholder()` helper)
- **Open redirect prevention**: `_is_safe_redirect()` in `routes.py` validates the `next=` parameter using `urlparse` — only same-host relative paths are followed after login
- **Timing-safe admin auth**: admin password comparison uses `secrets.compare_digest` to prevent timing attacks
- **Error message hygiene**: flash messages and JSON error responses never surface internal exception details to the client; raw errors go to the logger only
- **URL encoding**: `restaurant.get_google_maps_url()` uses `urllib.parse.quote_plus` to safely encode restaurant names in query strings

## Deployment

**Render (primary, configured via `render.yaml`):**
- Provisions a free PostgreSQL database (`dessertable-db`) and injects `DATABASE_URL`
- Start command: `gunicorn run:app --bind 0.0.0.0:$PORT --workers 2 --threads 2 --timeout 60`
- Required env vars to set manually: `GOOGLE_PLACES_API_KEY`, `DEEPSEEK_API_KEY`, `INVITE_CODE`, `ADMIN_PASSWORD`
- `SECRET_KEY` is auto-generated by Render

See `DEPLOYMENT.md` for additional platform guides (Railway, Fly.io, PythonAnywhere, AWS).

**Key production changes:**
- Set `FLASK_ENV=production`
- Generate secure `SECRET_KEY`
- Use Gunicorn with multiple workers
- PostgreSQL auto-provided on Render via `render.yaml`
- Enable HTTPS (automatic on most platforms)

---
> Source: [Colsai/Dessertable](https://github.com/Colsai/Dessertable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
