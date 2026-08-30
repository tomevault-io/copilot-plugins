## ml-movie-reccomendation-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Run locally:**
```bash
source venv/bin/activate
export TMDB_API_KEY="your_key"  # optional, enables poster images
python app.py                   # runs on port 5001
```

**Run with gunicorn (production-style):**
```bash
gunicorn app:app
```

**Install dependencies:**
```bash
pip install -r requirements.txt
```

**CLI recommendation engine (no web server):**
```bash
python recommendation_engine.py
```

## Architecture

The app is a single-file Flask application (`app.py`) with all ML logic inlined at module load time:

1. **Startup** — `movies.csv` is read into a pandas DataFrame, TF-IDF vectors are computed from combined text features (`keywords`, `genres`, `tagline`, `cast`, `director`), and a full cosine similarity matrix (`SIMILARITY`) is precomputed. This happens once at import time, so the app is stateless per-request.

2. **Recommendation logic** — `get_recommendations(title, limit=12)` looks up the movie index, sorts all movies by cosine similarity score, and returns enriched metadata dicts including a poster URL.

3. **Poster fetching** — `_fetch_poster(title)` calls the TMDB API and caches results in-memory (`POSTER_CACHE` dict). If `TMDB_API_KEY` is not set, posters are silently skipped.

4. **Title resolution** — `_resolve_title(query)` tries an exact case-insensitive match first, then falls back to `difflib.get_close_matches` with `cutoff=0.6` for typo tolerance.

5. **Routes** — `GET /` renders the search page; `POST /recommend` processes the form and re-renders the same `index.html` template with results.

6. **Frontend** — `static/app.js` and `static/styles.css` with a Netflix-inspired UI. The template (`templates/index.html`) receives `recommendations` as a list of dicts and renders a poster grid with a modal for details.

## Environment Variables

| Variable | Default | Purpose |
|---|---|---|
| `TMDB_API_KEY` | `""` | TMDB API key for poster images |
| `APP_TITLE` | `"CineScope"` | App name shown in UI |
| `APP_TAGLINE` | `"Find your next favorite movie"` | Subtitle shown in UI |
| `PORT` | `5000` | Port for production server |

## Key Files

- `app.py` — Flask app with all ML pipeline and routes
- `recommendation_engine.py` — Original standalone CLI prototype (not used by the web app)
- `movies.csv` — Dataset with ~4800 movies and 24 columns
- `runtime.txt` / `Procfile` — Render.com deployment config (Python 3.11.8, gunicorn)

---
> Source: [Jdtorres59/ML-Movie-reccomendation-project](https://github.com/Jdtorres59/ML-Movie-reccomendation-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
