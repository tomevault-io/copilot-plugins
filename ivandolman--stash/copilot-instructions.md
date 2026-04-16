## stash

> Stash is een Python-applicatie voor een Raspberry Pi met 5-inch touchscreen en camera.

# Stash — CLAUDE.md

## Project
Stash is een Python-applicatie voor een Raspberry Pi met 5-inch touchscreen en camera.
Het beheert producten op basis van houdbaarheidsdatum, herkent producten via de camera,
stelt recepten voor en genereert boodschappenlijstjes.

Zie `PLAN.md` voor het volledige architectuurplan en de fasering.

## Architectuur
- **Clean Architecture**: `core` heeft geen framework-afhankelijkheden. `infrastructure` en `ui` hangen af van `core`, nooit andersom.
- **Dependency injection**: services ontvangen hun afhankelijkheden via de constructor. Geen globals buiten `config/settings.py`.
- **SOLID**: elke service heeft één verantwoordelijkheid. Nieuwe strategieën (herkenning, AI) worden toegevoegd via nieuwe klassen die een interface implementeren, niet door bestaande code aan te passen.

## Laagstructuur
```
core/           → domeinmodellen, interfaces, services (puur Python)
infrastructure/ → database, camera, AI, externe APIs
ui/             → Kivy/KivyMD schermen, widgets, viewmodels
api/            → FastAPI routes en schemas (fase 6+)
config/         → settings.py via pydantic-settings
deploy/         → systemd service voor RPi
```

## Deployment
- **Kivy UI** draait **native** op de Raspberry Pi — niet in Docker (hardware-toegang vereist).
- **FastAPI API** draait in **Docker** via `docker-compose.yml`.
- De UI start automatisch via `deploy/stash.service` (systemd).
- UI en API delen dezelfde `stash.db` via een Docker volume mount.

## README
- Pas `README.md` aan bij **elke wijziging** aan de codebase: nieuwe bestanden, gewijzigde werking, nieuwe dependencies, statuswijzigingen in de fasering.
- De README is de primaire documentatie voor iemand die het project voor het eerst oppakt.

## Conventies
- Taal: **Nederlands** in commentaar, UI-teksten en commit messages. Engelse termen voor technische concepten zijn prima.
- Type annotations altijd aanwezig op functiesignaturen.
- Pydantic v2 voor alle datamodellen en validatie.
- Repository-pattern voor alle database-toegang — nooit directe SQLAlchemy queries in services of UI.
- `ExpiryService` berekent altijd vervalstatus; status niet zelf berekenen in UI-code.

## Testen
- Draai na **elke codewijziging** de testsuite: `pytest`
- Lever geen code op waarbij tests falen.
- Tests staan in `tests/` met dezelfde mappenstructuur als de broncode.
- Gebruik een in-memory SQLite database (`sqlite:///:memory:`) voor repository-tests.
- Mock externe services (Claude API, Google Vision, Open Food Facts) met `pytest-mock`.
- UI-code (Kivy) wordt niet getest; test uitsluitend `core/` en `infrastructure/`.

## Fasering
| Fase | Status |
|---|---|
| 1 — Basis (CRUD + UI + DB) | Geïmplementeerd |
| 2 — Camera (barcode + Open Food Facts) | Gepland |
| 3 — OCR (THT-datum via EasyOCR) | Gepland |
| 4 — Notificaties (APScheduler) | Gepland |
| 5 — Recepten (Claude API) | Gepland |
| 6 — REST API (FastAPI) | Gepland |
| 7 — Visuele herkenning (Google Vision / YOLO) | Gepland |

## Belangrijke bestanden
- `PLAN.md` — volledig architectuurplan inclusief Docker-strategie
- `requirements.txt` — volledige installatie voor native RPi
- `requirements-api.txt` — alleen API-dependencies voor Docker
- `config/settings.py` — alle configureerbare waarden
- `core/models/product.py` — centrale domeinmodellen (`Product`, `ExpiryStatus`, `ExpirySource`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivandolman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
