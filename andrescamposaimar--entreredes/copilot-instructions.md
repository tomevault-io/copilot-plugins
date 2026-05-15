## entreredes

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Entre Redes is a Flutter mobile app for a football/soccer league (Liga Escolar at Colegio Marianista). It fetches data from a WordPress REST API backend and provides match results, standings, player stats, and team information.

## Common Commands

```bash
# Run the app
flutter run

# Run on a specific device
flutter run -d <device_id>

# List available devices
flutter devices

# Build for release
flutter build apk          # Android
flutter build ios          # iOS

# Run tests
flutter test

# Run a single test file
flutter test test/widget_test.dart

# Lint and analyze code
flutter analyze

# Get dependencies
flutter pub get

# Upgrade dependencies
flutter pub upgrade

# Generate app icons (after changing assets/images/app_logo.png)
flutter pub run flutter_launcher_icons:main
```

## Architecture

The app uses a straightforward MVC-like architecture with three layers:

**UI Layer** → **Service Layer** → **Data Layer (WordPress API + SharedPreferences)**

### Navigation

`main.dart` bootstraps the app: runs weekly cache cleanup on startup, shows a splash screen, then loads `MainNavigation` — a bottom navigation with 5 tabs: Matches, Standings, Teams, Players, More.

### Service Layer (`lib/services/`)

- **`api_service.dart`** — All HTTP calls to the WordPress REST API. Every endpoint is a static method returning parsed JSON. Base URL: `https://entreredespadres.com.ar/wp-json/entre-redes/v1`
- **`cache_service.dart`** — SharedPreferences-based caching with 7-day TTL. Includes a weekly auto-cleanup window (Saturdays after 21:00). Cache keys are dynamic (e.g., per season).
- **`partidos_cache.dart`** — Singleton combining API + local cache for match data.
- **`remote_data_service.dart`** — Fetches remote JSON configs (ads, player waiting lists, live match results from a Google Apps Script).

### Screens (`lib/screens/`)

12 screens, all StatefulWidgets managing their own state via `setState()`. Key screens:
- `matches_screen.dart` — Played and upcoming matches with filters (date, zone, team); infinite scroll pagination.
- `match_detail_screen.dart` — Match details, lineups, goalscorers.
- `standings_screen.dart` — League tables filtered by season/zone.
- `players_screen.dart` / `player_detail_screen.dart` — Player list with search, player stats and match history.
- `teams_screen.dart` / `team_detail_screen.dart` — Team list and roster/match history.
- `listas_screen.dart` — Waiting/reserve player lists (fetched from remote JSON).
- `scorers_screen.dart` / `imbatibles_screen.dart` — Top scorers and goalkeeper clean-sheet stats.
- `more_screen.dart` — Settings, regulations link, manual cache clear.

### Theme (`lib/theme.dart`)

Centralized colors and MaterialApp theme. Primary blue: `0xFF005BBB`, accent cyan: `0xFF00B7CC`. Uses Roboto font (bundled in `assets/fonts/`).

## API Endpoints

All served from `https://entreredespadres.com.ar/wp-json/entre-redes/v1`:

| Path | Key Params |
|------|-----------|
| `/partidos` | `fecha`, `liga`, `temporada`, `equipo`, `page`, `per_page` |
| `/partidos-programados` | `page`, `per_page` |
| `/ligas` | `temporada` |
| `/temporadas` | — |
| `/zonas` | `liga` |
| `/equipos` | `liga`, `temporada`, `page`, `per_page` |
| `/tablas` | `temporada`, `zona`, `search`, `page`, `per_page` |
| `/jugadores` | `temporada`, `liga`, `zona`, `equipo_id`, `search`, `page`, `per_page` |
| `/jugadores/{id}` | — |
| `/partidos-jugador` | `jugador`, `page`, `per_page` |
| `/goleadores` | `partido_id` |
| `/tabla-goleadores` | `id_temporada`, `id_liga`, `page`, `per_page` |
| `/partidos-equipo` | `equipo` or `equipo_id` |
| `/tabla-imbatibles` | `temporada`, `page`, `per_page` |

Responses use `x-wp-total` header for total item count (pagination).

## Key Patterns

- **Pagination / infinite scroll**: Screens use a `ScrollController` with a listener; fetch next page when near bottom. Track `_currentPage`, `_hasMore`, `_isLoadingMore`.
- **Caching pattern**: Check cache first → if expired or missing, call API → save to cache. Cache TTL is 7 days; key format is `{entity}_{id_season}` for season-specific data.
- **StatefulWidget state management**: All state is local to each screen. No global state management library.
- **Ad banners**: `RemoteDataService.fetchAds()` returns a list of `AdItem`; screens cycle through them on a timer.

---
> Source: [andrescamposaimar/entreredes](https://github.com/andrescamposaimar/entreredes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
