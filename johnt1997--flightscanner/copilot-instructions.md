## flightscanner

> Günstige Wochenend-Flüge ab Wien/Bratislava/Budapest finden. Skyscanner-Scraper mit React-Frontend.

# Flight Scout – CLAUDE.md

## Überblick
Günstige Wochenend-Flüge ab Wien/Bratislava/Budapest finden. Skyscanner-Scraper mit React-Frontend.
Sprache: **Deutsch** (UI, Kommentare, Variablennamen teilweise englisch).

## Tech Stack
- **Backend:** Python 3.13, FastAPI, SQLite (WAL mode), bcrypt, fpdf2
- **Frontend:** React 18, Vite, Leaflet/react-leaflet (kein CSS-Framework, inline styles)
- **Deploy:** Docker multi-stage → Railway (Volume `/data` für DB)

## Projektstruktur
```
Dockerfile                          # Multi-stage: Node build → Python serve
flight-scout/
├── backend/
│   ├── main.py                     # FastAPI app, alle Endpoints, Job-Management
│   ├── scraper.py                  # SkyscannerAPI Klasse, PDF-Report, Proxy-Logik
│   ├── database.py                 # SQLite Schema, Auth, CRUD für alle Tabellen
│   ├── cities.py                   # CITY_DATABASE dict (75 Städte mit entity_id, sky_code, lat/lon)
│   ├── alerts.py                   # Telegram Deal-Alerts (täglicher Scheduler 7:00 UTC)
│   ├── requirements.txt            # fastapi, uvicorn, requests, fpdf2, pydantic, bcrypt
│   ├── proxies_europe.txt          # Residential Proxies (host:port:user:pass)
│   ├── verify_city_ids.py          # Hilfsskript: Skyscanner entity_ids verifizieren
│   └── flight_scout.db             # SQLite DB (gitignored)
├── frontend/
│   ├── src/
│   │   ├── main.jsx                # React Entry Point
│   │   ├── FlightScout.jsx         # Haupt-Komponente (~2400 Zeilen, alles-in-einem)
│   │   ├── CalendarView.jsx        # Kalender-Heatmap (Monatsansicht)
│   │   └── HeatmapView.jsx         # Leaflet-Karte mit Routen
│   ├── vite.config.js              # Dev: Port 3000, Proxy /api → localhost:8000
│   └── dist/                       # Production Build (wird ins Docker-Image kopiert)
```

## Datenbank (SQLite)
8 Tabellen: `users`, `saved_deals`, `deal_alerts`, `price_alerts`, `search_cache` (3h TTL), `saved_searches` (max 5/User), `search_log`, `public_deals`
- DB-Pfad: `/data/flight_scout.db` (Railway) oder `./flight_scout.db` (lokal)
- Auth: HMAC-Token (kein JWT), bcrypt-Passwörter

## API Endpoints (wichtigste)
| Method | Endpoint | Auth | Beschreibung |
|--------|----------|------|-------------|
| POST | `/search` | Ja | Flugsuche starten (rate limited: 10/30min) |
| GET | `/status/{job_id}` | Ja | Job-Status mit Progress + Partial Results (nur eigene Jobs) |
| POST | `/stop/{job_id}` | Ja | Suche abbrechen (nur eigene Jobs) |
| POST | `/deals/save` | Ja | Deal speichern (Duplikat-Check: city+dates) |
| POST | `/deal-alerts` | Ja | Telegram-Alert erstellen (max 2/User) |
| POST | `/searches/save` | Ja | Suche speichern (max 5/User) |
| POST | `/calendar` | Nein | Kalender-Preisdaten für einen Monat |
| GET | `/top-deals` | Nein | Public Deals für Landing Page (wöchentlicher Crawl) |
| GET | `/admin/*` | Admin | User-Liste, Suchverlauf, Test-Alerts |

## Architektur-Entscheidungen
- **Alles-in-einem Frontend:** FlightScout.jsx enthält die gesamte App (Tabs, Auth, Suche, Archiv, Admin). Kein Router, keine separaten Komponenten außer Karte und Kalender.
- **Landing Page:** Nicht-eingeloggte User sehen Top Deals aus wöchentlichem Crawl (`/top-deals`). Eingeloggte User haben einen "Top Deals" Tab.
- **Job-System:** Suchen laufen als Background Tasks. Frontend pollt `/status/{job_id}` alle 1.5s. Partial Results werden live angezeigt.
- **Proxy-Rotation:** Residential Proxies aus `proxies_europe.txt`. 407 → sofort neuer Proxy. 403 → 30s/60s Pause + neuer Proxy.
- **API-Strategie:** Everywhere-Suche → Country-Suche → City-Detail. Bei 403 Fallback auf Country-Level Preise.
- **Caching:** Everywhere-Ergebnisse 3h in SQLite. Cache-Key = airports+dates+duration.
- **Parallelisierung:** ThreadPoolExecutor (max 3 Worker) für Multi-Trip Suchen.
- **Flexible Duration:** Toggle "+/- 1 Nacht" expandiert gewählte Duration zu 3 Durations (z.B. 2→[1,2,3]). Frontend-seitig, Backend bekommt einfach mehr durations.

## Wichtige Patterns
- `get_user_id(request)` extrahiert User-ID aus Bearer Token (alle Auth-Endpoints)
- `ADMIN_USERS = {"john1997"}` – Admins umgehen Rate Limits und Suchbeschränkungen
- `_deal_to_dict(deal)` konvertiert FlightDeal Dataclass → Dict für JSON Response
- Neue Städte: In `cities.py` eintragen (entity_id via Skyscanner Network Requests finden)
- Neue Airports: In `main.py` AIRPORTS dict + `alerts.py` AIRPORTS dict (Duplikation!)

## Env-Variablen
| Variable | Beschreibung |
|----------|-------------|
| `PROXY_URL` | Fallback-Proxies (`;` getrennt) |
| `TELEGRAM_BOT_TOKEN` | Telegram Bot für Deal-Alerts |
| `FLIGHT_SCOUT_SECRET` | HMAC Secret für Auth-Tokens (**Pflicht** – App startet ohne nicht) |
| `PORT` | Server-Port (default 8000) |

## Befehle
```bash
# Backend lokal
cd flight-scout/backend && pip install -r requirements.txt && python main.py

# Frontend lokal
cd flight-scout/frontend && npm install && npm run dev

# Frontend Build
cd flight-scout/frontend && npx vite build

# Docker
docker build -t flight-scout . && docker run -p 8000:8000 flight-scout
```

## Suchbeschränkungen (Non-Admin)
- Max 3 Reisedauern pro Suche
- Max 3 Städte bei City-Suche
- Max 3 Monate Suchzeitraum (93 Tage)
- Rate Limit: 10 Suchen pro 30 Minuten
- Login: max 10 Versuche pro IP / 15 Min, Passwort min. 8 Zeichen, Tokens laufen nach 30 Tagen ab

## UI-Details
- Zeitraum-Chips statt roher Datumsfelder: "Nächste 4 Wochen" (Default) / 3 Monats-Chips / "Eigene Daten" (`windowMode` State; Chips berechnen start/end aus Modus + Wochentag)
- Datumsfelder ("Frühester/Spätester Hinflug") nur bei "Eigene Daten" sichtbar; Smart Sync (Wochentag + Enddatum Monatsende) läuft dort im onChange, nicht als Effect
- Konsequenz-Zeile unter dem Zeitraum: "→ prüft n× Fr als Hinflugtag …" (`countOccurrences` Helper, auch von Summary + Button-Disable genutzt)
- Such-Summary über Button: zeigt Trip-Anzahl, Zeitraum, Durations, Airports, Suchabfragen
- Länderflaggen via flagcdn.com (COUNTRY_CC Mapping in Frontend, AIRPORTS.cc für Airports)
- Everywhere-Suche liefert Ländernamen im `city`-Feld, `country`-Feld ist leer → Frontend Fallback: `COUNTRY_CC[deal.country] || COUNTRY_CC[deal.city]`

## Bekannte Eigenheiten
- AIRPORTS dict ist in `main.py` UND `alerts.py` dupliziert
- Flight-URL wird an 3 Stellen gebaut (scraper.py, alerts.py, main.py)
- Timestamps in DB sind UTC, Frontend konvertiert mit `+ 'Z'` Suffix zu Lokalzeit
- FlightScout.jsx ist sehr groß (~2500+ Zeilen) – Refactoring in Komponenten steht aus
- `price_alerts` Tabelle existiert, wird aber aktuell nicht aktiv genutzt (deal_alerts stattdessen)
- Easter-Periode (28.3.–6.4.2026) hat Sonderbehandlung beim Frühflug-Filter
- `public_deals` Tabelle wird wöchentlich (Montag 7:00 UTC) von alerts.py befüllt, max 15 Deals pro Airport unter 100€

---
> Source: [johnt1997/flightscanner](https://github.com/johnt1997/flightscanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
