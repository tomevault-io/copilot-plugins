## exercise-timer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run build       # format + minify JS + minify CSS (run before deploying)
npm run format      # prettier --write . (formats index.js and index.css)
npm run minify-js   # uglifyjs index.js > index.min.js
npm run minify-css  # cleancss -o index.min.css index.css
```

There are no tests.

## Architecture

This is a single-page, no-framework exercise timer app. All source code is in three files at the root:

- [index.html](index.html) — loads `index.min.css` and `index.min.js` (the built artifacts, not the sources)
- [index.js](index.js) — all application logic
- [index.css](index.css) — all styles

**After any change to `index.js` or `index.css`, run `npm run build`** to regenerate the minified files that `index.html` actually loads.

### Exercise data format

Exercises are stored in `localStorage` (key: `exercises`) and entered in a textarea. Each line: `<prefix><name>;<seconds>`

- `+` prefix → exercise (green highlight, `exercise-color`)
- `-` prefix → pause (yellow highlight, `pause-color`)

Example: `+kliky;30` means exercise "kliky" for 30 seconds.

`parseTextarea()` in [index.js:169](index.js#L169) converts raw lines into `{type, value, time}` objects.

## Naming conventions

Variable and function names must be full words, not abbreviations. Use `textarea`, not `ta`; use `exercise`, not `ex`.

### Timer state

Four module-level variables control state: `stoptime`, `pause`, `sec`, `index`. The `timerCycle()` function calls itself via `setTimeout(timerCycle, 1000)` — stopping is done by setting `stoptime = true`, which prevents re-scheduling.

A whistle sound (`whistle.mp3`) plays at `sec === 1` (one second before an exercise ends). The app requests a [Screen Wake Lock](https://developer.mozilla.org/en-US/docs/Web/API/WakeLock) on start to prevent the screen from sleeping during a workout.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pavelrampas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
