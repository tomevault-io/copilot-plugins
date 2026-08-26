## biblioteca-personal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal collection of standalone Python scripts for cataloging a personal ebook library, syncing files to Google Cloud Storage, and maintaining a searchable catalog in BigQuery. There is no package structure, test suite, linter config, or dependency manifest — each script is self-contained and run directly.

## Running the scripts

No `requirements.txt` exists. Dependencies used across the scripts: `pandas`, `pytz`, `google-cloud-storage`, `google-cloud-bigquery`, `google-auth`.

```bash
pip install pandas pytz google-cloud-storage google-cloud-bigquery google-auth

python3 main.py            # batch: process a completed catalog CSV
python3 incremental.py     # interactive: catalog new files one at a time
python3 listar_formatos.py # utility: list distinct file extensions under a folder
```

Both `main.py` and `incremental.py` need a Google Cloud service account JSON key with Storage + BigQuery access, plus network access to GCS/BigQuery — they authenticate and instantiate clients at **import time** (module top level), so they cannot be imported or unit-tested without valid credentials present at the configured path.

## Architecture

The scripts form a loose pipeline around a single catalog concept — a row per book with identity, location, and cloud-sync metadata (`id`, `titulo_original`, `autor`, `nuevo_titulo`, `extension`, `categoria`, `ruta_completa`/`ruta_local`, `uri_gcs`, `url_publica`, `url_autenticada`, `privado`, `comentarios`, `fecha_carga`). The `id` is always an md5 hash (first 12 hex chars) derived from path/title/author/category/extension, used as the BigQuery merge key.

- **`organizar_libros.ipynb`** — exploratory precursor. Walks a folder for book files (`.pdf .epub .mobi .fb2 .opf .cbr .cbz .txt .rtf`), builds a bare-bones DataFrame, and writes `catalogo_biblioteca.csv`. The user is expected to hand-fill in `autor`, `nuevo_titulo`, `categoria`, `comentarios` in that CSV afterward.
- **`main.py`** — batch mode. Reads the hand-completed `catalogo_biblioteca.csv` (`CSV_IN`), validates required columns, generates `id`s, reads filesystem creation/modification timestamps, optionally copies/renames files locally (`RENOMBRAR_LOCAL`), uploads to GCS (`SUBIR_GCS`), and upserts the whole DataFrame into BigQuery (`ACTUALIZAR_BIGQUERY`). Rows with `nuevo_titulo == "BORRAR"` are dropped/skipped. Writes `catalogo_actualizado.csv` at the end. Config (paths, project/dataset/bucket names, feature flags) is hardcoded at the top of the file, using Windows-style paths (`E:/Biblioteca_personal/...`).
- **`incremental.py`** — interactive incremental mode, meant for adding newly-acquired books without regenerating the full CSV. Walks only the top level of a folder (`os.walk` breaks after the base directory), and for each new recognized file prompts the user on stdin for `autor`/`nuevo_titulo`/`categoria`/`privado`/`comentarios`. Then moves (not copies) the file into `categoria/`-organized subfolders, uploads to GCS, and upserts to BigQuery with an explicit `bigquery.SchemaField` schema (stricter than `main.py`'s inferred schema). Config is portable — paths resolve relative to the script's own directory and read from `BIBLIOTECA_RAW` / `GOOGLE_APPLICATION_CREDENTIALS` env vars, unlike `main.py`'s hardcoded paths.
- **`listar_formatos.py`** — standalone utility, unrelated to the catalog pipeline. Prompts for a folder path and prints every distinct file extension found recursively; useful for deciding what to add to the `formatos_libros` extension sets in the other scripts.

### BigQuery upsert pattern

Both `main.py` and `incremental.py` implement the same upsert idiom in `upsert_bigquery()`: load the DataFrame into a throwaway `..._temp_upsert_<unix_ts>` table (`WRITE_TRUNCATE`), run a `MERGE` from that temp table into the real `catalogo` table keyed on `id`, then delete the temp table. `fecha_carga` is always stamped as `CURRENT_DATETIME('America/Lima')` on both insert and update, regardless of what was in the source DataFrame.

### Known inconsistencies (don't silently "fix" without checking with the user)

- `main.py` and `incremental.py` duplicate most helpers (`generar_id`, `safe_filename`, `upload_to_gcs`, `construir_urls_gcs`, `upsert_bigquery`, ...) with slightly different logic (e.g. `generar_id` hashes different fields in each; `main.py` copies files, `incremental.py` moves them). These have drifted independently across commits rather than sharing code — treat each script as authoritative only for itself.
- Config (credential paths, GCS bucket/project, local base directory) is hardcoded per-script rather than centralized, and `main.py` still uses Windows drive-letter paths while `incremental.py` uses env-var-driven relative paths.

---
> Source: [JChz6/biblioteca_personal](https://github.com/JChz6/biblioteca_personal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
