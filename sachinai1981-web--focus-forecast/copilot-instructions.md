## focus-forecast

> Predict next 7 days of Pomodoro focus minutes using Google TimesFM 2.5. Log each day, beat the forecast.

# Focus Forecast — daily app

Predict next 7 days of Pomodoro focus minutes using Google TimesFM 2.5. Log each day, beat the forecast.

## Daily flow
1. `./run.sh` (boots Flask + opens dashboard at http://localhost:8765)
2. Hit **Start** on the Pomodoro timer · focus 25 / break 5 (configurable inline)
3. Each completed focus session auto-logs to `focus.csv` (cumulative for the day)
4. Skip mid-session = partial credit (logs minutes elapsed if ≥ 30s)
5. Manual "Save & forecast" card at the bottom for backfilling missed days

## Timer
- localStorage persistence — refresh resumes; sessions across page reloads stay correct
- Browser title shows live countdown so it's visible from another tab
- Soft beep on phase change (WebAudio sine wave, 660 Hz, 0.6s)
- Auto-completion never fires on page restore (avoids logging AFK time)

## Architecture
```
focus.csv         single source of truth (date,minutes)
   │
   ▼
app.py            Flask · :8765 · serves dashboard + /log + /data
   │
   ▼
forecast.py       make_forecast(df) → cold-start baseline OR TimesFM 2.5
   │
   ▼
forecast.json     dashboard reads this · written on every refresh
forecast.png      shareable PNG · written on every refresh
```

## Two forecast modes
- **`baseline`** — days 1–13. Weekday-mean (or global mean if no weekday signal yet). Pill shows `BASELINE · DAY N`.
- **`timesfm`** — day 14+. Loads `google/timesfm-2.5-200m-pytorch` once (cached in Flask process), forecasts in <1s after warmup. Pill shows `TIMESFM`.

## Files
| File | Purpose |
|---|---|
| `app.py` | Flask backend (72 lines) |
| `forecast.py` | model logic + CLI entrypoint (109 lines) |
| `index.html` | single-file dashboard, no build step (268 lines) |
| `focus.csv` | data — header only when fresh |
| `focus.sample.csv` | 60-day synthetic demo data, restore for testing |
| `run.sh` | one-command start |

## Run
```bash
./run.sh                               # daily — boots app + opens browser
python forecast.py focus.csv           # one-shot CLI forecast
cp focus.sample.csv focus.csv          # restore demo data
```

## Reset to zero
```bash
printf "date,minutes\n" > focus.csv
curl http://localhost:8765/data        # triggers refresh of forecast.json
```

## Endpoints
- `GET /` → dashboard
- `GET /data` → `{mode, history_count, history_target, history[], forecast[]}`
- `POST /log {minutes, date?}` → upserts row in CSV, re-runs forecast, returns same shape as /data

## Notes
- All offline. No API keys. Model weights cached at `~/.cache/huggingface/`.
- `matplotlib.use("Agg")` set in `forecast.py` — required for Flask worker threads on macOS.

---
> Source: [sachinai1981-web/focus-forecast](https://github.com/sachinai1981-web/focus-forecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
