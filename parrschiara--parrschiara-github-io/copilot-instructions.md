## parrschiara-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static website for **Parrocchia Santa Chiara** (a Catholic parish), hosted on GitHub Pages. No build system, no framework, no backend — pure HTML/CSS/vanilla JavaScript committed directly to the repo and served as-is.

## Deployment

Pushes to `main` are automatically served via GitHub Pages. There is no build step.

Un workflow GitHub Actions (`.github/workflows/update_liturgia.yml`) gira ogni lunedì alle 05:00 UTC: scarica i prossimi 90 giorni di dati liturgici da lachiesa.it tramite `scraper.py`, committa i nuovi JSON in `assets/liturgia/` e pusha. Si può attivare manualmente dalla UI di GitHub con "Run workflow".

## Architecture

Three main pages:

- **`index.html`** — Main portal. Dynamically loads the current day's liturgical data from `assets/html/YYYYMMDD.html` using the Fetch API. Contains character-encoding fix logic (UTF-8 → Italian accented characters) and Italian locale date formatting.
- **`pdf_viewer.html`** — Custom PDF viewer using PDF.js (v2.10.377, CDN). Accepts a `?file=` query parameter to select which PDF to display.
- **`upload.html`** — Admin tool that uses the GitHub REST API to upload PDFs directly from the browser, without needing git CLI access. Uses a Personal Access Token stored in `localStorage`.

## Content Files

- **`assets/liturgia/YYYYMMDD.json`** — Un file per giorno, generato da `scraper.py`. Contiene: `settimana`, `celebrazione`, `colore`, `letture[]`. Aggiornato automaticamente dal workflow CI.
- **`assets/pdf/`** — Liturgical PDFs (weekly bulletin, adoration, compline, seasonal services).
- **`foglio_settimanale.pdf`** (repo root) — Current weekly parish bulletin; also mirrored at `assets/pdf/foglio_settimanale.pdf`.

## Generazione manuale dei JSON liturgici

```bash
pip install -r requirements.txt
python scraper.py 2026-01-01 2026-12-31
```

I file già presenti vengono saltati automaticamente. Usare `--force` per rigenerare un file già esistente (es. `python scraper.py --force 2026-04-26`).

Opzioni disponibili: `--force`, `--no-text`, `--fix-version`.

### Estrazione testi letture (scraper.py)

Il salmo viene estratto **inline** dalla pagina del calendario (`Detailed/YYYYMMDD.shtml`) tramite `_extract_psalm_inline(section_div)`: legge `div.section-content-testo`, distingue ritornello (`<b>`) e versi, e separa le strofe dai nodi `NavigableString` whitespace-only tra due `<br>`.

Anche le altre letture (prima lettura, seconda lettura, vangelo, canto_vangelo) vengono estratte **inline** tramite `_extract_reading_inline(section_div)`. Per le letture standard (`section-content-testo`): la pagina lachiesa.it usa HTML malformato — il primo `<br>` non chiuso funge da contenitore del testo e `br.get_text()` restituisce il testo senza numeri di versetto. Per il **canto dal vangelo** non esiste `section-content-testo`: il testo si trova in `div.section-content`, dentro un `<p>` che contiene "Alleluia, alleluia." + `<br/>` + verso + `<br/>` + "Alleluia." — la funzione estrae questo `<p>`, filtra le righe "Alleluia, alleluia." e "Alleluia.", e unisce i versi con spazio. Se l'estrazione inline fallisce, si usa `fetch_reading_text(href)` (bibbia.php) come fallback.

## Foglietto Settimanale

Pagina web (`foglietto.html`) per generare e stampare il foglietto parrocchiale settimanale. Target: catechista non tecnico, accesso solo da browser.

### File principali
- **`foglietto.html`** — Tutto-in-uno: UI (selezione domenica, form letture/calendario), anteprima A4 `contenteditable`, stampa via `stampaPDF()`. Vanilla JS, zero dipendenze. Accessibile solo via URL diretto (nessun link pubblico in index.html). Include notifiche Telegram all'utilizzo (vedi sezione dedicata).
- **`assets/foglietto/foglietto.css`** — Layout foglietto: intestazione 3 colonne, etichetta data verticale, due colonne letture, tabella calendario, footer, `@media print` A4.
- **`assets/foglietto/appuntamenti_fissi.json`** — Appuntamenti ricorrenti per giorno. Struttura: `{ giorni: [{ giorno: "domenica"|"lunedi"|..., appuntamenti: [{ orario, descrizione, chiesa, intenzione, fisso, sospeso }] }], note_permanenti: { confessioni, whatsapp_community } }`.

### Struttura foglietto.html (JS)
- **`liturgyData`** — JSON da `assets/liturgia/YYYYMMDD.json`, caricato al cambio domenica.
- **`appFissi`** — `appuntamenti_fissi.json` caricato una sola volta all'avvio.
- **`varAppts[dIdx]`** — Array appuntamenti variabili per giorno (indice 0–7 dalla domenica).
- **`apptOverrides[key]`** — Override `sospeso`/`intenzione` per appuntamenti fissi; `key` = `"dIdx-aIdx"`.
- **`renderPage1()`** — Genera pagina letture. Il salmo usa `\n\n` per strofe e `\n` per versi (`<br>`). Le altre letture: paragrafi senza margine verticale extra.
- **`renderPage2()`** — Genera tabella calendario + footer confessioni/avvisi.
- **`adjustPage1Layout()`** — Algoritmo auto-scaling font: prova da 12pt scendendo a 6.6pt, si ferma al massimo che entra. Se nemmeno 6.6pt basta: sposta sezioni da colonna sinistra a destra (max 2 passaggi), poi crea pagina extra `.foglietto-page-overflow`. `SIZES = [12, 11.5, ..., 6.6]`.

### Notifiche Telegram (foglietto.html)

Il foglietto invia una notifica Telegram al parroco ad ogni sessione di utilizzo effettivo.

- **Bot**: `@notifiche_foglietto_bot` — token e chat ID hardcoded in `foglietto.html` (uso interno, rischio basso).
- **Trigger**: click su "Aggiorna anteprima" → notifica immediata; click su "Stampa / Salva PDF" → notifica immediata; modifica nei campi form o nel preview `contenteditable` → notifica dopo 60 secondi di inattività.
- **Cooldown**: 30 minuti via `localStorage` (`foglietto_last_notify`). Più trigger ravvicinati generano al massimo una notifica per finestra.
- **Funzioni**: `notifyTelegram(azione)` (invia se cooldown scaduto), `notifyTelegramInput()` (debounce 60s prima di chiamare `notifyTelegram`), `stampaPDF()` (wrapper di `window.print()` che notifica prima di stampare).

### Formato testo letture nel JSON
- Prima lettura, seconda lettura, vangelo: testo continuo, `\n` tra blocchi logici (paragrafi), niente numeri di versetto (rimossi inline da scraper).
- Salmo: `ritornello` separato + `testo` con `\n\n` tra strofe e `\n` tra versi nella stessa strofa.
- Canto dal Vangelo: `sigla`, `url`, e `testo` (il verso dell'Alleluia senza le acclamazioni "Alleluia, alleluia." / "Alleluia." che il foglietto aggiunge autonomamente).

## Language

All content, UI strings, and commit messages are in **Italian**.

---
> Source: [ParrSChiara/parrschiara.github.io](https://github.com/ParrSChiara/parrschiara.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
