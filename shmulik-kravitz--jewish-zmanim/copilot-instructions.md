## jewish-zmanim

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
yarn test              # Run all tests (watch mode)
yarn test --run        # Run tests once
yarn test <file>       # Run a single test file, e.g. yarn test tests/zmanim.test.ts
yarn typecheck         # TypeScript type-check without emitting
yarn build             # Typecheck + bundle with tsdown (outputs to dist/)
yarn dev               # Watch mode build
```

## Architecture

This is a TypeScript library (dual CJS/ESM via `tsdown`) for calculating Jewish halachic times (zmanim).

**Core data flow:**

1. `cities.ts` — static registry of `CityRow` objects grouped by country (e.g. `cities.israel.jerusalem`)
2. `timezone.ts` — converts coordinates or a `CityRow` to a resolved `CityInfo` (IANA timezone → UTC offset + DST flag)
3. `solar.ts` — Jean Meeus algorithm; `calcSunTimes()` returns sunrise/sunset as fractional hours for a given zenith angle
4. `zmanim.ts` — `Zmanim` class; constructor takes a `CityInfo` + date, pre-computes all times using `solar.ts`. Exposes `getTimes()` → `ZmanimTimes` 
5. `index.ts` — re-exports everything; public API surface

**Key design points:**

- All times are stored internally as fractional hours (e.g. `6.456`) and converted to `"HH:MM:SS"` strings via `toTime()`.
- `Zmanim` has two static factories: `fromCityRow(row, date)` (resolves timezone from `CityRow`) and `fromCoordinates(lat, lng, date, opts?)` (auto-detects timezone, returns `Zmanim & { timezoneInfo: TimezoneResult }`).
- Shaah zmanit (halachic hour) has two flavors: GRA (sunrise→sunset / 12) and Alter Rebbe / MGA (alos72→tzeis72 / 12). Both are computed in the constructor.
- Candle lighting defaults: Jerusalem = 40 min, other Israeli cities = 30 min, diaspora = 18 min.
- Tefillin degrees defaults: Israel = 11.5°, diaspora = 10.2°.
- `isIsraelCity()` is a private method used inside the constructor for these Israel-specific defaults.

**Test fixtures** live in `tests/cityInfos.ts` — a map of pre-resolved `CityInfo` objects used across all test files.

---
> Source: [Shmulik-Kravitz/jewish-zmanim](https://github.com/Shmulik-Kravitz/jewish-zmanim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
