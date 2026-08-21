## hentested-streetview

> Henter Street View-bilder av hentesteder ut fra koordinater i en CSV og Google

# CLAUDE.md — arbeidsregler for dette prosjektet

## Hva prosjektet er
Henter Street View-bilder av hentesteder ut fra koordinater i en CSV og Google
Maps API. **Ingen maskinlæring** — ikke legg til YOLO, segmentering, deteksjon
eller GPS-estimering. Hold prosjektet til CSV → koordinat → bilde.

## Tolk og kjøring
Pakkene (`requests`, `numpy`) ligger i global Python 3.14. Kjør alt med `py`:
```
py -3.14 -m src.map_server
py -3.14 -m src.fetch_from_csv --csv data/hentesteder_chunks/hentesteder_001.csv
py -3.14 -m src.fetch_from_addresses
```

## Stier
- Bilder lagres i `data/images/`.
- Logg over hver henting: `data/streetview_log.csv`.
- CSV-data legges i `data/` (`hentesteder.csv`, `hentesteder_chunks/`, eller Uttrekk-eksport).

## Data og repo-hygiene
- CSV-data og bilder er kommunedata — **aldri commit dem** (allerede i `.gitignore`).
- Sett `GOOGLE_MAPS_API_KEY` som miljøvariabel; legg aldri nøkkelen i koden.

## Kodekvalitet
- Type-hint i alle funksjonssignaturer.
- Klare, beskrivende navn. Kommenter kun der grunnen ikke er åpenbar.
- Enkel og lesbar kode framfor smart og kompakt.

---
> Source: [davgei/hentested-streetview](https://github.com/davgei/hentested-streetview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
