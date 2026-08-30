## kinderforge

> - Django-based adaptive learning platform with frustration-aware mastery tracking.

# KinderForge

## Project summary
- Django-based adaptive learning platform with frustration-aware mastery tracking.
- Primary apps: accounts (roles), mastery (engine + tracking), content (concepts/courses), dashboard (UI), ai (OpenAI/Azure stubs).

## Stack
- Python 3.12, Django 6.0.2, SQLite (default).
- YAML-based concept definitions in `concepts.yaml`.

## Key paths
- `masteryforge/settings.py`: core settings + env vars.
- `accounts/`, `mastery/`, `content/`, `dashboard/`, `ai/`: main apps.
- `mastery/engine.py`: mastery/frustration logic.
- `mastery/graph.py`: concept eligibility + selection rules.
- `mastery/management/commands/load_concepts.py`: imports YAML concepts.
- `docker-entrypoint.sh`: auto-migrate, load concepts, create default users.
- `concepts.yaml`: concept graph source.

## Environment variables
- `DJANGO_SECRET_KEY`, `DJANGO_DEBUG`, `DJANGO_ALLOWED_HOSTS`.
- OpenAI: `OPENAI_API_KEY`, `OPENAI_MODEL`.
- Azure OpenAI: `AZURE_OPENAI_RESOURCE_NAME`, `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_DEPLOYMENT`, `AZURE_OPENAI_API_VERSION`, `AZURE_OPENAI_MODEL`.

## Local setup
```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py load_concepts
python manage.py createsuperuser
python manage.py runserver 0.0.0.0:8000
```

## Docker setup
```bash
docker build -t masteryforge -f Dockerfile .
docker run -d --name masteryforge -p 8000:8000 -v masteryforge-data:/app/data masteryforge
```

## Common commands
- Run server: `python manage.py runserver 0.0.0.0:8000`
- Load concepts: `python manage.py load_concepts --file concepts.yaml`
- Admin user: `python manage.py createsuperuser`
- Tests (if needed): `python manage.py test`

## Default accounts (Docker entrypoint)
- Admin: `admin` / `admin123`
- Student: `alice` / `student123`
- Student: `bob` / `student123`
- Parent: `parent1` / `parent123`

## Notes for changes
- If you modify concept schema, update `concepts.yaml` and `mastery/management/commands/load_concepts.py`.
- Changes to recommendation logic live in `mastery/engine.py` and `mastery/graph.py`.
- UI flows are in `dashboard/views.py` and templates under `dashboard/templates/`.

---
> Source: [dmulder/KinderForge](https://github.com/dmulder/KinderForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
