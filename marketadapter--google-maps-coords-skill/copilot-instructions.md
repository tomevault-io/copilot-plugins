## google-maps-coords-skill

> Tool that extracts latitude and longitude from any Google Maps URL.

# google-maps-coords

Tool that extracts latitude and longitude from any Google Maps URL.

## When to use

When the user provides a Google Maps URL and wants the coordinates (lat/lng).

## How to run

```bash
python3 extract.py "<google_maps_url>"
```

If installed globally (see README), use the full path:

```bash
python3 ~/.codex/tools/google-maps-coords-skill/extract.py "<google_maps_url>"
```

## Supported URL types

- Short URL: `https://maps.app.goo.gl/XXXXX`
- Place URL: `https://www.google.com/maps/place/.../@lat,lng,17z/`
- Search URL: `https://www.google.com/maps?q=lat,lng`
- iframe embed: `https://www.google.com/maps/embed?pb=...`
- data= URL: `https://www.google.com/maps/place/NAME/data=...!3d<lat>!4d<lng>...`

## Output

```
위도 (lat): 37.123456
경도 (lng): 127.456789
```

## Requirements

- Python 3.10+
- `curl` (for short URL resolution only — pre-installed on macOS/Linux)

---
> Source: [marketadapter/google-maps-coords-skill](https://github.com/marketadapter/google-maps-coords-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
