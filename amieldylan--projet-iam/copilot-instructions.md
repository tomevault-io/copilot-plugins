## projet-iam

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IAM (Interactions A Medicaments) is a Flask web application for visualizing drug interactions. Users enter two medication names and the system queries a MySQL database to display potential interactions between them.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Create .env file from example
cp .env.example .env
# Edit .env with your database credentials

# Run development server
python application.py
# or
flask run

# Run tests
pytest

# Run tests with coverage
pytest --cov=app

# Build Sphinx documentation
make html
# Documentation output: build/html/index.html
```

## Architecture

### Project Structure

```
projet-iam/
├── app/                      # Application package
│   ├── __init__.py          # Application factory (create_app)
│   ├── config.py            # Environment-based configuration
│   ├── services/            # Business logic layer
│   │   ├── database.py      # Connection pooling
│   │   ├── interaction.py   # Interaction lookup service
│   │   └── autocomplete.py  # Search/autocomplete service
│   ├── api/                  # JSON API endpoints (/api/v1/*)
│   │   ├── routes.py        # API route handlers
│   │   └── validators.py    # Input validation
│   ├── web/                  # Web page routes
│   │   └── routes.py        # HTML page handlers
│   └── errors/               # Error handlers
│       └── handlers.py
├── static/
│   ├── css/                  # Paper Design system
│   │   ├── variables.css    # Design tokens
│   │   ├── base.css         # Reset, typography
│   │   ├── components.css   # Buttons, cards, forms
│   │   └── layout.css       # Page structure
│   └── js/                   # Modular JavaScript (ES modules)
│       ├── config.js        # API endpoints
│       ├── api.js           # ApiClient class
│       ├── utils.js         # Debounce, helpers
│       ├── autocomplete.js  # Autocomplete with debouncing
│       ├── form.js          # AJAX form submission
│       └── main.js          # Entry point
├── templates/
│   ├── index.html           # Main page template
│   └── error.html           # Error page template
├── migrations/               # Database migrations
│   ├── 001_fix_schema.sql
│   ├── 002_add_foreign_keys.sql
│   ├── 003_fix_boolean_functions.sql
│   ├── 004_bidirectional_lookup.sql
│   └── 005_optimized_procedures.sql
├── tests/                    # Test suite
│   ├── conftest.py          # Pytest fixtures
│   ├── test_api.py          # API endpoint tests
│   ├── test_services.py     # Service unit tests
│   └── test_web.py          # Web route tests
├── application.py            # Entry point (for AWS EB compatibility)
├── requirements.txt          # Python dependencies
├── .env.example             # Environment template
└── pytest.ini               # Test configuration
```

### Backend (Flask + MySQL)

- **Application Factory**: `app/__init__.py` contains `create_app()` for creating Flask instances
- **Configuration**: Environment-based config via `app/config.py` and `.env` file
- **Database**: Connection pooling via `app/services/database.py` using mysql-connector-python
- **Services Layer**: Business logic separated from routes
  - `InteractionService`: Medication validation and interaction lookup
  - `AutocompleteService`: Search functionality
- **API Routes**: JSON API at `/api/v1/*` for AJAX requests
- **Web Routes**: HTML page routes with legacy endpoint compatibility

The Flask variable is named `application` (not `app`) for AWS Elastic Beanstalk compatibility.

### Database

Uses MySQL database `projet_ipa` with optimized stored procedures:

- `get_full_interactions(med1, med2)`: Single-call interaction lookup (replaces 10+ queries)
- `validate_medication(name)`: Combined type validation
- `get_classes_for_medication(name)`: Get classes regardless of input type
- `isClasse`, `isSubstance`, `isSpecialite`: Return TINYINT(1) (not VARCHAR "True"/"False")

Three medication entity types exist: **classes** (drug categories), **substances** (active ingredients), and **specialites** (brand names).

### Frontend

- **ES Modules**: Modern JavaScript with import/export
- **ApiClient**: Centralized API communication with proper error handling
- **Debouncing**: 300ms debounce on autocomplete to reduce API calls
- **AJAX Forms**: No page reload on form submission
- **Paper Design**: Clean, accessible UI with design tokens

### Deployment

AWS Elastic Beanstalk configuration in `.ebextensions/python.config` sets WSGI path to `application:application`.

## API Endpoints

### New API (JSON)

- `POST /api/v1/validate` - Validate medication (replaces 3 legacy calls)
- `POST /api/v1/interactions` - Get interactions between medications
- `GET /api/v1/autocomplete?q=` - Autocomplete search
- `POST /api/v1/classes` - Get classes for a substance

### Legacy Endpoints (maintained for compatibility)

- `POST /testClasse`, `/testSubstance`, `/testSpecialite`
- `POST /getListClasses`
- `POST /autocomplete_input`

## Environment Variables

See `.env.example` for all available options:

- `FLASK_ENV` - development/production
- `SECRET_KEY` - Flask secret key
- `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME` - Database config
- `DB_POOL_SIZE` - Connection pool size

## Key Improvements (v2.0)

1. **No `eval()`**: Database functions return proper booleans
2. **Connection Pooling**: Reuses database connections
3. **Single API Call**: Validation uses 1 call instead of 3
4. **Debounced Autocomplete**: Reduces server load
5. **AJAX Submission**: No page reload
6. **Bidirectional Lookup**: Interactions found regardless of input order
7. **Input Validation**: Server-side sanitization prevents injection
8. **Accessibility**: WCAG 2.2 AA compliant (skip links, ARIA, focus management)

---
> Source: [AmielDylan/projet-iam](https://github.com/AmielDylan/projet-iam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
