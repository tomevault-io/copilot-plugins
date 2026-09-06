## personal-dashboard

> A beginner-friendly personal dashboard that displays live weather and inspirational quotes. Runs entirely in the browser with no backend or build tools.

# Personal Dashboard

## Project Overview
A beginner-friendly personal dashboard that displays live weather and inspirational quotes. Runs entirely in the browser with no backend or build tools.

## Tech Stack
- **HTML5** — semantic markup
- **CSS3** — flexbox layout, card-based UI
- **Vanilla JavaScript** — no frameworks or libraries
- **APIs:**
  - [Open-Meteo](https://open-meteo.com) — weather (no API key, CORS-friendly)
  - [DummyJSON](https://dummyjson.com/docs/quotes) — random quotes (no API key, CORS-friendly)

## File Structure
```
personal-dashboard/
├── CLAUDE.md          ← this file
├── index.html         ← single-page entry point
├── css/
│   └── style.css      ← all styles (reset, layout, cards, button)
└── js/
    └── app.js         ← all logic (API calls, DOM updates, refresh)
```

## Key Architecture Decisions
- `refreshAll()` orchestrates both fetches and manages the Refresh button state
- `fetchDone()` uses a simple counter (`pendingCount`) to re-enable the button only after both fetches complete
- `loadWeather()` uses `navigator.geolocation` then calls Open-Meteo — it is callback-based (geolocation) wrapping async/await (fetch)
- `loadQuote()` is fully async/await
- Weather codes are mapped via a plain object in `getWeatherDescription()`

## Development Workflow
- No build step — open `index.html` directly in a browser or use VS Code Live Server
- One feature at a time: implement, test in browser, confirm, then move on
- Always handle errors with user-friendly fallback messages
- Always call `fetchDone()` in every exit path (success, error, permission denied)

## Coding Style Rules
- Use `var` for broad compatibility (beginner project)
- Use `async/await` with `try/catch` for API calls
- Use `textContent` (not `innerHTML`) for safe DOM updates
- Add short comments explaining *why*, not *what*
- Keep functions small and single-purpose
- No external dependencies

## Future Roadmap
- [ ] Add a clock / date-time display card
- [ ] Add Fahrenheit/Celsius toggle
- [ ] Show city name alongside weather
- [ ] Cache weather data in localStorage to reduce API calls
- [ ] Add dark mode toggle
- [ ] Make layout responsive for mobile
- [ ] Add a "new quote" button on the quote card only

---
> Source: [Deep-1997-08/personal-dashboard](https://github.com/Deep-1997-08/personal-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
