## travel-app

> **IMPORTANT FOR CLAUDE: Reference this file before implementing anything**

**IMPORTANT FOR CLAUDE: Reference this file before implementing anything**

# Project: Travel Planning Application [CL]

## Project Overview [CL-OVW]
Django web app for trip planning with itinerary management and weather integration.
- See [FUNCTIONAL.md] for detailed feature requirements
- See [ARCHITECTURE.md] for technical implementation details

## Tech Stack [CL-TECH]
- Language: Python 3.8+ [CL-TECH-LANG]
- Framework: Django 4.2+ [CL-TECH-FRAME]
- Database: PostgreSQL [CL-TECH-DB]
- Frontend: Django Templates with CSS/JS [CL-TECH-FRONT]
- Libraries: DRF, Requests, Pillow [CL-TECH-LIB]
- Package Manager: pip [CL-TECH-PKG]

## Code Style & Conventions [CL-STYLE]

### Import/Module Standards [CL-STYLE-IMP]
```python
# 1. Standard library imports
import datetime
import uuid

# 2. Django imports
from django.db import models
from django.shortcuts import render

# 3. Third-party app imports
import requests
from PIL import Image

# 4. Local app imports
from trips.models import Trip
from weather.services import get_weather
```

### Naming Conventions [CL-STYLE-NAME]
- Functions: `snake_case()` (create_trip, get_weather)
- Classes/Models: `CamelCase` (Trip, Activity)
- Constants: `UPPER_CASE_WITH_UNDERSCORES` (API_KEY)
- Files: `snake_case.py` (models.py, trip_views.py)
- Templates: `app_name/model_action.html` (trips/trip_detail.html)
- URL patterns: `snake_case-descriptive-names` (trip-detail)

### Patterns to Follow [CL-STYLE-PAT]
- Use Django's MTV pattern (ARCH-SYS-PAT)
- Use class-based views for complex functionality
- Create form classes for all data input and validation
- Implement service pattern for external API calls (weather)
- Follow OOP principles:
  - Encapsulation: Keep data and methods together
  - Single Responsibility: One class = one responsibility
  - Composition over inheritance: Prefer composing objects

## Development Workflow [CL-DEV]
- Branch naming: `feature/trip-creation`, `fix/weather-api` [CL-DEV-BRANCH]
- Commit format: [CL-DEV-COMMIT]
  ```
  feat: Add trip creation form
  fix: Handle weather API errors
  docs: Update README with setup instructions
  style: Format code according to PEP 8
  refactor: Simplify weather data caching
  test: Add tests for trip model validation
  chore: Update dependencies
  ```
- Pull Request requirements: [CL-DEV-PR]
  - All tests passing
  - Code follows style guidelines
  - Documentation updated

## Testing [CL-TEST]
- Use Django's TestCase for all testing [CL-TEST-FRAME]
- Test naming convention: `test_should_<description>_when_<condition>` [CL-TEST-NAME]
- Required test coverage: All model methods, form validation, views [CL-TEST-COV]

## Environment Setup [CL-ENV]
Required variables in `.env` file: [CL-ENV-VARS]
```
DATABASE_URL=postgresql://username:password@localhost:5432/travelapp
SECRET_KEY=your_secret_key
WEATHER_API_KEY=your_weather_api_key
```

## Common Commands [CL-CMD]
```bash
# Create/activate virtual environment
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt

# Database operations
python manage.py makemigrations
python manage.py migrate

# Run server
python manage.py runserver

# Testing
python manage.py test
```

## Project Structure [CL-STRUCT]
See [ARCHITECTURE.md#project-structure] for detailed layout.

## Review Checklist [CL-REVIEW]
Before submitting code, verify: [CL-REVIEW-CHECK]
1. Run `python manage.py check` and `python manage.py test`
2. Ensure code follows PEP 8 and Django style
3. Verify naming conventions are followed
4. Confirm code follows MTV pattern and OOP principles
5. Check proper error handling is implemented
6. Verify tests cover key functionality
7. Ensure docstrings document all classes/functions

## Error Handling [CL-ERR]
- Use Django's error pages for HTTP errors [CL-ERR-HTTP]
- Implement try-except for external operations: [CL-ERR-TRY]
  ```python
  def get_weather(location):
      try:
          response = requests.get(f"{API_URL}?location={location}&key={API_KEY}")
          response.raise_for_status()
          return response.json()
      except requests.RequestException as e:
          logger.error(f"Weather API error: {e}")
          return None
  ```
- Log all errors appropriately [CL-ERR-LOG]
- Show user-friendly messages [CL-ERR-MSG]

## Development Principles [CL-PRIN]
- **KISS**: Choose simplest implementation that meets requirements [CL-PRIN-KISS]
- **DRY**: Extract repeated code into functions/utilities [CL-PRIN-DRY]
- **Convention over Configuration**: Follow Django conventions [CL-PRIN-CON]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markhodierne) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
