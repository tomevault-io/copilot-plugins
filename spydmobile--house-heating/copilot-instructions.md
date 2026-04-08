## house-heating

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a data analysis project for tracking home heating fuel consumption for a property in Fort Smith, Northwest Territories. The project correlates fuel usage with weather data (Heating Degree Days) to:
- Measure fuel efficiency (L/HDD - liters per heating degree day)
- Predict refill timing
- Quantify window replacement impact (new windows installed Nov 6-8, 2025)
- Track costs and savings

## Key Files

- **43 Staint Annes Fuel Analyzer V1.1.xlsx**: Main Excel workbook with fuel tracking sheets (Fuel Fills, Gauge Readings, Weather Data, Analysis, Q1 Comparison, Instructions)
- **Fort_Smith_Fuel_Analysis_Summary.md**: Comprehensive analysis document with all historical data and methodology

## Domain Knowledge

### Heating Degree Days (HDD)
- Formula: `HDD = max(0, 18°C - average_daily_temperature)`
- Canadian standard base temperature is 18°C
- Higher HDD = colder weather = more heating needed

### Key Metrics
- **L/HDD**: Liters of fuel per Heating Degree Day (efficiency measure)
- Baseline efficiency (Q1 2025, old windows): 0.191 L/HDD
- Current efficiency (Sept-Nov 2025): 0.438 L/HDD
- Seasonal variation is expected (deep winter more efficient due to steady furnace operation vs. shoulder season cycling)

### Tank Specifications
- Capacity: 1000 Liters
- Gauge: Fractional readings (1/8, 1/4, 3/8, etc.)
- Location: Fort Smith, NWT (Station ID: 2202202 for weather data)

### Cost Structure
- 2¢/L flat discount on all fills
- 5% GST applied after discount
- Cost formula: `((Liters × Price/L) - (Liters × 0.02)) × 1.05`

## Weather Data Source
Environment Canada Climate Data: https://collaboration.cmc.ec.gc.ca/cmc/climate/Get_More_Data_Plus_de_donnees/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spydmobile)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spydmobile)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
