## usa-weather-cloth-suggestion

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A USA-only weather web app where users search for a US city (by name or zip code) and pick a date (today or future) to see the weather forecast on an interactive US map, along with clothing suggestions.

## Tech Stack

- **Runtime**: Node.js with TypeScript
- **Frontend**: React
- **Package manager**: Yarn (never use npm)

## Common Commands

```bash
yarn install          # Install dependencies
yarn dev              # Start development server (server :3001, client :5173)
yarn build            # Build for production
yarn lint             # Type-check client and server (tsc --noEmit)
```

> No test suite is set up yet.

## Architecture

The app has a React frontend and a Node.js/TypeScript backend:

- **Frontend** (`src/client/`): React components — city autocomplete input, date picker, US map, weather/clothing result panel
- **Backend** (`src/server/`): Express API with two endpoints: city autocomplete and weather query
- **Shared types** (`src/types/`): TypeScript interfaces shared between client and server (e.g., `CityResult`, `WeatherData`, `ClothingSuggestion`)

## Frontend Requirements

- **City input**: text box with dropdown autocomplete showing matching US cities/zip codes as the user types
- **Date picker**: only allows today or future dates
- **Submit button**: disabled until both a city and a date are selected
- **US map**: displays the whole USA; after submission, highlights the selected city with a temperature-based color indicator
- **Result panel**: read-only text area showing weather details and clothing suggestions

## Backend Requirements

- **`GET /api/cities?q=<query>`**: returns matching US cities; debounce is handled client-side, but the endpoint must be fast
- **`GET /api/weather?city=<city>&date=<date>`**: returns weather data and clothing suggestions for the given US city and date
- Restrict city lookups and weather queries to US locations only

## Key Dependencies

- **Weather data**: [Open-Meteo](https://open-meteo.com/) — free, no API key required; returns daily forecast by lat/lon
- **City/ZIP data**: `zipcodes` npm package — US-only, used for autocomplete and ZIP lookup
- **Map**: `react-simple-maps` + `us-atlas` — renders the AlbersUSA projection
- **Temperature color**: `d3-scale` / `d3-scale-chromatic` — maps °F to a color gradient

---
> Source: [lexiaoau/usa-weather-cloth-suggestion](https://github.com/lexiaoau/usa-weather-cloth-suggestion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
