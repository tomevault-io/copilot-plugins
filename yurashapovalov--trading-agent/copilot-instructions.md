## trading-agent

> AI-ассистент для анализа торговых данных NQ futures.

# Stock Trading Assistant

AI-ассистент для анализа торговых данных NQ futures.

## Быстрый старт

```bash
source .venv/bin/activate

# Unit tests
pytest agent/tests/ -v

# Integration tests
python -m agent.tests.test_graph -s    # single questions
python -m agent.tests.test_graph -i    # interactive
```

## Архитектура

**Принцип:** Code for Facts, LLM for Text

```
User → Parser → Executor → Presenter → Response
         ↓         ↓           ↓
     понимает   SQL/data    текст
```

Подробнее: [docs/architecture/](docs/architecture/)
- [overview.md](docs/architecture/overview.md) — общая картина
- [agents.md](docs/architecture/agents.md) — Intent, Parser, Planner, Executor
- [graph.md](docs/architecture/graph.md) — LangGraph, роутинг, state
- [memory.md](docs/architecture/memory.md) — token caching, conversation memory
- [data-layer.md](docs/architecture/data-layer.md) — Executor, 9 операций
- [config.md](docs/architecture/config.md) — паттерны, события, праздники
- [rules.md](docs/architecture/rules.md) — валидация и auto-fix ошибок LLM
- [rap.md](docs/architecture/rap.md) — Retrieval-Augmented Prompting

## Deployment

Auto-deploy on push to `main`. См. [DEPLOY.md](DEPLOY.md)

## Структура

```
agent/
├── agents/        # Parser, Clarifier, Responder, Presenter
├── prompts/       # LLM промпты + RAP
├── memory/        # Token cache + conversation
├── operations/    # list, count, compare, around...
├── rules/         # Валидация операций и фильтров
├── config/        # patterns, events, holidays
├── graph.py       # LangGraph
├── executor.py    # Query execution
├── types.py       # Pydantic models + auto-fix validators
└── tests/         # pytest + integration

docs/
├── architecture/  # Описание системы
├── plans/         # Roadmap, идеи
└── gemini/        # Gemini API guides
```

## Стек

- Python 3.11+, DuckDB, Google Gemini, Pydantic, LangGraph

## Code Style

<details>
<summary>Senior+ стандарты (click to expand)</summary>

### Типизация
```python
# ДА
def calculate_stats(rows: list[dict], metric: str) -> StatsResult: ...

# НЕТ
def calculate_stats(rows, metric): ...
```

### Функции
```python
# ДА — маленькие, одна задача
def extract_dates(rows: list[dict]) -> list[str]: ...
def check_holidays(dates: list[str]) -> dict: ...

# НЕТ — большая функция делает всё
def process_data(rows): ...  # 100 строк
```

### Ранний выход
```python
# ДА
if not name: return None
if name not in PATTERNS: return None
return PATTERNS[name]

# НЕТ
if name:
    if name in PATTERNS:
        return PATTERNS[name]
return None
```

### Принципы
- **Explicit > Implicit** — никакой магии
- **Flat > Nested** — max 2 уровня
- **Pure > Stateful** — чистые функции
- **Data > Objects** — dataclass вместо классов

</details>

## TODO

### Done
- [x] Cleanup артефактов
- [x] Token caching (73% savings)
- [x] Unit tests (127 tests)
- [x] Supabase persistence (conversation memory)
- [x] Patterns integration (30 patterns: candle + price)
- [x] RAP system (embeddings 1000 chars)
- [x] Auto-fix validators (operation/filter conflicts)

### Next
- [ ] Anomaly Finder (docs/plans/anomaly_finder.md)
- [ ] More instruments (ES, CL, GC)
- [ ] Auto-detect user language → key_facts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yurashapovalov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
