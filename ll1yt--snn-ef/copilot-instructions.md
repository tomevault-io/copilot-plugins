## snn-ef

> - Исследовательский стек на Swift/Mac (Apple Silicon M‑серия, целевой M4), собираем гибрид из «обратимого текстового капсюля» и событийно‑поточной SNN‑маршрутизации энергии.

# Project Context

- Исследовательский стек на Swift/Mac (Apple Silicon M‑серия, целевой M4), собираем гибрид из «обратимого текстового капсюля» и событийно‑поточной SNN‑маршрутизации энергии.
- Два связанных модуля: `ReversibleCapsule` (строка ↔ фикс‑размерный блок ↔ энерговектор) и `EnergeticRouter` (FlowRouter: непрерывная плоскость R², LIF‑динамика, круговая проекция в B бинов). Общая инфраструктура — `SharedInfrastructure` (ConfigCenter, LoggingHub, ProcessRegistry, diagnostics).
- Центральный YAML‑конфиг описан в `Docs/config_center_schema.md`; ConfigCenter валидирует все параметры, снапшот immutable внутри шага. UI и CLI читают одни и те же значения (цветов нет, важна только наглядность).
- Визуализация опциональна (SwiftUI/Metal). Headless режим для CLI и тестов обязателен.
- Кодовая база придерживается современных возможностей Swift/Metal; векторизация и GPU‑ядра — приоритет на горячих участках.
- Тесты и сборку запускает пользователь вручную; агенты не дергают `swift build/test`.
- Headless mode is mandatory for CLI and tests.

# Working Principles

**Приоритет: вдумчивость → постепенность → эффективность**

- Сначала понять задачу и данные, потом действовать.
- Берём простейший рабочий вариант, покрываем тестами, оптимизируем после подтверждения.

**Основные правила**

- Модульность и чёткое разделение Capsule/Router/UI.
- Все конфигурации и логирование — централизованы (ConfigCenter + LoggingHub, `process_id` обязателен).
- Минимум церемоний, максимум продуктивности; прямолинейные решения важнее «сложных» абстракций(основной приоритет — производительность GPU).
- Используем современные языковые возможности Swift, но без усложнения.
- Проект исследовательский, не продакшен: нет миграций, нет временных костылей; при некорректных данных — fail fast.
- Без fallback, лучше явная ошибка.
- Максимально задействуем Apple Silicon GPU (Metal/MPS); избегаем CPU, если это мешает GPU‑производительности.
- Без CLI-флаг - реализуем это через центральный конфиг, если надо.
- PipelineSnapshotExporter пишет JSON снапшоты в `paths.pipeline_snapshot`; UI и CLI читают/создают их на лету, LoggingHub хранит timestamp последнего события по `process_id`.

High-level architecture and flow

- SharedInfrastructure (common to both modules)

  - ConfigCenter: single source of truth for config. Reads YAML, validates all keys, enforces invariants, and provides an immutable snapshot per step. CLI/UI use the same snapshot; changes require restart.
  - LoggingHub: centralized logging with levels (trace|debug|info|warn|error) и helper `emit(process:alias, ...)` — alias всегда переводится в canonical id. Поддерживает stdout/file destinations, относительное/абсолютное время и os_signpost.

- ProcessRegistry: canonical process_id dictionary for all pipeline stages (e.g., capsule.encode, router.step, router.spike, ui.pipeline, cli.main). ConfigCenter.merge обновляет mapping; `resolve` бросает ошибку на неизвестный alias.

- ReversibleCapsule

  - Goal: deterministic, reversible mapping string ↔ fixed-size byte block ↔ base‑B digits ↔ energies [1..B]. Header (len/flags/CRC32) + PRP (Feistel or AES via CryptoKit; optional GPU permutation) ensure reversibility.
  - Base‑B conversion yields fixed-length digit vectors D for a given block size N and base B; energies E = digits + 1 ∈ [1..B]. Optional normalization x = E/(B+1) for floating inputs to the router.
  - BridgeSNN adapter exposes makeEnergies(from: Capsule) and recoverCapsule(from: Energies) with CRC‑guarded round‑trip.

- EnergeticRouter (FlowRouter)

  - Continuous flows on R²; seeds из энергий капсулы раскладываются по кольцу. LIF‑мембрана, спайки дают рывки скорости/направления. Шум и радиальный дрейф задают органичную динамику.
  - Проекция на окружность радиуса R, угловая гистограмма в B бинов (B = capsule.base) — инвариант конфигурации.
  - CPU first (SIMD/Accelerate), затем GPU (Metal) для апдейта частиц и бининга.

- UI and headless modes
  - SwiftUI/Metal visualization subscribes to LoggingHub (process_id: ui.pipeline) to render real pipeline traces: string → capsule → digits → energies → router → decoded string. Headless mode can export the same trace to Artifacts/pipeline_snapshot.json.

Repository conventions (planned per docs)

- Configs: Configs/\*.yaml with a complete profile that covers logging, process_registry, paths, capsule, router, ui. The example baseline profile is in Docs/config_center_schema.md. CLI поддерживает `SNN_CONFIG_PATH` для указания временного конфига без правок репо.
- Artifacts: Logs/, Artifacts/Checkpoints/, Artifacts/Snapshots/, and Artifacts/pipeline_snapshot.json. ConfigCenter normalizes and creates paths on startup.

Key config invariants (from Docs/config_center_schema.md)

- router.energy_constraints.energy_base must equal capsule.base (bins = base).
- logging.levels_override keys must exist in process_registry.
- All numeric parameters are validated for ranges; unknown fields except notes and metadata are disallowed.
- UI headless overrides are explicit and logged under cli.main; pipeline_snapshot_path should match paths.pipeline_snapshot.

Working principles (for future changes)

- Priorities: thoughtfulness → gradualism → efficiency. Implement the simplest working variant, cover with tests, then optimize.
- Strict modular separation (Capsule/Router/UI) and centralized configuration/logging. Prefer GPU acceleration on Apple Silicon for hot paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LL1yt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/LL1yt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
