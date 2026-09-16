## scheduler

> Проект представляет собой SaaS-систему для автоматического составления университетских расписаний. Основа - существующее Laravel-приложение с MVC-архитектурой, Filament-админкой и Docker-контейнеризацией. Цель - превращение в production-ready SaaS-продукт с автогенерацией расписаний и мульти-арендой.

# SPEC-1: University SaaS Scheduling System

## Overview

Проект представляет собой SaaS-систему для автоматического составления университетских расписаний. Основа - существующее Laravel-приложение с MVC-архитектурой, Filament-админкой и Docker-контейнеризацией. Цель - превращение в production-ready SaaS-продукт с автогенерацией расписаний и мульти-арендой.

## Architecture

### Tech Stack

- **Backend**: Laravel 10 (PHP 8.2)
- **Frontend**: Filament Admin (Alpine.js + Tailwind)
- **Database**: MySQL (может быть заменён на PostgreSQL)
- **Queue**: Redis
- **Solver Service**: Go с CP-SAT алгоритмом
- **Containerization**: Docker Compose

### Component Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                        NGINX                                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                   Laravel App                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Models    │  │  Filament  │  │   API Controllers   │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
│         │                │                    │              │
│  ┌──────▼────────────────▼────────────────────▼──────────┐  │
│  │              Tenant Manager (Middleware)               │  │
│  └────────────────────────────────────────────────────────┘  │
└─────────────────────┬───────────────────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        │             │             │
┌───────▼──────┐ ┌────▼────┐ ┌────▼────────┐
│   MySQL      │ │  Redis  │ │  Go Solver  │
│  (tenants)   │ │ (queue) │ │   Service   │
└──────────────┘ └─────────┘ └─────────────┘
```

## Database Schema

### Tenant Isolation

Все таблицы содержат `tenant_id` (UUID) для изоляции данных между арендаторами.

### Core Tables

| Table | Description |
|-------|-------------|
| `tenants` | Арендаторы (вузы/организации) |
| `users` | Пользователи с ролями (owner/admin/planner/teacher/viewer) |
| `rooms` | Аудитории с вместимостью и типами |
| `calendars` | Календари (семестры) с настройками |
| `time_slots` | Временные слоты (день/номер пары/время) |
| `activities` | Активности (занятия для размещения) |
| `activity_groups` | Связь активностей с группами |
| `activity_teachers` | Связь активностей с преподавателями |
| `teacher_unavailability` | Недоступность преподавателей |
| `teacher_preferences` | Пожелания преподавателей |
| `soft_weights` | Веса мягких ограничений |
| `schedule_versions` | Версии расписаний (draft/published/archived) |
| `schedule_assignments` | Назначения занятий в слоты |
| `violations` | Нарушения ограничений |
| `audit_logs` | Логи аудита |
| `import_jobs` | Задания импорта |

## Models

### Tenant
```php
App\Models\Tenant
- id (uuid, primary)
- name, subdomain, domain
- settings (jsonb) - настройки арендатора
- relationships: users, rooms, calendars, activities, scheduleVersions
```

### Room
```php
App\Models\Room
- id, tenant_id (uuid)
- code, title, capacity
- room_type (lecture/lab/seminar/pc/gym/other)
- features (jsonb), active (bool)
```

### Calendar
```php
App\Models\Calendar
- id, tenant_id (uuid)
- name, start_date, end_date
- weeks, parity_enabled
- days_per_week, slots_per_day
- slot_duration_minutes, break_duration_minutes
```

### Activity
```php
App\Models\Activity
- id, tenant_id (uuid)
- subject_id, calendar_id
- title, activity_type, duration_slots
- required_slots_per_period
- relationships: subject, calendar, groups, teachers
```

### ScheduleVersion
```php
App\Models\ScheduleVersion
- id, tenant_id (uuid)
- calendar_id, name, status (draft/published/archived)
- created_by, parent_version_id
- version_number, random_seed (для воспроизводимости)
- generation_params (jsonb), published_at
```

## Multi-Tenancy Implementation

### TenantManager

Сервис для управления текущим арендатором:

```php
App\Services\TenantManager
- getTenant(): ?Tenant
- setTenant(Tenant $tenant): void
- setTenantById(string $tenantId): ?Tenant
- resolveTenantId(): ?string (из subdomain/domain/auth)
- clearTenant(): void
```

### TenantMiddleware

Middleware для определения арендатора из HTTP-запроса:
1. Проверяет subdomain/domain в URL
2. Проверяет tenant_id авторизованного пользователя
3. Устанавливает tenant в контейнер приложения

### TenantScope Trait

Трейт для автоматической фильтрации по tenant_id:

```php
App\Models\Traits\TenantScope
- scopeTenant(Builder $query, ?string $tenantId = null): Builder
- getTenantId(): ?string
- bootTenantScope(): auto-set tenant_id при создании
```

## Filament Admin Resources

### Available Resources

| Resource | Navigation Group | Description |
|----------|-----------------|-------------|
| `TenantResource` | SaaS | Управление арендаторами |
| `RoomResource` | Розклад | Аудитории |
| `CalendarResource` | Розклад | Календари/семестры |
| `ActivityResource` | Розклад | Активности/занятия |
| `TeacherResource` | Управління даними | Преподаватели |
| `GroupResource` | Управління даными | Группы |
| `SubjectResource` | Управління даными | Дисциплины |
| `CourseResource` | Управління даными | Курсы |

## Go Solver Service

### Purpose

Отдельный сервис для вычислительно тяжёлых задач оптимизации расписания.

### Location

```
solver/
├── cmd/server/main.go      # Entry point
├── internal/
│   ├── solver/scheduler.go # Optimization logic
│   └── db/postgres.go      # Database operations
└── pkg/types/types.go      # Data structures
```

### API

HTTP API для запуска оптимизации:

```json
POST /api/v1/generate
{
  "tenant_id": "uuid",
  "calendar_id": 1,
  "schedule_id": 1,
  "weights": {
    "w_windows": 10,
    "w_prefs": 5,
    "w_balance": 2
  },
  "timeout_seconds": 420
}
```

Response:
```json
{
  "status": "FEASIBLE",
  "assignment_ids": [...],
  "violations": [...],
  "total_violations": 5,
  "solve_time_ms": 15000
}
```

### Algorithm

Жадный алгоритм с учётом:
- **Hard constraints**: конфликты преподавателей/групп/аудиторий, вместимость, недоступность
- **Soft constraints**: окна, пожелания, равномерность нагрузки

## User Roles

| Role | Permissions |
|------|-------------|
| `owner` | Полный доступ, управление арендаторами |
| `admin` | Управление всеми данными, запуск генерации |
| `planner` | Составление расписания |
| `teacher` | Просмотр, редактирование предпочтений |
| `viewer` | Только просмотр |

## Seeders

Порядок запуска сидеров:

1. `AdminUserSeeder` - создание admin пользователя
2. `TenantSeeder` - демо-университет
3. `RoomSeeder` - 8 аудиторий
4. `CalendarSeeder` - весенний семестр 2026
5. `TimeSlotSeeder` - 36 слотов (6 дней × 6 пар)
6. `TeacherSeeder` - преподаватели
7. `CourseSeeder` - курсы
8. `GroupSeeder` - группы
9. `SubjectSeeder` - дисциплины

## Docker Configuration

### Services

```yaml
app:        Laravel приложение
nginx:      Web сервер
mysql:      База данных
redis:      Очередь заданий
solver:     Go сервис оптимизации
composer:   Утилита для зависимостей
```

### Environment Variables

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_DATABASE=scheduler
DB_USERNAME=root
DB_PASSWORD=secret
```

## Workflow: Генерация расписания

1. Админ настраивает календарь, аудитории, активности
2. Запускает генерацию через UI
3. Задание попадает в очередь Redis
4. Go-сервис обрабатывает задачу:
   - Загружает данные (activities, rooms, time_slots, preferences)
   - Строит модель ограничений
   - Решает задачу оптимизации
   - Сохраняет результат как draft
5. Админ проверяет результат и публикует

## Extending the System

### Adding New Constraints

1. Добавить поле в `soft_weights` таблицу
2. Обновить `ScheduleRequest` в Go-сервисе
3. Модифицировать `buildObjectives()` в scheduler.go
4. Пересобрать и запустить solver

### Adding New Entity

1. Создать миграцию с tenant_id
2. Создать модель с TenantScope trait
3. Создать Filament Resource
4. Зарегистрировать в FilamentServiceProvider
5. Добавить сидер

## Performance

- **Generation**: ~100 групп, 80 преподавателей, 60 аудиторий на неделю ≤ 10 минут
- **UI Response**: < 200ms на основные операции
- **Database**: Индексы по tenant_id на всех таблицах

## Security

- Row-level isolation по tenant_id
- Role-based access control (RBAC)
- Аудит всех изменений в `audit_logs`
- Валидация входных данных на уровне моделей и контроллеров

---
> Source: [sviniabanditka/scheduler](https://github.com/sviniabanditka/scheduler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
