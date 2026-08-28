## omarchy-weather

> Omarchy shell plugin (`service` + `bar-widget`). Display name:

# AGENTS.md

Omarchy shell plugin (`service` + `bar-widget`). Display name:
**Detailed Weather**. Plugin id stays `io.github.calebhat.weather`.
Disable `omarchy.weather` when enabling this one so the bar has a
single weather pill.

## Layout

- `manifest.json` — contract, settings schema
- `BarWidget.qml` — bar pill; forwards panel contract; injects the service
- `Panel.qml` — forecast panel, peek, Open radar, Settings
- `Service.qml` — optional storm alerts (one per session)
- `Model.js` — forecast parsing (Open-Meteo / wttr)
- `RadarModel.js` — saved radar website URLs; alert helpers

There is no in-panel radar map.

## Data

Open-Meteo, wttr.in. Location file owned by `omarchy-weather-location`.
Open radar launches a user-saved https website.

## Dev

```
omarchy plugin validate ~/.config/omarchy/plugins/io.github.calebhat.weather
qmllint -I "$OMARCHY_PATH/shell" BarWidget.qml Panel.qml Service.qml
omarchy restart shell
```

Hot reload of QML is unreliable after atomic writes; restart the shell.

---
> Source: [calebhat/omarchy-weather](https://github.com/calebhat/omarchy-weather) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
