## research-pipeline

> Automatisierte Recherche-Pipeline: YouTube-Videos scrapen, mit NotebookLM analysieren, Insights generieren.

# Research Pipeline

Automatisierte Recherche-Pipeline: YouTube-Videos scrapen, mit NotebookLM analysieren, Insights generieren.

## Projektstruktur
- `scripts/yt_scraper.py` -- YouTube Scraper (Search + Batch-URL Modus) mit yt-dlp
- `output/` -- Alle Scraper-Ergebnisse (Transkripte, Metadaten, Analysen)
- `.claude/commands/yt-scrape.md` -- Skill: YouTube scrapen
- `.claude/commands/notebooklm.md` -- Skill: NotebookLM steuern
- `.claude/commands/insight.md` -- Workflow: Komplette Recherche-Pipeline

## Verfuegbare Befehle
- `/yt-scrape <suchbegriff oder URLs>` -- Videos suchen/verarbeiten
- `/notebooklm <natuerlichsprachlicher Befehl>` -- NotebookLM steuern
- `/insight <suchbegriff oder URLs>` -- Komplette Pipeline ausfuehren

## Abhaengigkeiten
- `yt-dlp` (YouTube-Downloads)
- `notebooklm-py[browser]` (NotebookLM API)
- `playwright` + Chromium (fuer NotebookLM Login)

## Setup
```bash
uv sync
playwright install chromium
notebooklm login  # einmalig in separatem Terminal
```

## Wichtig
- NotebookLM erfordert einmaligen Login: `notebooklm login` in separatem Terminal
- Max 50 Quellen pro NotebookLM Notebook
- Output-Verzeichnisse haben Zeitstempel-Prefix

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skatekowski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
