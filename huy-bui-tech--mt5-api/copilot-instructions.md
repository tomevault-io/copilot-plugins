## mt5-api

> <skills_system priority="1">

# AGENTS

<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively. Skills provide specialized capabilities and domain knowledge.

How to use skills:
- Invoke: Bash("openskills read <skill-name>")
- The skill content will load with detailed instructions on how to complete the task
- Base directory provided in output for resolving bundled resources (references/, scripts/, assets/)

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
- Each skill invocation is stateless
</usage>

<available_skills>

<skill>
<name>writing-plans</name>
<description>Create detailed implementation plans with bite-sized tasks for engineers with zero codebase context</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>
@../../ai/systems/SOFT-DEV-PRIN.md

# Agent Memory & Knowledge Strategy

You are an AI Agent equipped with two types of memory. You MUST follow these rules whenever a user provides a piece of information that should be "remembered."

## 1. Memory Storage Options
When the user shares a fact, preference, or architectural decision, you must categorize it:

### Option A: Local Project Context (`AGENTS.md`)
* **Use for:** Project-specific technical specs, todo lists, active bugs, or folder structures.
* **Scope:** This project only.
* **Action:** Append the info to the "Project Log" section at the bottom of this file.

---

## 3. Project Log (Automated Section)

### 2025-12-27: Python Server Implementation Complete

**Architecture**: Python (TCP Server) ← → MT5 EA (TCP Client)
- MQL5 lacks server sockets, so Python must be the server

**Structure**:
```
mt5_bridge/
├── protocol/       # JSON message models & parsing
├── server/         # TCP server & connection handling
├── trading/        # MT5Bridge high-level API
└── main.py         # Entry point
```

**API Methods**:
- `buy()`, `sell()`, `place_pending_order()`
- `get_rates()`, `get_tick()`
- `get_account()`, `get_positions()`
- `close_position()`, `modify_position()`

**Next**: Implement MT5 EA (MQL5 client)

### 2025-12-27: MT5 EA Implementation Complete

**File**: `mql5/Experts/MT5SocketClient.mq5`

**Features**:
- Socket client connecting to Python server
- OnTimer-based polling (100ms interval)
- JSON message parsing and response building
- All commands: TRADE, GET_DATA, GET_TICK, GET_ACCOUNT, GET_POSITIONS, CLOSE_POSITION, HEARTBEAT

**Setup on MT5**:
1. Copy EA to `MQL5/Experts/` folder
2. Allow WebRequest for Python server IP in Tools > Options > Expert Advisors
3. Attach EA to any chart, configure ServerAddress and ServerPort

**Status**: Ready for testing

### 2025-12-27: Winsock DLL Migration & Testing Complete

**Issue**: MQL5 native socket functions unreliable
**Solution**: Migrated to Windows Winsock DLL (`ws2_32.dll`)

**Key imports**:
- `WSAStartup`, `WSACleanup`, `WSAGetLastError`
- `socket`, `connect`, `send`, `recv`, `closesocket`
- `ioctlsocket` (non-blocking mode)

**Testing Results**:
- BTCUSD BUY 0.01 lots - SUCCESS
- CLOSE_POSITION - SUCCESS ($0.12 profit)

**All commands verified working**:
- TRADE (BUY/SELL)
- GET_TICK, GET_DATA, GET_ACCOUNT, GET_POSITIONS
- CLOSE_POSITION

---

### 2025-12-27: Order Panel UI Implementation Complete

**Framework**: Tkinter (standard library)

**Structure**:
```
mt5_bridge/ui/
├── __init__.py       # Module exports
├── config.py         # PanelConfig, Colors dataclasses
├── async_bridge.py   # AsyncBridge for Tkinter-asyncio integration
├── widgets.py        # TradeButton, StatusBar, SymbolSelector, VolumeInput
└── order_panel.py    # Main OrderPanel class

run_panel.py          # Standalone runner
```

**Features**:
- B/S/C buttons with keyboard shortcuts (B=Buy, S=Sell, C=Close, Esc=Quit)
- Symbol dropdown (BTCUSD, EURUSD, GBPUSD, USDJPY, XAUUSD)
- Volume input with +/- buttons
- Visual feedback (hover, active, success/error flash)
- Status bar showing connection and last action
- Thread-safe async operations via queue

**Usage**:
```bash
python run_panel.py --symbol BTCUSD --volume 0.01
```

**Status**: Ready for testing

---

### 2025-12-28: Execution Speed Measurement Complete

**Approach**: Embedded EA timestamps + round-trip calculation

**Timestamp points**:
- T1: Python sends command
- T2: EA receives (`GetMicrosecondCount()`)
- T3: EA executes trade
- T4: Python receives response

**Metrics**:
- Round-trip latency (T4-T1)
- EA processing time (T3-T2)
- Network estimate ((T4-T1)-(T3-T2))/2

**Structure**:
```
mt5_bridge/timing/
├── __init__.py    # Module exports
├── models.py      # TimingData, LatencyStats, TimingReport
├── collector.py   # TimingCollector class
├── analyzer.py    # LatencyAnalyzer with stats (avg, P50, P95, P99)
└── reporter.py    # ConsoleReporter, CSVReporter

test_latency.py    # Benchmark script
```

**EA modifications**:
- Added `g_t2_receive` global variable
- Capture timestamp in `HandleMessage()` using `GetMicrosecondCount()`
- Added `timing` object in JSON response from `BuildSuccessResponse()`

**Bridge integration**:
- Added `enable_timing` flag to MT5Bridge constructor
- Methods: `get_timing_samples()`, `get_timing_report()`, `get_timing_report_by_action()`, `clear_timing_data()`

**Usage**:
```bash
python test_latency.py --iterations 50 --symbol BTCUSD --output latency.csv
```

**Benchmark Results** (10ms polling interval):

| Metric | Min | Avg | P50 | P95 | P99 | Max |
|--------|-----|-----|-----|-----|-----|-----|
| Round-trip | 1.06ms | 13.64ms | 17.36ms | 22.64ms | 26.41ms | 26.41ms |
| EA Processing | 0.01ms | 0.02ms | 0.02ms | 0.04ms | 0.06ms | 0.06ms |
| Network (est.) | 0.52ms | 6.81ms | 8.67ms | 11.31ms | 13.20ms | 13.20ms |

**Polling Interval Comparison**:

| Interval | Avg Latency | Improvement |
|----------|-------------|-------------|
| 100ms | 99.88ms | baseline |
| 10ms | 13.64ms | 7.3x faster |

**Status**: Complete

---

### 2025-12-28: UI Improvements - Phase 2

**Goals**:
1. Position modify dialog (replace inline editing)
2. Spread display in dollar amount
3. Symbol autocomplete from tradeable pairs

#### Task 1: Position Modify Dialog
**Status**: Complete
**Files**: `mt5_bridge/ui/widgets.py`
**Description**: Add "Edit" button on position row that opens a modal dialog for modifying SL/TP/trailing stop. This replaces inline editing which is difficult due to auto-refresh.

#### Task 2: Spread Display
**Status**: Complete
**Files**: `mt5_bridge/ui/widgets.py`, `mt5_bridge/ui/order_panel.py`
**Description**: Display spread cost in dollar amount (spread * volume * pip_value) near the volume input area.

#### Task 3a: GET_SYMBOLS EA Command
**Status**: Complete
**Files**: `mql5/Experts/MT5SocketClient.mq5`
**Description**: Add GET_SYMBOLS handler to EA using SymbolsTotal() and SymbolName() to return all tradeable symbols.

#### Task 3b: Python Bridge get_symbols
**Status**: Complete
**Files**: `mt5_bridge/protocol/models.py`, `mt5_bridge/trading/bridge.py`
**Description**: Add GET_SYMBOLS action and get_symbols() method to Python bridge.

#### Task 3c: Symbol Autocomplete
**Status**: Complete
**Files**: `mt5_bridge/ui/widgets.py`, `mt5_bridge/ui/order_panel.py`
**Description**: Replace symbol dropdown with searchable autocomplete that filters tradeable pairs from account.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huy-bui-tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
