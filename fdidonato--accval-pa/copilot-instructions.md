## accval-pa

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Lingua: comunica e commenta in **italiano** (vedi istruzioni globali utente).
> Ambiente: **Windows**, shell primaria **PowerShell**; Bash disponibile per script POSIX.

## Cos'è

`accval-pa` — validatore di accessibilità **WCAG 2.1** da riga di comando, interamente locale.
Motore di regole: **axe-core** (vendorizzato); modello di report e metriche: **MAUVE++**
(CNR-ISTI / AgID). Non reimplementa le regole WCAG: le presenta e le aggrega secondo MAUVE++.

## Comandi

Ambiente: usa il **virtualenv dedicato** `.venv/` (non il Python di sistema). Prefissa i comandi
con `.venv/Scripts/python.exe -m ...` (Windows). Il primo setup:

```powershell
python -m venv .venv
.venv/Scripts/python.exe -m pip install -e ".[dev]"  # pacchetto + dev deps (include pikepdf, pypdf)
.venv/Scripts/python.exe -m playwright install chromium  # browser di rendering (~180 MB), una tantum

.venv/Scripts/python.exe -m pytest -q                  # test unitari veloci (default: SENZA browser)
python -m pytest -q -m browser       # end-to-end con Chromium reale (lenti)
python -m pytest -q -m "browser or not browser"   # tutto
python -m pytest -q tests/test_scoring.py::TestAccessibilityPercentage  # singolo test
python -m ruff check src tests       # lint (deve restare pulito)
```

- I test browser sono **deselezionati di default** (`addopts = "-m 'not browser'"` in `pyproject.toml`);
  vanno chiesti esplicitamente con `-m browser`.
- Le fixture end-to-end sono in `tests/fixtures/`, caricate via `file://`. Un test verifica
  anche l'accessibilità del report generato (axe sul report stesso: 0 violazioni A/AA).
- Stato attuale: **352 test non-browser + 77 browser = 429**, ruff pulito. Le fixture PDF sono
  costruite a runtime con pikepdf (`tests/test_pdf.py`).

## Architettura

Pipeline lineare, un modulo per fase, orchestrata da `cli.py`:

```
cli → renderer (Playwright) → engine (axe-core) → mapping → scoring → report

webui (server locale) → sottoprocesso `accval-pa --progress-json` → eventi JSON
```

| Modulo | Responsabilità |
|---|---|
| `cli.py` | argparse, wiring, tre percorsi: `run_single`, `run_pdf`, `run_site` (crawler / sitemap / elenco); codici di uscita |
| `renderer.py` | sessione Chromium **riusata** su tutte le pagine; `goto` + `--settle-ms`; screenshot full-page |
| `engine.py` | inietta ed esegue axe sul **DOM renderizzato**; arricchisce i frammenti HTML |
| `shots.py` | ritagli PNG per singola occorrenza (overlay a anello, nessun velo) |
| `mapping.py` | JSON axe → `TechniqueResult`; collezioni axe → esiti MAUVE++ |
| `pdf.py` | validazione PDF: 7 tecniche MAUVE++ con pikepdf/pypdf → `TechniqueResult` |
| `scoring.py` | metriche di pagina + aggregazione di sito |
| `wcag.py` | tabella criteri WCAG 2.1+2.2, livelli, tag axe, conversioni |
| `crawler.py` | BFS sul dominio, `robots.txt`, sitemap XML, elenco di URL (`--urls-file`) |
| `models.py` | dataclass del modello dati |
| `progress.py` | eventi di avanzamento; `TextProgress` (output storico) e `JsonProgress` (`--progress-json`) |
| `webui/{server,jobs,args,pages}.py` + `templates/app.html.j2` | interfaccia grafica: server locale (token + Host + Origin), sottoprocesso della CLI, pagina HTML con progressive enhancement |
| `report/{html,pdf,site,earl}.py` + `templates/*.j2` | report di pagina, report PDF, indice di sito, export EARL |

**Punti che richiedono di leggere più file per capirli:**

- **axe gira sul DOM vivo, non sull'HTML della risposta.** L'ordine è: `renderer` carica e
  stabilizza la pagina → `engine` inietta `axe.min.js` → `axe.run(document)`. È ciò che rende
  valutabili le SPA (Angular/Vue/React). La fixture `tests/fixtures/spa.html` esiste per
  proteggere questo invariante.
- **Le opzioni di rendering sono globali**, non ristrette alla pagina singola: `run_single` e
  `run_site` chiamano lo stesso `build_render_options`; l'attesa vive in `Session.open`, percorso
  comune. Vale per `--settle-ms`, `--timeout`, `--wait`, `--viewport`, `--level`, `--wcag`,
  `--snippet-context`, `--screenshot`, `--disable-cors`.
- **Ogni regola axe = una "tecnica" MAUVE++.** La mappatura esiti è in `models.AXE_COLLECTION_TO_OUTCOME`:
  `passes→pass`, `violations→error`, `incomplete→warning` (cannot tell), `inapplicable→not_applicable`.
- **I tag axe sono il prodotto di due assi cumulativi** (`wcag.py`): versione (`_VERSION_PREFIXES`)
  × livello (`_LEVEL_SUFFIXES`). `--wcag` e `--level` sono entrambi cumulativi.

## Invarianti da NON rompere

Vedi `.claude/rules/accval-pa.md` per il dettaglio con motivazioni. In sintesi:

1. **Metrica di sito = macro-average** (media aritmetica delle pagine). Scelta deliberata dell'utente.
2. **Best practice axe escluse di proposito** (`runOnly` a soli tag `wcag*`): fedeltà a MAUVE++.
3. **Default `--wcag 2.1`**: cambiarlo sposterebbe i punteggi e romperebbe la confrontabilità
   con i report già emessi.
4. **Il report resta accessibile e funzionante senza JS** (progressive enhancement: Jinja2
   renderizza tutto, il JS solo filtra). L'escaping dei frammenti HTML è demandato all'autoescape
   di Jinja2 — **non** pre-escapare in Python.
5. **Isolamento licenza**: `accval-pa` è MPL-2.0 (come axe-core). L'eventuale modalità LWGD usa il
   `wcag21.xml` di MAUVE++ (**GPL-3.0**, copyleft forte) e va tenuta in un modulo opzionale isolato.

## Convenzioni

- Python **3.11+** (nota: la regola globale dice 3.12+, ma questo progetto è pinnato a 3.11 in
  `pyproject.toml` — segui il progetto). Type hint ovunque, `from __future__ import annotations`.
- Errori applicativi tipizzati in `errors.py` con `exit_code` (`RenderError`=1, `ConfigError`=2);
  la CLI li mappa ai codici di uscita. Nessun `except` nudo.
- `logging` (logger `"accval-pa"`), mai `print` per diagnostica. Messaggi in italiano.
- Docstring in italiano; molte referenziano il paragrafo del PRD (`PRD §N`) da cui derivano —
  mantieni il riferimento quando tocchi quella logica.
- axe-core è **versionato nel repo** (`assets/axe/axe.min.js`, v4.12.1), non scaricato a runtime;
  override con env `ACCVAL_AXE_PATH`.

## Riferimenti

- `.claude/rules/accval-pa.md` — invarianti di dominio con motivazioni.
- `.claude/skills/report-sito/SKILL.md` — come comporre il comando giusto per analizzare un sito.
- `docs/STATO-PROGETTO.md` — memoria interna: fasi implementative e cosa resta.
- `docs/RIFERIMENTI.md` — fonti verificate (MAUVE++ help.jsp/paper, LWGD, WCAG, EARL, axe-core) e
  verifica che pesi e formule di `scoring.py` coincidono con la metodologia ufficiale MAUVE++.

---
> Source: [fdidonato/accval-pa](https://github.com/fdidonato/accval-pa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
