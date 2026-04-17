## kgboard

> Версия хранится в `gradle.properties` → поле `pluginVersion`.

# KGBoard — Project Rules

## Версионирование (Semantic Versioning)

Версия хранится в `gradle.properties` → поле `pluginVersion`.

**При КАЖДОМ коммите ОБЯЗАТЕЛЬНО:**
1. Повышай версию в `gradle.properties` соответственно уровню изменений
2. Обновляй `CHANGELOG.md` — добавляй запись в секцию текущей версии (или создавай новую секцию при смене версии)

Формат CHANGELOG — [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Категории: Added, Changed, Fixed, Removed, Deprecated, Security.

**Уровни изменений:**

### PATCH (0.1.0 → 0.1.1)
- Исправление багов
- Мелкие правки UI/текстов
- Рефакторинг без изменения поведения
- Обновление зависимостей
- Изменения в тестах/документации

### MINOR (0.1.0 → 0.2.0)
- Новая функциональность (backwards-compatible)
- Новые listener'ы, эффекты, настройки
- Новые UI-секции
- Поддержка новых событий IDE

### MAJOR (0.1.0 → 1.0.0)
- Breaking changes в конфигурации (несовместимые с предыдущими `kgboard.xml`)
- Смена протокола OpenRGB
- Удаление существующей функциональности
- Крупные архитектурные изменения

## Структура проекта

```
src/main/kotlin/com/kgboard/rgb/
├── client/     — OpenRGB TCP клиент и протокол
├── effect/     — Система эффектов (static, pulse, flash, progress)
├── bridge/     — Listener'ы событий IDE → эффекты
└── settings/   — Настройки и UI конфигурации
```

## Технологии
- IntelliJ Platform Plugin SDK 2.3.0
- Kotlin 1.9.25, JDK 21
- Target: IntelliJ IDEA 2024.3+ (sinceBuild=243)
- OpenRGB SDK (TCP, порт 6742)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dzerik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
