## api-ventas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

API REST (FastAPI) that creates sales vouchers (comprobantes de venta) in an Oracle database. It inserts into `FACTURA_VENTAS` (header) and `DET_FAC_VEN` (detail lines). The `PENDIENTES` table is populated automatically by an Oracle trigger on `DET_FAC_VEN`.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run dev server (port 8010, auto-reload)
uvicorn main:app --host 0.0.0.0 --port 8010 --reload

# Or via main.py directly
python main.py
```

Swagger UI: `http://localhost:8010/docs`

## Architecture

Single-endpoint API (`POST /comprobante`) with this flow:

1. Get next ID from `SEQ_FACTURAS_VEN` sequence
2. Lock and increment `NUMERADORES` (C_NUMERADOR=20) for `R_COMPROBAN`
3. Insert header into `FACTURA_VENTAS` (hardcoded: C_COMPROBANTE=21, C_TIPO_COMPRO=2, C_SUCURSAL=1, C_EMPRESA=5)
4. Insert detail lines into `DET_FAC_VEN` (hardcoded: C_TIPO_CLASE=40, C_DEPOSITO=79)
5. Commit — Oracle trigger handles `PENDIENTES`

All steps run in a single transaction with rollback on failure.

**Key files:**
- `main.py` → FastAPI app, registers routers
- `db.py` → Oracle connection (thick mode for Oracle 11g compatibility)
- `models.py` → Pydantic schemas (request/response)
- `routers/comprobante.py` → Endpoint logic with hardcoded business constants

## Database

Connection configured via `.env` (see `.env.example`). Requires Oracle Instant Client (thick mode).

| Environment | User           | DSN                         |
|-------------|----------------|-----------------------------|
| TEST        | REDV12_TEST005 | 192.168.160.26:1521/PRODRDL |
| PROD        | REDV12_EMP005  | 192.168.160.26:1521/PRODRDL |

**Important:** Never prefix schema name in queries — connect directly as the schema user.

## Test Client (TEST environment)

C_CLIENTE=456, N_CLIENTE="VENTA INTERNA" — clone of PROD client 123, for testing only.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/efrancalancia)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/efrancalancia)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
