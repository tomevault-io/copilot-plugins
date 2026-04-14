## tradingview

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a TradingView Pine Script repository for technical analysis indicators and screeners. Pine Script is TradingView's domain-specific language for creating custom technical indicators, strategies, and studies.

## Architecture

- `indicators/` - Contains all technical analysis indicators
  - `movings/` - Moving average indicators and related technical analysis scripts
  - `relative_strength/` - Relative strength indicators
  - `volumes/` - Volume-based indicators
- `screeners/` - Contains market screening scripts for filtering stocks/assets
- Pine Script files use `.pine` extension and follow Pine Script v6 syntax

## Pine Script Development

Pine Script files are text-based indicators that run on TradingView's platform. Key characteristics:
- Each script starts with `//@version=6` directive
- Use `indicator()` function to define script metadata
- Technical analysis functions are available under `ta.` namespace (e.g., `ta.sma()` for simple moving average)
- Plotting functions use `plot()` with customizable colors and line styles
- **All comments must be written in English only** - never use Japanese or other languages in code comments

## Code Structure Patterns

Pine Script indicators in this repository follow this pattern:
1. License header with MPL 2.0 reference
2. Version directive and indicator declaration
3. Calculate technical analysis values using `ta.*` functions
4. Plot results with descriptive titles and color coding

## Color Scheme Convention

Moving average indicators use consistent color coding:
- SMA 10: Cyan (`color.rgb(0, 255, 255)`)
- SMA 21: Pink (`color.rgb(255, 192, 203)`)
- SMA 50: Blue (`color.blue`)
- SMA 200: Red (`color.red`)

## Testing and Deployment

Pine Script indicators are tested and deployed directly within the TradingView platform. There are no traditional build, lint, or test commands for this repository type.

## Documentation Requirements

**IMPORTANT**: Each indicator/screener directory has its own README.md file that documents the functionality and visual characteristics. When you make any changes to logic, colors, or calculations, you MUST update the corresponding README.md file in the same directory to reflect these changes. This ensures documentation stays synchronized with the actual implementation.

## Pine Script Screener Development

Pine Script screeners are different from regular indicators and have specific requirements:

### Key Differences from Regular Indicators
- **No input() functions allowed** - Use fixed values only
- **Must have plot() or alertcondition()** - At least one is required
- **Maximum 5 request.* calls** - Limited external data requests
- **Works on watchlists** - Scans multiple symbols at once, not just single charts

### Screener Structure Pattern
```pine
//@version=6
indicator("Screener Name", overlay=false)  // Usually overlay=false for screeners

// Fixed parameters (no inputs)
period = 20

// Calculation logic
value = ta.sma(close, period)
condition = close > value

// Plots - become screener columns (visible, sortable, filterable by value)
plot(value, title="Column Name")  // Shows as numeric column: "Column Name: 123.45"

// Alert conditions - become filter options (not visible as columns)
alertcondition(condition, title="Filter Name")  // Shows as: "Filter Name = True/False"
```

### plot vs alertcondition Usage
- **plot()**: Creates visible columns with numeric values
  - Users can see the actual values
  - Can filter with operators (>, <, =, etc.)
  - Can sort by these values
  
- **alertcondition()**: Creates filter-only conditions  
  - No column displayed
  - Only True/False filtering
  - Good for complex multi-condition logic

### Example Implementation
```pine
// Numeric values to display and filter
plot(performance, title="Performance %")  // User sees: "Performance %: 15.3"

// Complex condition for filtering only
is_strong = perf_1m > 5 and perf_3m > 10
alertcondition(is_strong, title="Strong Momentum")  // User filters: "Strong Momentum = True"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hidapple)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hidapple)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
