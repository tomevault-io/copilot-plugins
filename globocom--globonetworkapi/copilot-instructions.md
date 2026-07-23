## globonetworkapi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GloboNetworkAPI is a REST API for managing IP networking resources (VLAN, IP, Network, Equipment, VIP, etc.). Built with Django 1.5 and Python 2.7, running on MySQL with Memcached, Celery/RabbitMQ for async tasks.

## Development Environment

All development runs inside Docker containers via docker-compose.

```bash
# Start all services (MySQL, RabbitMQ, Memcached, Celery, NetAPI)
make start

# Stop all services
make stop

# Shell into app container
make api

# Shell into database container
make db

# View logs
make logs

# Build Docker images
make build_img
```

## Running Tests

Tests run inside the Docker container. The test settings module is `networkapi.settings_ci`.

```bash
# Full test suite (fresh DB each run)
make test_ci

# Full test suite (reuses DB, faster)
make test

# Specific app tests
make test_ci app=networkapi/api_vlan
make test app=networkapi/plugins/SDN/

# Single test module from inside the container
docker exec -it netapi_app ./fast_start_test_reusedb.sh networkapi/api_vlan
```

Test runner uses `python manage.py test` with nose (`django-nose`). Test base class is `NetworkApiTestCase` in `networkapi/test/test_case.py`, which provides helpers for HTTP auth, JSON comparison, and fixture loading.

## Code Quality

Pre-commit hooks enforce: trailing whitespace, autopep8 (ignoring E501), flake8 (ignoring E501, E402, F401-F405, F821, F841), import reordering, JSON/YAML validation.

## Architecture

### API Module Pattern

Each `api_*` module under `networkapi/` follows this structure:

```
api_<resource>/
├── views/v1.py, v3.py    # DRF viewsets (API versions)
├── facade/v1.py, v3.py   # Business logic layer
├── serializers.py         # DRF serializers
├── permissions.py         # Permission classes
├── exceptions.py          # Custom exceptions
├── tasks.py               # Celery async tasks
├── urls.py                # URL routing
├── specs/                 # JSON specs for request validation
├── fixtures/              # Test fixtures (JSON)
└── tests/
    ├── sanity/sync/       # End-to-end tests (synchronous)
    └── unit/async/        # Unit tests (async operations)
```

Views delegate to facades for business logic; facades interact with models. Async operations (deploy/undeploy) go through Celery tasks.

### Legacy vs Modern Modules

- **`api_*` modules**: RESTful API endpoints (v1, v3) using Django REST Framework
- **Non-prefixed modules** (`equipamento/`, `vlan/`, `ip/`, `ambiente/`): Legacy modules with older-style views and XML-based responses

Both coexist; legacy modules contain the Django models used by the `api_*` facades.

### URL Routing

All API endpoints mount under `/api/` in `networkapi/urls.py`. Each `api_*` module provides its own `urls.py` included from the main router.

### Plugins

`networkapi/plugins/` contains vendor-specific equipment plugins (Cisco NXOS, Dell FTOS, Juniper JUNOS, SDN/ODL) for deploying configurations to network devices.

### Database Migrations

Uses `simple-db-migrate` (not Django migrations). Migration files are in `dbmigrate/migrations/`. Config in `dbmigrate/simple-db-migrate.conf`.

### Key Configuration

- Settings: `networkapi/settings.py` (main), `networkapi/settings_ci.py` (tests)
- Celery: `networkapi/celery_app.py`
- Environment variables: `NETWORKAPI_DATABASE_HOST`, `NETWORKAPI_DATABASE_USER`, `NETWORKAPI_DATABASE_PASSWORD`, `NETWORKAPI_DATABASE_PORT`, `NETWORKAPI_MEMCACHE_HOSTS`, `NETWORKAPI_DEBUG`, `DJANGO_SETTINGS_MODULE`

---
> Source: [globocom/GloboNetworkAPI](https://github.com/globocom/GloboNetworkAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
