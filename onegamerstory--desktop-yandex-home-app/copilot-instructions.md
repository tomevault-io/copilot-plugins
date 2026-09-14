## desktop-yandex-home-app

> Файловые операции за пределами разрешённых opencode директорий.

# MCP-инструменты для проекта Yandex Smart Home Control

## Доступные MCP-серверы

### 1. Filesystem (`@modelcontextprotocol/server-filesystem`)
Файловые операции за пределами разрешённых opencode директорий.
- `read_file`, `write_file`, `edit_file` — чтение/запись/редактирование файлов
- `create_directory`, `list_directory`, `directory_tree` — работа с директориями
- `move_file`, `search_files`, `get_file_info` — поиск и метаданные
- Разрешён доступ только к корню проекта

### 2. Sequential Thinking (`@modelcontextprotocol/server-sequential-thinking`)
Структурированное пошаговое мышление для сложных задач.
- `sequential_thinking` — вызвать с параметрами: `thought`, `nextThoughtNeeded`, `thoughtNumber`, `totalThoughts`
- Опционально: `isRevision`, `revisesThought`, `branchFromThought`, `branchId`, `needsMoreThoughts`
- Использовать для: рефакторинга, отладки, архитектурных решений, анализа зависимостей

### 3. Memories.sh (`@memories.sh/cli`)
Персистентная память для coding-агентов (SQLite, опционально облачная синхронизация).
- `add_memory` — запись контекста, правил, фактов о проекте
- `search_memories` — поиск по сохранённым воспоминаниям
- `get_context` — получение релевантного контекста для текущей задачи
- `list_memories`, `edit_memory`, `forget_memory` — управление памятью
- `add_reminder`, `list_reminders`, `run_due_reminders` — напоминания
- Использовать для: запоминания архитектурных решений, конфигурации устройств Yandex IoT, токенов, предпочтений, правил работы с проектом

### 4. CodeGraph (`@sdsrs/code-graph`)
Индексация кодовой базы в AST-граф знаний.
- `project_map` — архитектура проекта (модули, зависимости, entry points)
- `semantic_code_search` — семантический поиск (BM25 + векторный + графовый)
- `get_call_graph` — цепочки вызовов функций (callers/callees)
- `impact_analysis` — анализ влияния изменений на символ
- `find_references` — поиск всех ссылок на символ
- `find_dead_code` — поиск неиспользуемого кода
- `ast_search` — поиск AST-узлов с фильтрацией по типу, параметрам, возвращаемому типу
- `dependency_graph` — визуализация зависимостей между модулями
- Использовать перед рефакторингом, для понимания архитектуры, поиска багов

## Порядок использования

1. **Анализ задачи** → `sequential_thinking` для структурирования подхода
2. **Изучение кода** → `codegraph` (project_map, semantic_code_search, get_call_graph)
3. **Работа с файлами** → встроенные инструменты opencode + `filesystem` MCP если нужно
4. **Сохранение контекста** → `memory` для записи важных решений и конфигурации
5. **Валидация** → запускать сборку (`npm run build`) после изменений

---
> Source: [onegamerstory/Desktop-Yandex.Home-App](https://github.com/onegamerstory/Desktop-Yandex.Home-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
