## palomatika

> **PALOMATIKA** — EdTech SaaS платформа для подготовки к экзаменам по математике:

# PALOMATIKA - EdTech Platform for OGE & EGE Math Preparation

## О проекте

**PALOMATIKA** — EdTech SaaS платформа для подготовки к экзаменам по математике:
- **ОГЭ** — для 9 класса (Основной Государственный Экзамен)
- **ЕГЭ** — для 11 класса (Единый Государственный Экзамен)

**ВАЖНО:** ОГЭ и ЕГЭ — это **обособленные направления**, они не пересекаются.

**Ключевая идея:** Уникальная система обучения через "пазлы" — ученики собирают решение из готовых блоков.

**Production URL:** https://palomatika.ru (Timeweb hosting)

---

## КРИТИЧЕСКИ ВАЖНЫЕ ПРИНЦИПЫ

### 1. SVG — основа проекта
- **НИКОГДА** не использовать PNG/JPEG из PDF
- **ВСЕГДА** создавать собственные SVG
- PDF изображения (`docs/oge_data/images/`, `docs/ege_data/images/`) — **только референс**

### 2. Единый источник данных
- ОГЭ: `storage/app/tasks/topic_{id}.json` (геометрия: `*_geometry.json`)
- ЕГЭ: `storage/app/tasks/ege/topic_{id}.json`
- `TaskDataService` — единственный сервис для доступа к данным
- Никаких захардкоженных данных в контроллерах или views

### 3. Номера заданий = номера в PDF
- НЕЛЬЗЯ менять номера для "красоты"
- Текст и ответы — точно как в источнике

---

## Технический стек

| Технология | Версия | Назначение |
|------------|--------|------------|
| PHP | 8.2+ | Backend |
| Laravel | 10 LTS | Framework |
| MySQL | 8.0 | База данных |
| Tailwind CSS | 3.x (CDN) | Стили |
| Alpine.js | 3.x (CDN) | Интерактивность |

**Без сборщиков** (Vite/Webpack). Всё через CDN.

---

## Ключевые пути

| Путь | Назначение |
|------|------------|
| `app/Services/TaskDataService.php` | Центральный сервис доступа к заданиям |
| `app/Services/GeometrySvgRenderer.php` | Рендерер SVG из geometry данных |
| `app/Services/OgeVariantBuilderService.php` | Детерминированный генератор вариантов |
| `app/Services/OgeAttemptService.php` | Жизненный цикл попыток + скоринг |
| `app/Services/TaskAnswerResolver.php` | Нормализация и проверка ответов |
| `app/Http/Controllers/TestPdfController.php` | Mega-контроллер (legacy, 4400+ строк) |
| `storage/app/tasks/` | JSON файлы заданий ОГЭ |
| `storage/app/tasks/ege/` | JSON файлы заданий ЕГЭ |
| `public/images/tasks/` | Изображения к заданиям |
| `.claude/tasks.json` | Kanban задачи проекта |

---

## Роли пользователей

| Роль | Описание |
|------|----------|
| `student` | Решает задачи, участвует в дуэлях |
| `teacher` | Создаёт ДЗ, отслеживает учеников |
| `admin` | Полный доступ |

---

## Деплой (кратко)

- Ветки `claude/*` **автоматически** мержатся в `main` и деплоятся на production
- Post-deploy: `php artisan deploy:refresh` (очистка кэшей + SVG регенерация)
- **Подробности:** используй скилл `deploy-ops`

---

## Production доступ (MCP)

MCP сервер `palomatika-db` (настроен в `.mcp.json`):
- `list_tables` — список таблиц с row counts
- `query` — read-only SQL
- `describe_table` — структура таблицы
- `run_artisan` — artisan-команда из whitelist

---

## Скиллы для детальной информации

Детальная документация вынесена в скиллы, которые загружаются **только когда нужны**:

| Скилл | Когда использовать |
|-------|--------------------|
| `geometry-svg` | Работа с SVG для геометрии: создание, редактирование, svg:bake, geometry.json |
| `oge-tasks` | Работа с заданиями ОГЭ: JSON структура, TaskDataService, типы заданий |
| `ege-tasks` | Работа с заданиями ЕГЭ: структура, типы, процесс создания |
| `deploy-ops` | Деплой, CI/CD, кэши, artisan на production, webhook |

---

## Kanban

Задачи проекта в `.claude/tasks.json`. Страницы: `/kanban`, `/roadmap`, `/forstas`.

При работе над задачей — обновлять статус в `tasks.json` (`backlog` → `in-progress` → `done`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Palomig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
