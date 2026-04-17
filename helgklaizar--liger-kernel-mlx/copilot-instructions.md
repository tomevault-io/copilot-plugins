## liger-kernel-mlx

> - **Engine**: Apple MLX (Metal)

# 🍏 Mac AI Ecosystem Initiative — Глобальный индекс

## Стек (Global Tech Stack)
- **Engine**: Apple MLX (Metal)
- **Optimizations**: 1-3 bit KV Cache (TurboQuant), FlashAttention-3
- **Languages**: Python, Metal Shading Language (MSL), Rust, C++

## 🗺 Карта проектов (Current Roadmap)

### 🔥 Титаны (Готовы / Active MVP)
- `turboquant-mlx` — Флагман: сжатие ОЗУ до 5 раз.
- `forge-mlx` — Фреймворк для оптимизации (Active).
- `liger-kernel-mlx` — Высокопроизводительные Metal кернелы.
- `flash-attention-mlx` — Ядерная база для FA3 на Metal.
- `paged-attention-mlx` — Управление KV-кешем.
- `cuda-bridge-mlx` — Мокинг CUDA под мак.

### 🛠 В разработке (Core Progress)
*(Все перенесены в Титаны)*

### 🧪 R&D Бэклог (KV-Cache и Attention Optimizations)
Экстремальные оптимизаторы механизмов внимания и работы с памятью:
- `evol-kv-mlx` — Эволюционный отбор токенов.
- `h2o-mlx` — Реализация алгоритма Heavy-Hitter Oracle.
- `kvtc-mlx` — Transform Coding для KV-кеша.
- `rocket-kv-mlx` — Реализация "ракетного" прунинга.
- `sage-attention-mlx` — Ускорение внимания (Scaffold+).
- `attention-matching-mlx` — R&D механизм внимания.
- `mla-mlx` — Multi-head Latent Attention (DeepSeek-style).
- `pyramid-kv-mlx` — Пирамидальное распределение KV-кэша.
- `tri-attention-mlx` — Pre-RoPE анализ для long-chain reasoning.
- `spotlight-attention-mlx` — Извлечение токенов через нелинейные хеш-функции.
- `mimo-flash-mlx` — Разреженно-глобально-локальное внимание.

## Ключевые решения
1. **Infrastructure**: Все служебные скрипты для управления вынесены в папку `/scripts`.
2. **Naming Standard**: Все проекты используют суффикс `*-mlx`. 
3. **Monorepo / Dependencies**: Пакетные менеджеры управляются глобально из корня:
   - **Python**: Используется `uv` и единое виртуальное окружение `.venv` в корне. Запрещено раздувать под-папки локальными копиями тяжелых библиотек (MLX, PyTorch).
   - **Rust**: В корне настроен Cargo `[workspace]` для объединения всех `target/` директорий и предотвращения дублирования кэшей компилятора.
4. **Apple Silicon Only**: Тюнинг под Metal Shading Language без поддержки CUDA.
5. **Концентрация**: Все проекты из групп Vision, Audio, RL удалены в угоду фокуса на Core и память.

## 📂 Текущие задачи (Active Backlog)

### 1. ⚙️ Инфраструктура и низкий уровень
- [x] **`flash-attention-mlx`**: Завершение трансляции CUDA FA во внутренние вызовы Metal.
- [x] **`paged-attention-mlx`**: Реализация менеджера страниц для PagedAttention под память M-чипов.
- [x] **`cuda-bridge-mlx`**: Расширение поддержки JIT-операций (Phase 2).
- [x] **`forge-mlx`**: Улучшение диспетчера модулей.

### 2. 💾 KV-Cache Optimization (The KV-Quartet)
Экстремальное сжатие контекста:
- [x] **`rocket-kv-mlx`**: Реализация "ракетного" прунинга.
- [x] **`kvtc-mlx`**: Transform Coding для KV-кеша.
- [x] **`evol-kv-mlx`**: Эволюционный отбор токенов.
- [x] **`h2o-mlx`**: Реализация алгоритма Heavy-Hitter Oracle.

### 3. 🧠 Механизмы Внимания (Attention R&D)
- [ ] **`attention-matching-mlx`**: Глубокое R&D по механизму Attention Matching (Phase 1).
- [ ] **`sage-attention-mlx`**: R&D алгоритмов Sage-Attention.

### 4. 🔮 Новые Архитектуры (2025-2026)
- [ ] **`mla-mlx`**: Реализация латентного пространства для KV.
- [ ] **`pyramid-kv-mlx`**: Настройка неравномерного бюджета слоев.
- [ ] **`tri-attention-mlx`**: Адаптация pre-RoPE пространства для M-Series.
- [ ] **`spotlight-attention-mlx`**: Имплементация hash-retrieval ядра.
- [ ] **`mimo-flash-mlx`**: Реализация чередования скользящего окна.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/helgklaizar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
