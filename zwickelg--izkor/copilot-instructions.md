## izkor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start          # Dev server at http://localhost:3000
npm run build      # Production build to /build
npm run deploy     # Build + publish to GitHub Pages (gh-pages)
npm test           # Run tests in watch mode
npm test -- --watchAll=false   # Run tests once (CI mode)
npm test -- --testPathPattern=<file>  # Run a single test file
```

Deployment also uploads the `/build` folder contents to an S3 bucket behind CloudFront at `https://d5ajvage8yosb.cloudfront.net`.

## Architecture

**Izkor** (יזכור) is a Hebrew-language PWA for Jewish memorial prayers. Users enter the name and details of a deceased person, then the app guides them through a sequence of prayers (Psalms, Kaddish, El Male Rahamim, etc.) displayed one screen at a time.

### Routing

Uses `HashRouter` (required for GitHub Pages). Routes are in `src/Main.tsx`:

| Route | Component | Purpose |
|---|---|---|
| `/` or `/page0` | `MainForm` | Enter deceased's name, gender, liturgical version |
| `/page1` | `PrayerDetails` | Review details; share/copy URL, NFC write, print |
| `/page2` | `PrayerAll` | Full prayer sequence, scroll-paged |
| `/print` | `PrayerAllPrint` | Print-optimized view of all prayers |
| `/nfc` | `NfcReadWrite` | NFC tag read/write utility |

### State (Redux)

`src/features/izkor/izkorSlice.ts` holds the single slice used throughout the app:

```ts
{ firstName, lastName, gender, parentName, version, mode, theme }
```

- `version`: `"ashkenazic"` | `"sephardic"` — controls which liturgical variant is rendered in prayer components
- `mode`: `"all"` | `"readonly"` — set to `"readonly"` when the app is opened via a shared URL (disables Back/Edit navigation)
- `gender`: affects grammatical forms in Hebrew prayer text

### URL Sharing / Deep Links

`src/components/utils/compressUtil.ts` compresses the Redux state to a Base64 string using LZ-string. `PrayerDetails` and `PrayerAllPrint` encode the state as a `?data=` query parameter. When the app loads with `?data=`, `MainForm` decompresses it, hydrates the Redux store via `updateFields`, and sets `mode: "readonly"`, then navigates to `/page1`.

### Prayer Components

Each prayer lives in `src/components/prayers/PrayerXxx.tsx`. They receive `IzkorDoc` props (or read from Redux) to interpolate the deceased's name and gendered Hebrew text.

`PrayerAll.tsx` renders all prayers in a single scrollable container. It tracks which section is visible using `IntersectionObserver`-style scroll tracking and shows a section navigation indicator.

`PrayerAllPrint.tsx` renders all prayers flat (no scroll paging) for browser `window.print()`.

`IzkorPdfDocument.tsx` uses `@react-pdf/renderer` to produce a downloadable PDF, loaded lazily via the print flow.

### UI / Theming

- MUI v5 with RTL support via `stylis-plugin-rtl` + `@emotion/react` `CacheProvider`
- Theme configured in `App.tsx` with dark/light toggle and `--prayer-font-size` CSS variable for font size control (A+/A- buttons)
- Hebrew fonts (`Assistant`, `FrankRuehl`, `nrkis`) are in `src/fonts/` and `public/fonts/`
- All text is Hebrew (RTL); `<html dir="rtl" lang="he">` is set in `public/index.html`

### PWA

Service worker is registered in `src/index.tsx` via `serviceWorkerRegistration.register()`, enabling offline use.

### NFC

`src/components/utils/NfcHandler.ts` wraps the Web NFC API (`NDEFReader`) to write the shareable URL to an NFC tag. Only available on Chrome for Android.

---
> Source: [zwickelg/izkor](https://github.com/zwickelg/izkor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
