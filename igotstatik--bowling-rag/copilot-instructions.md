## bowling-rag

> RAG-система для боулинг-сообщества. Классифицирует запросы, ищет в базе знаний, генерирует ответы через LLM.

# Bowling RAG Assistant - Project Context

## Overview
RAG-система для боулинг-сообщества. Классифицирует запросы, ищет в базе знаний, генерирует ответы через LLM.

## Tech Stack (финальный)

| Компонент | Технология |
|-----------|------------|
| Framework | FastAPI + LangChain |
| Vector DB | Qdrant (Docker) |
| Embeddings | BAAI/bge-m3 (568M params, 8192 tokens) |
| Reranker | cross-encoder/ms-marco-MiniLM-L-6-v2 |
| LLM | OpenRouter API (DeepSeek) |
| Queue | ARQ (async Redis queue) - легковесная альтернатива Celery |
| Config | Pydantic Settings + YAML |
| Monitoring | LangSmith |
| Deploy | Docker Compose |

## Документы для базы знаний

Папка `/root/bowling_rag/docs_raw/`:
- `Rulebook-25-26_Nov-20-2025.pdf` - правила USBC 2025-2026
- `LeagueLevelHandbook3-0.pdf` - руководство по лигам
- `Bowling+Coaching+Guide.pdf` - техника, коучинг (Special Olympics)
- `mental_game_bowl_com.md` - психология боулинга (BOWL.com)

## Структура проекта

```
bowling_rag/
├── config/
│   └── config.yaml
├── src/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── main.py          # FastAPI app, lifespan
│   │   └── routes.py        # /ask, /health, /ingest
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py        # Pydantic Settings
│   │   └── dependencies.py  # DI
│   ├── services/
│   │   ├── __init__.py
│   │   ├── classifier.py    # Intent classification
│   │   ├── retriever.py     # Qdrant + reranker
│   │   ├── generator.py     # LLM generation
│   │   └── pipeline.py      # RAG orchestration
│   ├── models/
│   │   ├── __init__.py
│   │   └── schemas.py       # Pydantic schemas
│   ├── ingestion/
│   │   ├── __init__.py
│   │   ├── loader.py        # PDF/MD loading
│   │   ├── chunker.py       # Text splitting
│   │   └── indexer.py       # Embeddings + Qdrant
│   └── workers/
│       ├── __init__.py
│       └── tasks.py         # ARQ background tasks
├── data/
│   └── documents/           # Processed docs
├── scripts/
│   ├── ingest.py            # CLI indexing
│   └── test_queries.py      # Test queries
├── tests/
├── docker-compose.yaml
├── Dockerfile
├── requirements.txt
└── .env.example
```

## Intents (классификация)

- `RULES` - правила, регламент, подсчет очков, штрафы
- `TECHNIQUE` - техника броска, стойка, подбор угла
- `PSYCHOLOGY` - mental game, давление, стресс, фокус
- `EQUIPMENT` - шары, обувь, снаряжение
- `GENERAL_SPORT` - общие вопросы о боулинге как спорте
- `OFF_TOPIC` - не про боулинг → отказ отвечать

## API Endpoints

- `POST /api/ask` - основной endpoint для вопросов
- `POST /api/ingest` - добавление документов
- `GET /api/health` - health check
- `GET /api/config` - текущий конфиг (debug)

## Environment Variables

```
OPENROUTER_API_KEY=...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=...
LANGCHAIN_PROJECT=bowling-assistant
REDIS_URL=redis://redis:6379
```

---

## Промпты

### Intent Classification Prompt

```
You are an intent classifier for a bowling assistant chatbot.

Classify the user's query into ONE of these categories:
- RULES: Questions about bowling rules, regulations, scoring, penalties
- TECHNIQUE: Questions about bowling technique, form, throws, spare shooting
- PSYCHOLOGY: Questions about mental game, pressure, stress, focus, confidence
- EQUIPMENT: Questions about bowling balls, shoes, gear
- GENERAL_SPORT: General questions about bowling as a sport
- OFF_TOPIC: Questions not related to sports/bowling at all

User query: {query}

Respond with ONLY the category name, nothing else.
```

### RAG Generation Prompt

```
You are a helpful bowling assistant for athletes. Answer the question based on the provided context.

Rules:
1. Answer in the same language as the question
2. If the context doesn't contain relevant information, say so honestly
3. Be concise but complete
4. If discussing technique, be specific and actionable
5. For rules questions, cite the specific rule if available

Context:
{context}

Question: {query}

Answer:
```

---

## Implementation Plan

### Phase 1: Core Infrastructure ✅
1. [x] Создать структуру папок
2. [x] config.yaml + Pydantic Settings
3. [x] Docker Compose (Qdrant + Redis + App)
4. [x] requirements.txt

### Phase 2: Ingestion Pipeline ✅
5. [x] PDF loader (PyMuPDF) + MD loader
6. [x] Text chunker (500 tokens, overlap 100)
7. [x] Embeddings (BGE-M3)
8. [x] Qdrant indexer
9. [x] CLI script: ingest.py

### Phase 3: RAG Pipeline ✅
10. [x] Intent classifier (LangChain + LLM)
11. [x] Retriever (Qdrant search)
12. [x] Reranker (cross-encoder)
13. [x] Generator (LangChain + OpenRouter)
14. [x] Pipeline orchestration

### Phase 4: API Layer ✅
15. [x] FastAPI app с lifespan
16. [x] Routes: /ask, /health, /ingest
17. [x] Pydantic schemas

### Phase 5: Async Queue (ARQ) ✅
18. [x] Redis setup
19. [x] ARQ worker
20. [x] Background task для long queries

### Phase 6: Testing & Demo
21. [x] Test queries script
22. [ ] Проверка full pipeline
23. [ ] Docker build & run

---

## Узкие места при 1000 RPS

| Компонент | Проблема | Решение |
|-----------|----------|---------|
| Embeddings (CPU) | ~50-100ms/запрос | GPU / отдельный сервис / батчинг |
| Reranker (CPU) | ~30-50ms на 10 docs | GPU / отключить для простых запросов |
| Qdrant | ОК до ~10K RPS | Шардинг, реплики |
| LLM API | Rate limits, 1-3s latency | Кэширование, несколько провайдеров |
| FastAPI | ~500-1000 RPS single instance | Горизонтальное масштабирование |

### Рекомендации для прода
1. **Redis кэш** — для частых/похожих вопросов (semantic cache)
2. **GPU** — для embeddings и reranker
3. **Replicas** — несколько инстансов app за load balancer
4. **Rate limiting** — защита от спама

---

## Дебаггинг галлюцинаций LLM

### Стратегия
1. **LangSmith трейсинг** — видим весь pipeline: что нашли → что отдали LLM → что получили
2. **Source attribution** — всегда возвращаем источники, чтобы можно было проверить
3. **Confidence scoring** — если retrieval score низкий, предупреждаем пользователя
4. **Prompt engineering** — инструкция "If the context doesn't contain relevant information, say so"

### Классификация ошибок
| Тип ошибки | Причина | Решение |
|------------|---------|---------|
| Retrieval failure | Не нашли нужный чанк | Улучшить embeddings, добавить документы |
| Reranker failure | Нужный чанк не попал в top-3 | Увеличить top_k, fine-tune reranker |
| Generation failure | LLM неправильно интерпретировала | Улучшить промпт, сменить модель |
| Knowledge gap | Нет информации в базе | Добавить документы |

---

## Feedback Loop с экспертами

### Процесс улучшения качества

```
1. Сбор данных
   └── LangSmith логирует все вопросы и ответы

2. Разметка экспертами (тренерами)
   └── Оценка: correct / partially_correct / incorrect
   └── Комментарии: что не так, как исправить

3. Анализ ошибок
   ├── Retrieval failure — не нашли нужный чанк
   ├── Reranker failure — нужный чанк не попал в top-3
   ├── Generation failure — LLM плохо сформулировала
   └── Knowledge gap — нет информации в базе

4. Улучшения
   ├── Добавление документов в базу
   ├── Корректировка промптов
   ├── Fine-tuning reranker (если нужно)
   └── Улучшение чанкинга
```

### Метрики качества

| Метрика | Как измерять |
|---------|--------------|
| Intent Accuracy | Тестовый датасет с размеченными интентами |
| Retrieval Recall@K | Есть ли нужный чанк в top-K |
| Answer Correctness | Экспертная оценка (1-5) |
| User Satisfaction | Thumbs up/down в интерфейсе |

### Предложение по организации feedback loop

1. **Еженедельный review** — тренеры смотрят 20-30 случайных ответов
2. **Tagging в LangSmith** — помечаем проблемные ответы
3. **Priority queue** — сначала фиксим частые ошибки
4. **A/B testing** — тестируем изменения на части трафика

---

## Key Decisions

### Почему ARQ вместо Celery?
- Нативная поддержка asyncio
- Легковесный (один файл)
- Отлично работает с FastAPI
- Проще конфигурация

### Почему BGE-M3?
- Лидер на ruMTEB для русского языка
- 8192 токенов контекст (vs 512 у e5) — лучше для длинных документов
- Поддержка dense + sparse + multi-vector retrieval
- Cross-lingual (100+ языков)

### Почему DeepSeek через OpenRouter?
- Дешевле GPT-4
- Хорошее качество для RAG
- Легко сменить на другую модель

---

## Test Queries

```
1. "Где стоит 10-я кегля?" → RULES
2. "Можно ли сбивать кеглю из желоба?" → RULES
3. "Какие штрафы за несанкционированную полировку шаров?" → RULES
4. "Я 3-й раз подряд промахиваюсь в 10-ю кеглю, что делать?" → TECHNIQUE
5. "Соперник давит морально, как защититься?" → PSYCHOLOGY
6. "Какая погода сегодня в Москве?" → OFF_TOPIC
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/igotstatik)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/igotstatik)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
