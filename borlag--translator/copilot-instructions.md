## translator

> Главный приоритет: снизить структурные ошибки (`placeholders_mismatch`, `style_tags_mismatch`) и нестабильность batch-режима локальных моделей, не ломая текущий pipeline.

# План внедрения полезных идей Gemini для `docxru` (OpenAI + локальные LLM)

## Краткое резюме
Главный приоритет: снизить структурные ошибки (`placeholders_mismatch`, `style_tags_mismatch`) и нестабильность batch-режима локальных моделей, не ломая текущий pipeline.  
По текущим артефактам есть явная точка роста: в `run.log` массовые `Batch response is not valid JSON`, а в `qa.jsonl` доминируют ошибки маркеров; в ABBYY-сэмплах много `w:framePr`/`w:trHeight` (жесткая верстка), но их нормализация сейчас не выполняется.

## Оценка предложений Gemini
1. Принять сразу: структурированный JSON-вывод для LLM, усиление проверки/ретраев маркеров, контекстные улучшения, fuzzy TM, ABBYY OXML-нормализация (опционально).
2. Принять с адаптацией: OCR line merging делать не через «склейку абзацев», а через контролируемое удаление искусственных soft-break токенов (`BRLINE`) по правилам.
3. Уже реализовано частично: соседний контекст (`prev/next`), run-консолидация, валидация тегов/плейсхолдеров, fallback после batch-ошибок.
4. Не внедрять в первой волне: shrink шрифта по `len()` и агрессивные типографские замены коротких предлогов; это высокий риск регрессии верстки/смысла.

## Изменения API/интерфейсов (decision-complete)
1. `src/docxru/config.py` (`LLMConfig`) добавить поля:
`structured_output_mode: off|auto|strict = auto`
`glossary_prompt_mode: off|full|matched = full`
`glossary_match_limit: int = 24`
`batch_skip_on_brline: bool = true`
`batch_max_style_tokens: int = 16`
2. `src/docxru/config.py` (`TMConfig`) добавить поля:
`fuzzy_enabled: bool = false`
`fuzzy_top_k: int = 3`
`fuzzy_min_similarity: float = 0.75`
`fuzzy_prompt_max_chars: int = 500`
3. `src/docxru/config.py` (`PipelineConfig`) добавить поля:
`abbyy_profile: off|safe|aggressive = off`
`glossary_lemma_check: off|warn|retry = off`
4. `src/docxru/cli.py` добавить флаги:
`--structured-output`, `--glossary-prompt-mode`, `--fuzzy-tm`, `--abbyy-profile`.
5. Формат LLM-ответа:
для single translate: `{"translated_text":"..."}`
для repair: `{"repaired_text":"..."}`
для batch: текущий `{"translations":[{"id":"...","text":"..."}]}`.

## План внедрения

## Этап 0. Базовая метрика и контроль регрессий
1. Добавить агрегатор QA-кодов и fallback-метрик в лог (`placeholders_mismatch`, `style_tags_mismatch`, `batch_fallback_single`, `batch_validation_fallback`).
2. Зафиксировать baseline на одном и том же наборе:
`samples/...abby_short_2.docx` c OpenAI-профилем и Ollama-профилем.
3. Критерий входа в следующий этап: baseline сохранен в `docs/benchmark_baseline.md`.

## Этап 1. Надежность маркеров и parity API/local (главный)
1. `src/docxru/llm.py`: включить JSON-contract для OpenAI и Ollama в задачах `translate`, `repair`, `batch_translate` с парсером JSON+fallback.
2. `src/docxru/pipeline.py`: добавить фильтр batch-eligibility.
Сегменты с `BRLINE`, большим числом style-токенов или высоким числом плейсхолдеров переводить только single-pass.
3. `src/docxru/pipeline.py`: добавить controlled soft-break collapse для ABBYY.
При `abbyy_profile in {safe,aggressive}` удалять искусственные `BRLINE` в «текстовых» сегментах, но не в TOC-подобных/табличных структурных строках.
4. `src/docxru/pipeline.py`: сохранить текущую repair-логику, но добавить отдельный issue-code при JSON-нарушении схемы для точной диагностики.

## Этап 2. Глоссарий без ломки морфологии
1. `src/docxru/llm.py`: режим `glossary_prompt_mode=matched`.
В prompt вставлять только термины, найденные в текущем сегменте, с лимитом `glossary_match_limit`.
2. `src/docxru/pipeline.py`: вычислять matched-термины на этапе подготовки сегмента и класть в `context`.
3. `src/docxru/validator.py`: добавить опциональную проверку лемм (`glossary_lemma_check`) через `pymorphy3` только при наличии библиотеки.
4. Политика retry:
если `glossary_lemma_check=retry` и провайдер поддерживает repair, делать 1 дополнительный rewrite с явным напоминанием о термине.

## Этап 3. Fuzzy TM (для всех prompt-based провайдеров)
1. `src/docxru/tm.py`: добавить fuzzy lookup поверх SQLite (FTS5 + similarity score).
2. `src/docxru/pipeline.py`: при промахе exact-TM подмешивать top-K примеров в контекст.
3. `src/docxru/llm.py`: в user prompt добавить компактный блок `TM_REFERENCES`.
4. Ограничения:
если fuzzy отключен или FTS недоступен, поведение полностью прежнее (exact-only).

## Этап 4. ABBYY OXML-нормализация (опционально, по профилю)
1. `src/docxru/oxml_table_fix.py` расширить:
функция снятия `w:trHeight` с `w:hRule="exact"` (safe).
функция нормализации `w:framePr` (только aggressive).
2. `src/docxru/pipeline.py`: вызывать OXML-нормализацию перед сохранением при `abbyy_profile != off`.
3. Логировать количественно: сколько `trHeight exact` снято, сколько `framePr` изменено.

## Этап 5. Документация и профили
1. `README.md` и `config/*.yaml` обновить под новые режимы.
2. Для `config/config.agent_openai.yaml` и `config/config.ollama.hy-mt.yaml` рекомендовать:
`structured_output_mode: strict`
`glossary_prompt_mode: matched`
`batch_skip_on_brline: true`
`abbyy_profile: safe` только для ABBYY-конвертированных файлов.

## Тесты и сценарии

## Unit tests
1. `tests/test_llm_clients.py`:
JSON-ответы OpenAI/Ollama для single/repair/batch.
fallback на plain text в `auto`.
ошибка в `strict` при невалидном JSON.
2. `tests/test_batch_mode.py`:
новый eligibility-фильтр batch.
3. `tests/test_pipeline_tm_profile.py`:
новые поля конфигурации входят в TM profile key.
4. Новый `tests/test_glossary_matched.py`:
корректный выбор matched терминов.
5. Новый `tests/test_tm_fuzzy.py`:
ранжирование fuzzy-кандидатов и порог `fuzzy_min_similarity`.
6. Новый `tests/test_oxml_table_fix.py`:
снятие `hRule=exact` и профильное поведение `safe/aggressive`.

## Integration tests
1. `samples/...abby_short_2.docx` + Ollama profile:
проверка снижения batch JSON-fallback и marker-ошибок.
2. `samples/...part1_Short.docx` + OpenAI profile:
проверка отсутствия регрессии по QA-кодам и структуре.
3. `docxru verify`:
нулевая утечка маркеров `⟦...⟧` в финальном документе.

## Acceptance criteria
1. На фиксированном ABBYY-сэмпле:
`batch_fallback_single` снизить минимум на 70%.
`placeholders_mismatch` снизить минимум на 40%.
`style_tags_mismatch` снизить минимум на 40%.
2. Для OpenAI и Ollama:
нет роста `writeback_skipped_due_to_errors` относительно baseline.
3. При `abbyy_profile=off` поведение обратно-совместимо.

## Предположения и выбранные дефолты
1. Приоритет принят: сначала надежность маркеров и стабильность batch/local.
2. Новые зависимости (pymorphy3/rapidfuzz/доп. NLP) подключаются только опционально и за флагами.
3. ABBYY-нормализация включается только явно (`abbyy_profile`), по умолчанию выключена.
4. Автоматический shrink шрифта по длине строки в первую волну не входит.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Borlag)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Borlag)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
