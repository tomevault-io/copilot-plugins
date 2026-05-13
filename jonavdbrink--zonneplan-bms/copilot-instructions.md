## zonneplan-bms

> This document provides context for AI agents interacting with the `zonneplan_peakdetect` integration.

# Agent Context: Zonneplan BMS / Battery Optimizer

This document provides context for AI agents interacting with the `zonneplan_peakdetect` integration.

## Purpose
The integration calculates an optimal battery charging and discharging schedule based on hourly energy prices (typically provided by Zonneplan). It aims to maximize profit by performing energy arbitrage: charging when prices are low and discharging when prices are high.

## Core Entity
- **Entity ID**: `sensor.battery_optimizer`
- **States**: `Charge`, `Discharge`, `Stop`
- **Attributes**:
    - `schedule`: A list of dictionaries containing `datetime`, `price_eur_kwh`, `price_multiplier`, `action`, and `interval_id`.
    - `intervals`: Number of identified arbitrage "waves" in the current forecast.
    - `min_profit_required_eur_kwh`: The threshold for a price difference to be considered profitable.

## Optimization Logic (The "Wave" Algorithm)
The integration uses a custom segmentation algorithm in `sensor.py`:
1. **Segmentation**: It splits the 24-hour forecast into "waves" by identifying local valleys (dips) and subsequent local peaks (humps).
2. **Profit Threshold**: A wave is only processed if the difference between the peak and the valley exceeds `min_profit_c_kwh`.
3. **Slot Selection**:
    - **Charge**: Selects the cheapest hours *before* the valley index within that wave.
    - **Discharge**: Selects the most expensive hours *after* the valley index within that wave.
4. **Balancing**: The number of charge slots is constrained by the number of available profitable discharge slots to ensure energy balance.

## Configuration Parameters
- `forecast_entity`: The source sensor providing a `forecast` attribute with hourly prices.
- `price_delta_percent`: Percentage-based threshold (used for multiplier calculation).
- `min_profit_c_kwh`: Minimum cents/kWh difference required to trigger an action.
- `charge_hours` / `discharge_hours`: Max hours per interval to perform the respective action.

## Common Agent Tasks
- **Analyzing the Schedule**: When asked "When will my battery charge?", the agent should look at the `schedule` attribute for items where `action` is `Charge`.
- **Troubleshooting**: If the sensor stays on `Stop`, check if the `min_profit` threshold is higher than the current market volatility in the `forecast_entity`.
- **Code Maintenance**:
    - Logic updates should happen in `BatteryOptimizerSensor._calculate_action_schedule`.
    - Ensure that price conversions in `_convert_price` match the raw data format (currently expects deci-micro-euro).

## Key Files
- `sensor.py`: Contains the main optimization logic and entity state management.
- `config_flow.py`: Handles user setup and parameter input.
- `const.py`: Defines DOMAIN and configuration keys.

## Dependencies
- Requires a forecast sensor with a specific `forecast` attribute structure (list of dicts with `datetime` and `electricity_price`).

---
> Source: [JonavdBrink/zonneplan_bms](https://github.com/JonavdBrink/zonneplan_bms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
