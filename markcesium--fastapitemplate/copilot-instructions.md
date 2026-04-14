## fastapitemplate

> Шаблон для FastAPI-проектов с чистой архитектурой.

# FastAPITemplate

Шаблон для FastAPI-проектов с чистой архитектурой.

## Структура проекта

```
.
├── backend/                         # FastAPI-приложение
├── docker-compose.yaml              # Базовые сервисы (database + backend)
├── docker-compose.dev.yaml          # Dev-оверрайды (порты, volume-маунты, hot-reload)
├── docker-compose.prod.yaml         # Prod-оверрайды (production Dockerfile)
├── Makefile                         # make dev / prod / down / logs / migration / migrate
└── .env.template                    # Переменные для PostgreSQL-контейнера
```

## Стек

- **Python 3.14+**, FastAPI, Hypercorn (ASGI)
- **PostgreSQL 17** + SQLAlchemy 2.0 (async) + Alembic
- **Dishka** — DI-контейнер (scopes: APP, REQUEST)
- **Pydantic v2** + pydantic-settings (env `__` разделитель)
- **orjson** — ORJSONResponse по умолчанию
- **uv** — пакетный менеджер
- **Ruff** — линтер/форматтер, **mypy** — тайпчекер (strict)

## Docker-сервисы

| Сервис   | Образ              | Порт (dev) |
|----------|--------------------|------------|
| database | postgres:17        | 5432       |
| backend  | backend            | 8000       |

## Makefile-команды

```bash
make dev                    # Запуск dev-окружения с hot-reload
make prod                   # Запуск prod-окружения
make down                   # Остановка всех сервисов
make logs                   # Логи
make migration m="описание" # Создать миграцию Alembic
make migrate                # Применить миграции
make format                 # ruff check --fix + ruff format
make check                  # ruff check + mypy
```

## Конфигурация

### Корневой .env

```
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=app
```

### Backend .env (backend/.env)

Pydantic Settings с разделителем `__`:

- `APP__DEBUG`, `APP__ALLOWED_ORIGINS`
- `LOGGING__LEVEL`
- `DATABASE__URL`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MarkCesium)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MarkCesium)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
