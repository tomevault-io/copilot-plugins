## ollie-contabilidade

> Generates `dist/ExportadorPC23.exe` (standalone, no Python required on target machine).

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

```bat
build.bat
```
Generates `dist/ExportadorPC23.exe` (standalone, no Python required on target machine).

Requires on Windows:
- Python 3 + `pip install -r requirements.txt`
- ODBC Driver 17 for SQL Server (installed separately, not bundled by PyInstaller)

## Architecture

Single-file app: `app.py` — class `App(tk.Tk)` encapsulates everything.

- `get_config_path()` — returns path to `config.json` beside the executable (`sys.frozen` for bundled exe, `__file__` for dev)
- `_build_ui()` — builds all UI sections: connection frame, CSV output frame, export button, log area
- `_load_config()` / `_save_config()` — reads/writes `config.json` with `servidor`, `banco`, `usuario` (never `senha`)
- `_testar_conexao()` — opens/closes a pyodbc connection; result in log
- `_exportar()` — connects, runs query, writes CSV with `;` delimiter and `utf-8-sig` encoding
- `_escolher_csv()` — file save dialog

## Query

Fixed SQL query against tables `PC23T`, `PC23TA`, `PC23T1`. Do not modify without confirming with the business.

## CSV format

- Delimiter: `;`
- Encoding: `utf-8-sig` (UTF-8-BOM, required for Brazilian Excel)
- Header: `Modelo;Cor;Plano;Qdade_Total`
- Default filename: `PC23_YYYYMMDD_HHMMSS.csv`

## Connection string

`DRIVER={ODBC Driver 17 for SQL Server};SERVER=...;DATABASE=...;UID=...;PWD=...`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/edsonpreis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
