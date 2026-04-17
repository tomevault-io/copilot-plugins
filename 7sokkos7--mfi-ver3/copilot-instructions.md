## mfi-ver3

> PROJECT: MFI_VER3 — MasterForex-V Modular Indicator (MetaTrader 5)

PROJECT: MFI_VER3 — MasterForex-V Modular Indicator (MetaTrader 5)

MODE: DEVELOPMENT • IDEMPOTENT • MINIMAL CHANGES • LOCAL-ONLY
- Источник истины: ТОЛЬКО локальный workspace. Никакой сети, git, API.
- Перед действиями ПРОЧИТАЙ текущее состояние файлов и README.md.
- Если пункт уже выполнен — SKIPPED. Изменять минимум необходимого.
- Никаких «улучшений по вдохновению».

ALLOWED FILETYPES & AREAS
- Разрешены только .mq5 и .mqh.
- Разрешено изменять: README.md и файлы в indicator/** (кроме indicator/ThirdParty/**).
- Создавать НОВЫЕ файлы можно ТОЛЬКО в indicator/include/** и ТОЛЬКО .mqh/.mq5, если это явно указано в текущем задании.

FORBIDDEN
- Любые C/C++/Python/скрипты, импорт DLL, `#import`, внешние API, iCustom-вызовы ZigZag.
- `#pragma once` в .mqh — запрещено. Использовать include-гварды `#ifndef/#define/#endif`.
- Создание файлов вне явного списка из текущего задания.
- Любые правки внутри data/** (CSV только читаем).

MQL5 BASELINE (должно быть всегда)
- `#property strict`
- Версия: `#property version "1.000"` (формат `xxx.yyy`). Если уже валидна — SKIPPED.
- В `MFI_Modular.mq5` минимум 1 plot и 1 buffer (можно `DRAW_NONE`), без предметной логики.
- Порядок include в `MFI_Modular.mq5`:
  Types → Config → State → Utils → MarketData → ZigZagAdapter → PivotEngine → TrendEngine → Breakout → Filters → Signals → Draw → Panel → Logger
- Массивы в сигнатурах функций — ТОЛЬКО по ссылке: `type &name[]` (например, `int &idx[]`, `double &price[]`).

ARCH CONTRACT
- Главный файл `indicator/MFI_Modular.mq5` — только «проводка» и вызов оркестратора `MFV_UpdateAll(...)`.
- Оркестратор и бизнес-логика живут в модулях под `indicator/include/core/*` и `features/*`.
- Вся графика — только в `Draw.mqh`. Панель — только в `Panel.mqh`. Никакой графики в ядре и оркестраторе.

ZIGZAG POLICY (ОДИН внутренний)
- Единственная реализация ZigZag: `indicator/include/core/ZigZagInternal.mqh`.
- `indicator/include/core/ZigZagAdapter.mqh` — ТОЛЬКО включает `ZigZagInternal.mqh`. НЕ объявлять здесь прототипы без тела и НЕ писать вторую реализацию.
- Запрещены любые `iCustom(..., "*/ZigZag", ...)`, `#property tester_indicator` и внешние зависимости ZigZag.
- Если существует indicator/ThirdParty/ZigZag.* — игнорировать (не использовать). Удаление возможно ТОЛЬКО по явному заданию.

ZIGZAG MF PRESET (входные параметры и использование)
- В `indicator/include/core/Config.mqh` поддерживать входы:

  input bool ZZ_UseMF_Preset = true;  // true: использовать пресет MF по ТФ

  input int ZZ_MF_M5_Depth = 12;   input int ZZ_MF_M5_Deviation = 5;   input int ZZ_MF_M5_Backstep = 3;
  input int ZZ_MF_M15_Depth = 12;  input int ZZ_MF_M15_Deviation = 5;  input int ZZ_MF_M15_Backstep = 3;
  input int ZZ_MF_H1_Depth = 24;   input int ZZ_MF_H1_Deviation = 5;   input int ZZ_MF_H1_Backstep = 3;
  input int ZZ_MF_H4_Depth = 48;   input int ZZ_MF_H4_Deviation = 5;   input int ZZ_MF_H4_Backstep = 3;
  input int ZZ_MF_D1_Depth = 60;   input int ZZ_MF_D1_Deviation = 5;   input int ZZ_MF_D1_Backstep = 3;

  // Ручной режим (если нужно уйти от пресета MF):
  input int ZZ_Manual_Depth = 12, ZZ_Manual_Deviation = 5, ZZ_Manual_Backstep = 3;

- В `Utils.mqh` должна быть утилита подстановки:

  void MFV_ZZ_ParamsForTF(const ENUM_TIMEFRAMES tf, int &depth, int &deviation, int &backstep)
  {
    if(!ZZ_UseMF_Preset){ depth=ZZ_Manual_Depth; deviation=ZZ_Manual_Deviation; backstep=ZZ_Manual_Backstep; return; }
    switch(tf){
      case PERIOD_M5:  depth=ZZ_MF_M5_Depth;  deviation=ZZ_MF_M5_Deviation;  backstep=ZZ_MF_M5_Backstep;  break;
      case PERIOD_M15: depth=ZZ_MF_M15_Depth; deviation=ZZ_MF_M15_Deviation; backstep=ZZ_MF_M15_Backstep; break;
      case PERIOD_H1:  depth=ZZ_MF_H1_Depth;  deviation=ZZ_MF_H1_Deviation;  backstep=ZZ_MF_H1_Backstep;  break;
      case PERIOD_H4:  depth=ZZ_MF_H4_Depth;  deviation=ZZ_MF_H4_Deviation;  backstep=ZZ_MF_H4_Backstep;  break;
      case PERIOD_D1:  depth=ZZ_MF_D1_Depth;  deviation=ZZ_MF_D1_Deviation;  backstep=ZZ_MF_D1_Backstep;  break;
      default:         depth=ZZ_Manual_Depth; deviation=ZZ_Manual_Deviation; backstep=ZZ_Manual_Backstep;  break;
    }
  }

- `ZigZagInternal.mqh`: функция

  bool MFV_ZZ_FindExtrema(const string symbol, ENUM_TIMEFRAMES tf,
                          int depth, int deviation, int backstep,
                          int &outCount, int &outIdx[], double &outPrice[]);

  Массивы — по ссылке. Никаких import.

- В `Signals.mqh` перед вызовом ZigZag подставлять MF-параметры:

  int d,v,b; MFV_ZZ_ParamsForTF(chart_tf, d, v, b);
  int cnt=0; int idx[512]; double prc[512];
  MFV_ZZ_FindExtrema(symbol, chart_tf, d, v, b, cnt, idx, prc);

README EXPECTATIONS (поддерживать)
- Дерево проекта с модульной структурой.
- Жёсткие пункты: «Без `#pragma once`», «Dummy-plot 1/1 DRAW_NONE», «Версия `xxx.yyy`».
- Раздел «ZigZag — внутренний модуль» и «Пресет MF для ZigZag» (без слов “MasterForex”, только “MF” в формулировках параметров).

OUTPUT FORMAT (каждое задание)
- Таблица статусов: OK/CHANGED/SKIPPED по пунктам.
- Список изменённых файлов и минимальные патчи (только реально изменённые места).
- Тексты обновлённых ключевых файлов, если менялись (MFI_Modular.mq5, Signals.mqh, ZigZagInternal.mqh, ZigZagAdapter.mqh).
- Лог компиляции (0 ошибок; 0 предупреждений по версии/plots/импортам).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/7SoKKoS7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
