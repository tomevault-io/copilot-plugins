## vokzal

> Общие правила проекта Вокзал.ТЕХ для разработки


# Правила проекта Вокзал.ТЕХ

## Базовые принципы

### Название проекта
- **ВСЕГДА** использовать "Вокзал.ТЕХ" (только на русском)
- НЕ использовать "Deparq", "Vokzal Tech" или другие варианты
- Домен: `vokzal.tech`
- API: `api.vokzal.tech`

### Язык разработки
- Код: английский (переменные, функции, комментарии)
- Документация: русский
- UI: русский с поддержкой английского (i18next)
- Комментарии в коде: английский для технических деталей

### Маскот
- Имя: "Вокзалик"
- Фраза: "Вокзалик знает, когда уезжать!"
- Использовать на страницах ошибок и в маркетинговых материалах

## Соответствие законодательству

### 152-ФЗ (Защита персональных данных)
- Шифровать PII: ФИО, паспорт, телефон (AES-256)
- Логировать все операции с ПД в audit сервис
- Хранить логи 1+ год
- Получать согласие на обработку ПД

### 54-ФЗ (Фискализация)
- Все продажи через ККТ (АТОЛ)
- Чеки в ОФД в течение 30 секунд
- Z-отчёты ежедневно
- Хранить чеки 5 лет

## Архитектура

### Микросервисы (Go)
- Порт по умолчанию: 8080 + номер сервиса (auth=8081, schedule=8082, ticket=8083, fiscal=8084, payment=8085, board=8086, notify=8087, audit=8088, document=8089, geo=8090)
- Структура:
  - `cmd/main.go` — точка входа
  - `internal/` — внутренний код
  - `pkg/` — экспортируемые пакеты
  - `migrations/` — SQL миграции

### Frontend (React)
- React 18.2+, TypeScript 5.0+
- Vite для сборки
- TanStack Query для API запросов
- Tailwind CSS + Fluent UI React

### База данных
- PostgreSQL 15+
- UUID для всех PK
- JSONB для гибких данных (stops, days_of_week)
- Индексы для всех FK

## Стиль кода

### Go
- gofmt для форматирования
- golangci-lint для проверки
- Errors: wrap с контекстом
- Логирование: Zap (JSON format)

### TypeScript/React
- ESLint + Prettier
- Функциональные компоненты
- Custom hooks для переиспользуемой логики
- Props типизация через TypeScript interfaces

## Безопасность

### Аутентификация
- JWT токены (exp: 24h, refresh: 7d)
- RBAC: cashier, dispatcher, controller, admin
- Middleware для проверки ролей

### API
- HTTPS only (TLS 1.3)
- Rate limiting: 100 req/min per IP
- CORS: только `*.vokzal.tech`
- Валидация всех входных данных

## Git workflow

### Ветки
- `main` — production
- `staging` — staging environment
- `feature/*` — новый функционал
- `hotfix/*` — срочные исправления

### Коммиты
- Формат: `type(scope): description`
- Типы: feat, fix, docs, style, refactor, test, chore
- Примеры:
  - `feat(auth): add JWT refresh endpoint`
  - `fix(ticket): correct return penalty calculation`

## Не выдумывай функции
- Используй ТОЛЬКО функции из документации
- Если функция не описана — спроси пользователя
- Не добавляй функционал "на будущее" без запроса

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thevladbog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
