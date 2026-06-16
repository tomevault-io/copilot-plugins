## claude-trading-bot

> 68 tools for reading and controlling a live TradingView Desktop chart via CDP (port 9222).

# TradingView MCP — Claude Instructions

68 tools for reading and controlling a live TradingView Desktop chart via CDP (port 9222).

## Decision Tree — Which Tool When

### "What's on my chart right now?"
1. `chart_get_state` → symbol, timeframe, chart type, list of all indicators with entity IDs
2. `data_get_study_values` → current numeric values from all visible indicators (RSI, MACD, BBands, EMAs, etc.)
3. `quote_get` → real-time price, OHLC, volume for current symbol

### "What levels/lines/labels are showing?"
Custom Pine indicators draw with `line.new()`, `label.new()`, `table.new()`, `box.new()`. These are invisible to normal data tools. Use:

1. `data_get_pine_lines` → horizontal price levels drawn by indicators (deduplicated, sorted high→low)
2. `data_get_pine_labels` → text annotations with prices (e.g., "PDH 24550", "Bias Long ✓")
3. `data_get_pine_tables` → table data formatted as rows (e.g., session stats, analytics dashboards)
4. `data_get_pine_boxes` → price zones / ranges as {high, low} pairs

Use `study_filter` parameter to target a specific indicator by name substring (e.g., `study_filter: "Profiler"`).

### "Give me price data"
- `data_get_ohlcv` with `summary: true` → compact stats (high, low, range, change%, avg volume, last 5 bars)
- `data_get_ohlcv` without summary → all bars (use `count` to limit, default 100)
- `quote_get` → single latest price snapshot

### "Analyze my chart" (full report workflow)
1. `quote_get` → current price
2. `data_get_study_values` → all indicator readings
3. `data_get_pine_lines` → key price levels from custom indicators
4. `data_get_pine_labels` → labeled levels with context (e.g., "Settlement", "ASN O/U")
5. `data_get_pine_tables` → session stats, analytics tables
6. `data_get_ohlcv` with `summary: true` → price action summary
7. `capture_screenshot` → visual confirmation

### "Change the chart"
- `chart_set_symbol` → switch ticker (e.g., "AAPL", "ES1!", "NYMEX:CL1!")
- `chart_set_timeframe` → switch resolution (e.g., "1", "5", "15", "60", "D", "W")
- `chart_set_type` → switch chart style (Candles, HeikinAshi, Line, Area, Renko, etc.)
- `chart_manage_indicator` → add or remove studies (use full name: "Relative Strength Index", not "RSI")
- `chart_scroll_to_date` → jump to a date (ISO format: "2025-01-15")
- `chart_set_visible_range` → zoom to exact date range (unix timestamps)

### "Work on Pine Script"
1. `pine_set_source` → inject code into editor
2. `pine_smart_compile` → compile with auto-detection + error check
3. `pine_get_errors` → read compilation errors
4. `pine_get_console` → read log.info() output
5. `pine_get_source` → read current code back (WARNING: can be very large for complex scripts)
6. `pine_save` → save to TradingView cloud
7. `pine_new` → create blank indicator/strategy/library
8. `pine_open` → load a saved script by name

### "Practice trading with replay"
1. `replay_start` with `date: "2025-03-01"` → enter replay mode
2. `replay_step` → advance one bar
3. `replay_autoplay` → auto-advance (set speed with `speed` param in ms)
4. `replay_trade` with `action: "buy"/"sell"/"close"` → execute trades
5. `replay_status` → check position, P&L, current date
6. `replay_stop` → return to realtime

### "Screen multiple symbols"
- `batch_run` with `symbols: ["ES1!", "NQ1!", "YM1!"]` and `action: "screenshot"` or `"get_ohlcv"`

### "Draw on the chart"
- `draw_shape` → horizontal_line, trend_line, rectangle, text (pass point + optional point2)
- `draw_list` → see what's drawn
- `draw_remove_one` → remove by ID
- `draw_clear` → remove all

### "Manage alerts"
- `alert_create` → set price alert (condition: "crossing", "greater_than", "less_than")
- `alert_list` → view active alerts
- `alert_delete` → remove alerts

### "Navigate the UI"
- `ui_open_panel` → open/close pine-editor, strategy-tester, watchlist, alerts, trading
- `ui_click` → click buttons by aria-label, text, or data-name
- `layout_switch` → load a saved layout by name
- `ui_fullscreen` → toggle fullscreen
- `capture_screenshot` → take a screenshot (regions: "full", "chart", "strategy_tester")

### "TradingView isn't running"
- `tv_launch` → auto-detect and launch TradingView with CDP on Mac/Win/Linux
- `tv_health_check` → verify connection is working

## Context Management Rules

These tools can return large payloads. Follow these rules to avoid context bloat:

1. **Always use `summary: true` on `data_get_ohlcv`** unless you specifically need individual bars
2. **Always use `study_filter`** on pine tools when you know which indicator you want — don't scan all studies unnecessarily
3. **Never use `verbose: true`** on pine tools unless the user specifically asks for raw drawing data with IDs/colors
4. **Avoid calling `pine_get_source`** on complex scripts — it can return 200KB+. Only read if you need to edit the code.
5. **Avoid calling `data_get_indicator`** on protected/encrypted indicators — their inputs are encoded blobs. Use `data_get_study_values` instead for current values.
6. **Use `capture_screenshot`** for visual context instead of pulling large datasets — a screenshot is ~300KB but gives you the full visual picture
7. **Call `chart_get_state` once** at the start to get entity IDs, then reference them — don't re-call repeatedly
8. **Cap your OHLCV requests** — `count: 20` for quick analysis, `count: 100` for deeper work, `count: 500` only when specifically needed

### Output Size Estimates (compact mode)
| Tool | Typical Output |
|------|---------------|
| `quote_get` | ~200 bytes |
| `data_get_study_values` | ~500 bytes (all indicators) |
| `data_get_pine_lines` | ~1-3 KB per study (deduplicated levels) |
| `data_get_pine_labels` | ~2-5 KB per study (capped at 50) |
| `data_get_pine_tables` | ~1-4 KB per study (formatted rows) |
| `data_get_pine_boxes` | ~1-2 KB per study (deduplicated zones) |
| `data_get_ohlcv` (summary) | ~500 bytes |
| `data_get_ohlcv` (100 bars) | ~8 KB |
| `capture_screenshot` | ~300 bytes (returns file path, not image data) |

## Tool Conventions

- All tools return `{ success: true/false, ... }`
- Entity IDs (from `chart_get_state`) are session-specific — don't cache across sessions
- Pine indicators must be **visible** on chart for pine graphics tools to read their data
- `chart_manage_indicator` requires **full indicator names**: "Relative Strength Index" not "RSI", "Moving Average Exponential" not "EMA", "Bollinger Bands" not "BB"
- Screenshots save to `screenshots/` directory with timestamps
- OHLCV capped at 500 bars, trades at 20 per request
- Pine labels capped at 50 per study by default (pass `max_labels` to override)

## Architecture

```
Claude Code ←→ MCP Server (stdio) ←→ CDP (localhost:9222) ←→ TradingView Desktop (Electron)
```

Pine graphics path: `study._graphics._primitivesCollection.dwglines.get('lines').get(false)._primitivesDataById`

## 12 Hr Update — Custom Watchlist Scanner

When the user asks for a "12 hr update" or "twelve hour update", follow this workflow:

### Step-by-Step
1. Read `rules.json` in the project root to get the watchlist symbols, timeframes, and bias criteria
2. For each symbol in `watchlist`:
   a. `chart_set_symbol` to switch to that symbol
   b. For each timeframe in `timeframes` (e.g., "W", "D", "240"):
      - `chart_set_timeframe` to switch
      - `quote_get` for current price
      - `data_get_study_values` for all indicator readings
      - `data_get_ohlcv` with `summary: true` for price stats
      - `data_get_pine_lines` for support/resistance levels
   c. Apply `biasCriteria` from rules.json to determine: Bullish / Bearish / Neutral
3. Compile all results into a structured report
4. Save to `~/.tradingview-mcp/sessions/` as `YYYY-MM-DD_HH-mm.json`
5. Display the formatted report to the user

### Report Format
Each asset should show: Symbol | Bias | Price | Key Level | Per-Timeframe Breakdown | Indicator Values | Watch Points

### Summary Section
End with a summary grouping assets by Bullish/Bearish/Neutral, plus any risk checklist items from `rules.json.riskRules`

## Telegram Trade Alerts

Send trade alerts to Telegram using the alert system in `alerts/telegram.cjs`.

### How to Send Alerts

All alerts are sent via Node.js CLI:

```bash
# Trade entry
node ~/tradingview-mcp/alerts/telegram.cjs entry '<json>'

# Stop loss hit
node ~/tradingview-mcp/alerts/telegram.cjs sl '<json>'

# Take profit hit
node ~/tradingview-mcp/alerts/telegram.cjs tp '<json>'

# Trade exit
node ~/tradingview-mcp/alerts/telegram.cjs exit '<json>'

# 12hr update
node ~/tradingview-mcp/alerts/telegram.cjs update '<json>'

# Chart screenshot
node ~/tradingview-mcp/alerts/telegram.cjs photo '/path/to/screenshot.png'
```

### When to Send Alerts

1. **Entry Alert** — When a trade setup triggers based on the strategy:
   - Include: direction, symbol, entry, SL, TP1/TP2/TP3, timeframe, R:R
   - Include WHY (the reason for the trade)
   - Include CONFLUENCE (list of confirming factors)
   - Take a screenshot and attach it

2. **TP Alert** — When price hits a take-profit level:
   - Include which TP was hit and partial close %

3. **SL Alert** — When stop loss is hit:
   - Include what went wrong

4. **Exit Alert** — When closing a trade manually:
   - Include P&L and reason for exit

5. **12hr Update** — After running the 12hr scan:
   - Send the bias summary to Telegram

### Entry Alert JSON Format
```json
{
  "direction": "SHORT",
  "symbol": "GOLD",
  "entry": "4808",
  "sl": "4822",
  "tp1": "4741",
  "tp2": "4718",
  "tp3": "4611",
  "timeframe": "5min entry / 15min bias",
  "riskReward": "4.8:1",
  "reason": "Why we are taking this trade — structure, indicator confirmation, zone reaction",
  "confluence": [
    "Factor 1 — e.g. below Smart Trail",
    "Factor 2 — e.g. CHoCH+ bearish on 4H",
    "Factor 3 — e.g. LuxAlgo signal fired",
    "Factor 4 — e.g. rejection at EQ zone"
  ],
  "screenshot": "/path/to/chart_screenshot.png"
}
```

### Intraday Execution Model
- **15min** sets the bias and marks structure/zones
- **5min** provides sniper entries within 15min zones
- Wait for LuxAlgo signal confirmation before entering
- SL goes behind 15min structure, not 5min swings

## MT5 Trade Execution

Execute trades on MetaTrader 5 via the file-based bridge (`alerts/mt5.cjs` → `signal.json` → TV.mq5 EA).

### Requirements
- MT5 must be open with XAUUSD chart
- TV EA must be attached to the chart with AutoTrading enabled
- EA secret must match: `test123`

### Commands

```bash
# Market buy with price-based SL/TP (PREFERRED)
node ~/tradingview-mcp/alerts/mt5.cjs buy '{"symbol":"XAUUSD","lot":0.01,"sl_price":4773,"tp_price":4841}'

# Market sell with price-based SL/TP
node ~/tradingview-mcp/alerts/mt5.cjs sell '{"symbol":"XAUUSD","lot":0.01,"sl_price":4835,"tp_price":4718}'

# Close all positions on symbol
node ~/tradingview-mcp/alerts/mt5.cjs close '{"symbol":"XAUUSD","reason":"TP2 hit"}'

# Check bridge status
node ~/tradingview-mcp/alerts/mt5.cjs status
node ~/tradingview-mcp/alerts/mt5.cjs check
```

### Signal JSON format (written to MQL5/Files/signal.json)

**Entry (price-based SL/TP — preferred):**
```json
{"secret":"test123","symbol":"XAUUSD","event":"ENTRY","direction":"SELL","lot":0.01,"sl_price":4835,"tp_price":4718,"id":"claude-xxx"}
```

**Entry (point-based SL/TP — legacy):**
```json
{"secret":"test123","symbol":"XAUUSD","event":"ENTRY","direction":"BUY","lot":0.01,"sl_points":5000,"tp_points":10000,"id":"claude-xxx"}
```

**Close all:**
```json
{"secret":"test123","symbol":"XAUUSD","event":"CLOSE","id":"claude-xxx"}
```

**Partial close (e.g., 40% at TP1):**
```json
{"secret":"test123","symbol":"XAUUSD","event":"PARTIAL_CLOSE","percent":40,"id":"claude-xxx"}
```

**Modify SL/TP (e.g., move SL to breakeven):**
```json
{"secret":"test123","symbol":"XAUUSD","event":"MODIFY","sl_price":4808,"tp_price":4611,"id":"claude-xxx"}
```

### Trade Management Workflow
1. Entry → sends signal, waits for ack, sends Telegram alert + screenshot
2. TP1 hit → PARTIAL_CLOSE 40% + MODIFY SL to breakeven + Telegram TP alert
3. TP2 hit → PARTIAL_CLOSE 50% of remaining + Telegram TP alert
4. TP3 / SL → CLOSE all + Telegram exit alert

### Broker Info
- Server: MaziMaticFinancial-Server
- Symbol on broker: `XAUUSD.o` (EA normalizes automatically)
- Account: Demo #894258

---
> Source: [Bodasaieswar/claude-trading-bot](https://github.com/Bodasaieswar/claude-trading-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
