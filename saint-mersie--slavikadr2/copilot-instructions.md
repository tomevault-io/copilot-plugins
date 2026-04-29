## slavikadr2

> Choose and read the best-matching skill(s) for each task; skills live under user .cursor/skills and Cursor plugin caches


# Маршрутизация скиллов

Перед **существенной** работой (не одна строка/опечатка) агент **сам выбирает** 1–3 наиболее подходящих скилла, **читает** их `SKILL.md` и следует workflow из файла.

**Где искать**

- Пользовательские: `~/.cursor/skills/<имя-папки>/SKILL.md` (Windows: `%USERPROFILE%\.cursor\skills\`).
- Плагины Cursor: `%USERPROFILE%\.cursor\plugins\cache\cursor-public\<вендор>\...\skills\<тема>\SKILL.md` — когда задача явно про продукт (Vercel, Stripe, Figma, Render, Encore, Supabase и т.д.).

**Как выбирать**

1. Классифицируй задачу: UI, API/бэкенд, БД, тесты, деплой, контент, исследование, безопасность, рефакторинг.
2. Бери **узкий** скилл по стеку/инструменту, а не общий «на всякий случай».
3. Если релевантны оба — сначала доменный (например `django-patterns`), потом общий (`python-patterns`).

## Карта ситуаций → скиллы (`~/.cursor/skills/`)

| Ситуация | Папки скиллов (приоритет сверху) |
|----------|----------------------------------|
| Веб-UI, дизайн-система, визуал | `ui-ux-pro-max` |
| React/фронт архитектура | `frontend-patterns`, `coding-standards` |
| Chakra UI | `ux-ui-chakra` |
| HTML-слайды / презентации | `frontend-slides` |
| REST/API проектирование | `api-design` |
| Node/Express/Next API | `backend-patterns` |
| NestJS | `nestjs-patterns` |
| Django | `django-patterns`, `django-tdd`, `django-verification` |
| Laravel | `laravel-patterns`, `laravel-tdd`, `laravel-verification`, `laravel-plugin-discovery` |
| Spring Boot | `springboot-patterns`, `springboot-tdd`, `springboot-verification`, `java-coding-standards` |
| Go | `golang-patterns`, `golang-testing` |
| Kotlin / сервер | `kotlin-patterns`, `kotlin-ktor-patterns`, `kotlin-exposed-patterns`, `kotlin-coroutines-flows` |
| Kotlin тесты | `kotlin-testing` |
| Rust | `rust-patterns`, `rust-testing` |
| C# / .NET | `dotnet-patterns`, `csharp-testing` |
| C++ | `cpp-coding-standards`, `cpp-testing` |
| Perl | `perl-patterns`, `perl-testing` |
| Python | `python-patterns`, `python-testing` |
| Dart/Flutter | `dart-flutter-patterns` |
| Compose / KMP UI | `compose-multiplatform-patterns` |
| Android clean arch | `android-clean-architecture` |
| Общий TDD / проверка перед релизом | `tdd-workflow`, `tdd-guide`, `verification-loop` |
| E2E (Playwright и т.п.) | `e2e-testing` |
| AI-регрессии / песочница API | `ai-regression-testing` |
| MCP-серверы | `mcp-server-patterns` |
| Stripe | плагин: `.../stripe/.../skills/stripe-best-practices`, `upgrade-stripe` |
| Vercel / Next / AI SDK / shadcn | плагин Vercel: папки `skills` внутри кэша плагина |
| Render | плагин Render: `render-deploy`, `render-debug`, `render-monitor`, `render-workflows`, `render-migrate-from-heroku` |
| Encore (Go/TS) | плагин Encore: `getting-started`, `api`, `database`, и т.д. |
| Supabase Postgres | плагин Supabase: `supabase-postgres-best-practices` |
| Figma ↔ код | плагин Figma: `figma-use`, `figma-implement-design`, `figma-generate-design`, и др. |
| Databricks | плагин Databricks: `databricks`, `databricks-apps` |
| Neon Postgres | плагин Neon |
| Контент / соцсети | `content-engine` |
| Инвесторы / питч | `investor-materials`, `investor-outreach` |
| Рынок / конкуренты | `market-research` |
| Статьи / лонгрид | `article-writing` |
| Документация проекта | `doc-updater` |
| Настройка Cursor (rules/skills) | `skills-cursor/create-rule`, `skills-cursor/create-skill`, `skills-cursor/update-cursor-settings` |
| Качество скиллов | `skill-stocktake` |
| Обучение из сессий | `continuous-learning`, `continuous-learning-v2` |

## Минимальные ожидания

- Для **UI**: см. также `.cursor/rules/ui-ux-pro-max.mdc` при работе с фронтенд-файлами.
- Не перечислять скиллы пользователю как список «на показ» — **просто применять** выбранные.
- Если подходящего скилла нет — опираться на кодовую базу и стандартные практики стека.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saint-mersie) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
