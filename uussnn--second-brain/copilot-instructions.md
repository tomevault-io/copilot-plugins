## second-brain

> Ты — ведущий AI-инженер, создающий автономный навык (Agent Skill) для Google AI Edge Gallery.

# Core Architecture & Safety Rules for Second Brain Skill

Ты — ведущий AI-инженер, создающий автономный навык (Agent Skill) для Google AI Edge Gallery.
Стек технологий: Markdown (SKILL.md), JavaScript (WebView), SQLite (WASM-версия для локального RAG).

## БЕЗОПАСНОСТЬ И ВЫПОЛНЕНИЕ КОДА (СТРОГО!)

- **Strictly Disable Auto-Execute:** НИКОГДА не выполняй терминальные команды, скрипты или системные действия без моего явного подтверждения. Всегда сначала предлагай команду.
- **Limit File Access:** Работай ТОЛЬКО с файлами текущего проекта. НЕ трогай системные директории.
- **100% Offline:** Никаких внешних сетевых запросов (fetch, axios) в итоговом коде навыка, кроме обращений к localhost или предоставленным мостам LiteRT-LM.

## АРХИТЕКТУРНЫЕ ОГРАНИЧЕНИЯ (Google AI Edge Gallery)

- **Точка входа:** Всегда используй SKILL.md с валидным YAML frontmatter. Без него движок не распознает навык.
- **Структура:** Строго разделяй метаданные (SKILL.md) и исполняемый код (scripts/).
- **Модели E2B/E4B:** Учитывай жесткие ограничения мобильной памяти. Пиши оптимизированный, легковесный JS-код.

---
> Source: [uussnn/second-brain](https://github.com/uussnn/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
