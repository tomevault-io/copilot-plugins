## contractgenerator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Contract Generator v3** is a modular contract generation system for creating professional legal contracts (specifically for Wmo/Jeugd agreements). The system uses a flow-based engine with conditional logic to guide users through contract creation, clausule selection, and Word document export.

## Architecture

### Three-Layer System

1. **Frontend Layer** - Single-page React application (`standalone_contract_generator_v3_flow.html`)
   - Self-contained HTML file with embedded React 18 + Tailwind CSS
   - Uses CDN-delivered libraries: React, Tailwind, DOMPurify, docx.js, FileSaver
   - No build process required - runs directly in browser

2. **Backend API Layer** - Express.js server (`server.js`)
   - RESTful API for flows and clausules management
   - SQLite database (`flows.db`) for persistent storage
   - Serves static files and provides CRUD operations
   - Port: 3001 (mapped to 8080 in Docker)

3. **Data Layer** - JSON configuration files
   - `flows/*.json` - Flow definitions with steps, questions, and conditional logic
   - `clausules/*.json` - Clausule library organized by category
   - Files are seeded into SQLite database on startup

### Key Components

**Flow Engine**
- Processes multi-step contract generation workflows
- Supports conditional routing based on user answers
- Step types: `parameters` (questions), `clausules` (selection), `edit` (modification), `review` (final)
- Question types: `text`, `textarea`, `select`, `multiselect`, `boolean`, `date`
- Conditional logic via `conditioneel` fields that reference previous answers

**Admin Interfaces**
- `beheer/flow-beheer.html` - Flow configuration management
- `beheer/clausule-beheer.html` - Clausule library management
- Both interfaces communicate with backend API for CRUD operations

**Database Schema**
```sql
flows (
  id, flow_id, naam, beschrijving, versie, auteur,
  laatste_update, flow_data (JSON), created_at, updated_at
)

clausules (
  id, clausule_id, titel, categorie, inhoud (JSON),
  versie, auteur, actief, tags, created_at, updated_at
)
```

## Development Commands

### Local Development (Docker - Recommended)

```bash
# Start application with auto-rebuild
./test-lokaal.sh

# Manual Docker commands
docker-compose up --build -d
docker-compose ps
docker-compose logs -f
docker-compose restart
docker-compose down
```

Access points:
- Main app: http://localhost:8080
- Flow management: http://localhost:8080/beheer/flow-beheer.html
- Clausule management: http://localhost:8080/beheer/clausule-beheer.html
- Health check: http://localhost:8080/api/health

### Backend Development

```bash
# Install dependencies
npm install

# Start server (production)
npm start

# Start with auto-reload (development)
npm run dev
```

### Testing Changes

After modifying flows or clausules JSON files:
1. Changes are automatically reflected via Docker volumes (no rebuild needed)
2. For database reseeding: `docker-compose restart`
3. For full reset: `docker-compose down && docker-compose up --build`

## Important Business Logic

### Flow Processing Rules (based on `20251010 flow regels.md`)

The flow engine implements specific legal contract assembly rules:

1. **Conditional Clausules** - Clausules appear/disappear based on:
   - Inkoopmethode (aanbesteding vs toelating)
   - Uitvoeringsvariant (inspanningsgericht, outputgericht, taakgericht)
   - Multiple opdrachtnemers (affects Deel 2 visibility)

2. **Three-Part Contract Structure**:
   - **Deel 1**: Bepalingen tussen opdrachtgever en alle opdrachtnemers (always shown)
   - **Deel 2**: Individuele bepalingen voor specifieke opdrachtnemer (conditional on meerdere_opdrachtnemers)
   - **Deel 3**: Generieke bepalingen (numbered 3.1 - 3.31, some conditional on variant)

3. **Article Numbering**: Dynamic renumbering based on included/excluded clauses

### API Endpoints

**Flows**
- `GET /api/flows` - List all flows
- `GET /api/flows/:flowId` - Get specific flow
- `POST /api/flows/:flowId` - Create/update flow
- `DELETE /api/flows/:flowId` - Delete flow
- `POST /api/import/flows` - Import flows from JSON files

**Clausules**
- `GET /api/clausules` - List all clausules (with optional filters: `?categorie=X&actief=true`)
- `GET /api/clausules/categories` - Get unique categories
- `GET /api/clausules/:clausuleId` - Get specific clausule
- `POST /api/clausules` - Create new clausule
- `PUT /api/clausules/:clausuleId` - Update clausule
- `DELETE /api/clausules/:clausuleId` - Delete clausule
- `POST /api/import/clausules` - Import clausules from JSON files

## Working with Data Files

### Flow JSON Structure
```json
{
  "flow_id": "unique-id",
  "naam": "Flow Name",
  "beschrijving": "Description",
  "versie": "1.0",
  "stappen": [
    {
      "stap_id": "step1",
      "type": "parameters",  // or "clausules", "edit", "review"
      "groepen": {
        "opdrachtgever": {
          "label": "Opdrachtgever Gegevens",
          "info_button": false,
          "toelichting": "Optioneel: toelichting op groepsniveau"
        }
      },
      "vragen": [
        {
          "vraag_id": "example",
          "titel": "Question Title",
          "type": "text",  // or select, textarea, boolean, date, multiselect, etc.
          "verplicht": false,
          "info_button": true,  // Show/hide info button for this question (default: false)
          "groep": "opdrachtgever", // NEW: group key this question belongs to
          "groep_label": "(optioneel) override van group label for this question"
        }
      ],
      "volgende_stappen": ["step2"]
    }
  ]
}
```

### Info Button Feature (vraag- en groepsniveau)
Per vraag (question) kan je bepalen of de info-knop getoond moet worden via het `info_button` veld:
- `"info_button": true` - Toont de ℹ️ knop waarmee gebruikers toelichting kunnen openen
- `"info_button": false` - Verbergt de info-knop voor deze vraag (standaard)
- De toelichting text wordt opgehaald uit het `toelichting` veld van de vraag
- De frontend rendert het info-icoontje alleen als `info_button` niet gelijk is aan `false`

Op groepsniveau (binnen een parameters-stap) kan je ook een info-knop tonen:
- `stap.groepen[<key>].info_button: true|false`
- `stap.groepen[<key>].toelichting: "..."`
- `stap.groepen[<key>].label: "Weergavetitel"`

De frontend groepeert in deze volgorde:
1. `vraag.groep` (configureerbaar)
2. legacy hardcoded mapping (backwards compatible)
3. prefix-heuristiek voor IDs (tekst vóór eerste underscore)

### Clausule JSON Structure
```json
{
  "CLAUSULE_ID": {
    "artikelnummer": "3.1",
    "titel": "Title",
    "tekst": "Main text with {placeholders}",
    "toelichting": "Explanation",
    "categorie": "Category",
    "verplicht": true/false
  }
}
```

### Placeholder System
Clausule text can contain placeholders like `{gemeente}`, `{opdrachtgever_naam}` which are replaced with user-provided flow answers.

## Code Modification Guidelines

### Frontend Changes
- The main HTML file is standalone - no build step required
- When editing, preserve the embedded React structure
- DOMPurify sanitizes all user input before rendering
- Word export uses docx.js library (browser-compatible, no server-side required)

### Backend Changes
- Server uses async/await pattern for database operations
- All API responses follow `{ error: "..." }` or `{ message: "...", data: ... }` format
- Database operations have 1-2 second delays for async completion (see import endpoints)
- CORS is enabled for all origins (suitable for local development)

### Database Changes
- SQLite database auto-initializes on first run
- Seed data loaded from `flows/` and `clausules/` directories
- To reset database: delete `flows.db` file and restart server
- Use `updated_at` timestamps for tracking modifications

## Docker Configuration

The application uses two Dockerfiles:
- `Dockerfile.backend` - Main production image (Node.js 18 Alpine)
- `Dockerfile` - Legacy static nginx setup (not currently in use)

Volumes mounted in docker-compose for hot-reload:
- `./standalone_contract_generator_v3_flow.html:/app/public/index.html:ro`
- `./clausules:/app/clausules:ro`
- `./flows:/app/flows:ro`
- `./beheer:/app/public/beheer:ro`

## Git Branch Structure

- Main branch: `main`
- Create PRs against `main`
- Version history visible in commits (see `95485f0` for initial code)

## Known Issues & Considerations

1. **Async Import Operations**: Import endpoints (`/api/import/*`) use setTimeout for async database operations. This is not ideal for production but works for current use case.

2. **Static File Paths**: The main HTML file references itself from multiple directories. When moving files, update relative paths in both frontend and docker-compose volumes.

3. **JSON Data Seeding**: Only flows are seeded on startup (clausules seeding is commented out in server.js:87). Use the import API endpoint to load clausules.

4. **Word Export**: Uses browser-based docx.js library. Export is client-side only, no server processing required.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/i-Sociaal-Lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
