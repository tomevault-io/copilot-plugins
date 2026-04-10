## cognify

> Этот файл содержит инструкции для всех AI-агентов, работающих с проектом `docwise-dev`.

# AGENTS.md

Этот файл содержит инструкции для всех AI-агентов, работающих с проектом `docwise-dev`.

## Основные правила

**ОБЯЗАТЕЛЬНО** изучите и применяйте «Конституцию проекта» перед началом работы:

- [Навык: Project Guidelines](.agents/skills/project-guidelines/SKILL.md) — язык коммуникации, commit convention, clean code, правила ревью.

## Кратко о структуре проекта

- `backend/` — Laravel 12 (бизнес-логика, API, мульти-тенантность, интеграции)
- `frontend/` — Vue 3 (пользовательский UI)
- `admin/` — React 19 (административная панель `/dashboard`)
- `rag/` — Python/FastAPI (RAG API + Redis Streams воркеры)
- `integrations/` — интеграционные сервисы и конфиги (Telegram, SpeechKit, LLM Proxy, Code Executor)

Запуск/инфраструктура проекта завязаны на `Makefile` и `docker-compose.yml`.

`llm-proxy` поднимается отдельным сервисом и доступен через nginx по префиксу `/api/llm/`.
Конфигурация прокси хранится в `integrations/llm-proxy/config.yaml` (шаблон: `config.example.yaml`).

## Доступные навыки (Skills)

Проект оснащен набором процедурных навыков (`Agent Skills Standards`). Используйте их для специфических задач вместо импровизации.

Навыки можно активировать вручную через `@skill-name` (например, `@docwise-ops`) или они могут быть выбраны автоматически по `description` в `SKILL.md`.

### 📜 Базовые правила

- **Навык**: [Project Guidelines](.agents/skills/project-guidelines/SKILL.md)
- Используется всегда в начале работы.

### 🧱 DevOps & Infrastructure

Для операций с Docker/БД/окружением используйте **Make-команды**.

- **Навык**: [Docwise Ops](.agents/skills/docwise-ops/SKILL.md)
- **Примеры**: `make up`, `make backend-migrate`, `make admin-logs`, `make build-sandbox`.
- ❌ Не используйте `docker compose`/`php artisan` напрямую без веской причины.

### 🐘 Backend (Laravel)

Для разработки API, доменной логики и админских backend-контрактов.

- **Навык**: [Backend Laravel](.agents/skills/backend-laravel/SKILL.md)
- **Ключевое**: tenant isolation (`tenant_id`), тонкие контроллеры, сервисы/репозитории.

### ⚡ Frontend (Vue 3)

Для пользовательского UI.

- **Навык**: [Frontend Vue](.agents/skills/frontend-vue/SKILL.md)
- **Ключевое**: auto imports, PrimeVue 4, TailwindCSS 4, Vue Router/Pinia.

### 🧩 Admin (React)

Для административной панели `/dashboard`.

- **Навык**: [Admin React](.agents/skills/admin-react/SKILL.md)
- **Ключевое**: React Router 7, TanStack Query, Redux Toolkit, Axios.

### 🧠 RAG & ML

Для работы с поисковым контуром и ML-пайплайном.

- **Навык**: [RAG Python](.agents/skills/rag-python/SKILL.md)
- **Ключевое**: Redis Streams pipeline, артефакты, retries, DLQ, валидация чанков.

### 🔌 Integrations

Для Telegram, SpeechKit, LLM Proxy и Code Executor.

- **Навык**: [Integrations Python](.agents/skills/integrations-python/SKILL.md)

### 🔍 Code Review

Для глубокого ревью качества и согласованности кода.

- **Навык**: [Code Review](.agents/skills/code-review/SKILL.md)
- **Ключевое**: баги, риски, антипаттерны, узкие места, SOLID/DRY/KISS/YAGNI, производительность.

## Структура навыков

Единый каталог навыков проекта:

- `.agents/skills/`

Поддерживайте навыки только в этом каталоге.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ivglad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ivglad)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
