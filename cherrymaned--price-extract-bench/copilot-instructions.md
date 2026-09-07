## price-extract-bench

> Бенчмарк LLM-экстракции структурированных данных о товарах

# price-extract-bench

Бенчмарк LLM-экстракции структурированных данных о товарах
из грязного HTML маркетплейсов + агрегатор цен для Армении.

## Стек
Python 3.12, uv, pydantic v2, selectolax, pytest, ruff, mypy, Langfuse

## Команды
uv run pytest
uv run ruff check .
uv run mypy src/

## Конвенции
- Все внешние вызовы через кэш на диске (по хешу входа)
- Опциональные поля явно `| None`, ограниченные наборы через `Literal`
- Type hints обязательны

## Правила
- Не добавляй зависимости без спроса
- Не пиши комментарии к очевидному коду
- Тесты до реализации
- Не трогай .env

---
> Source: [cherrymaned/price-extract-bench](https://github.com/cherrymaned/price-extract-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
