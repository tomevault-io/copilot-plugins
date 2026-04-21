## homologacion-ukuvi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a vehicle homologation system that unifies vehicle catalogs from multiple insurance companies (aseguradoras) into a single master catalog. The system extracts data from each insurer's database, normalizes it using n8n workflows, and stores it in a Supabase PostgreSQL database with intelligent deduplication powered by token-overlap matching.

## Architecture & Data Flow

### Core Components

1. **Data Sources**: Insurance company databases (Qualitas, HDI, AXA, GNP, Mapfre, Chubb, Zurich, Atlas, BX, El Potosí, ANA)
2. **n8n Workflows**: ETL processes for extraction, normalization, and batch processing
3. **Supabase Database**: PostgreSQL database with RPC functions for intelligent homologation
4. **Canonical Model**: Standardized vehicle representation with hash-based deduplication and token-overlap matching

### Data Flow Process

```
Insurance DB → SQL Extraction → n8n Normalization → Batch Processing → Supabase RPC → Master Catalog
```

## Key Architecture Patterns

### Hash-Based Deduplication with Token Overlap

- **`hash_comercial`**: SHA-256 of `marca|modelo|anio|transmision` for basic vehicle identification
- **Token comparison**: `tokenize_version(version)` generates normalized arrays for order-independent matching across insurers
- **Similarity thresholds**: Configurable ratios (typically ≥0.92 for same-insurer reprocesses and ≥0.50 for cross-insurer merges)

### Canonical Data Model

The master table `catalogo_homologado` follows this schema:

- **Core fields**: `marca`, `modelo`, `anio`, `transmision`
- **Integrated version**: `version` field contains ALL technical specifications as a single string
- **Availability tracking**: JSONB field `disponibilidad` with per-insurer active/inactive status
- **Traceability**: `string_comercial`, `string_tecnico` for debugging and audit
- **No separate technical fields**: All specs (power, displacement, cylinders, doors, traction) are embedded in the `version` string

### Active/Inactive Status Management

- Each insurer can mark vehicles as active/inactive in their availability data
- Global validity: vehicle is considered active if ANY insurer reports it as active
- No records are deleted; inactivation updates the `disponibilidad` JSONB field
- Reactivation is supported by updating `activo=true`

## Directory Structure

### `/src/insurers/[name]/`

Each insurer has dedicated files:

- `[name]-analisis.md`: Data profiling and field mapping analysis
- `[name]-query-de-extraccion.sql`: SQL extraction query
- `[name]-codigo-de-normalizacion.js`: n8n normalization logic (when applicable)
- `ETL - [Name].json`: Complete n8n workflow definition

### `/src/supabase/`

3. **Normalization Code Node**: JavaScript transformation using insurer-specific logic
4. **Batch Processing**: Split data into chunks (10k-50k records)
5. **Supabase RPC Call**: POST to `/rest/v1/rpc/procesar_batch_homologacion`

### Normalization Requirements

Each normalization script must:

- Generate `hash_comercial` (SHA-256 of marca|modelo|anio|transmision)
- Generate `id_canonico` (SHA-256 of complete record including version)
- Create integrated `version` field containing ALL technical specifications in a single string
- Format: `[TRIM] [BODY] [POWER] [DISPLACEMENT] [CYLINDERS] [DOORS] [TRACTION]`
- Example: `"ADVANCE SEDAN 145HP 2L 4CIL 4PUERTAS AWD"`
- Map transmission codes to standard values (MANUAL/AUTO/null)
- Remove comfort/security features (AA, EE, CD, ABS, BA) from version strings
- Convert door format from "4P" to "4PUERTAS"
- Apply consistent marca/modelo standardization
- Preserve original data for traceability

## Supabase RPC Interface

### Main Function: `procesar_batch_homologacion`

**Endpoint**: `/rest/v1/rpc/procesar_batch_homologacion`
**Input Schema**:

```json
{
  "body": {
    "vehiculos_json": [
      {
        "id_canonico": "string",
        "hash_comercial": "string",
        "string_comercial": "string",
        "string_tecnico": "string",
        "marca": "string",
        "modelo": "string",
        "anio": integer,
        "transmision": "MANUAL|AUTO|null",
        "version": "string - integrated technical specs",
        "origen_aseguradora": "string",
        "id_original": "string",
        "version_original": "string",
        "activo": boolean
      }
    ]
  }
}
```

### Processing Logic

1. **Staging**: Load records into temp table with deduplication
2. **Exact Match**: By `id_canonico` → updates availability only
3. **Token Match**: Same `hash_comercial` + overlap ratio on `version_tokens_array`
4. **Similarity thresholds**: Records with scores ≥ 0.92 (same insurer) or ≥ 0.50 (cross insurer) are considered matches
5. **New Record**: Creates new entry when no overlap exceeds thresholds
6. **Conflict Detection**: Reports low-overlap candidates for manual review
   **Returns**: Processing metrics including counts of new/updated/matched records plus warnings/errors

## Token Overlap Strategy

### PostgreSQL Helpers

- **`tokenize_version`**: PL/pgSQL helper that lowercases, strips punctuation, and deduplicates tokens
- **`version_tokens` / `version_tokens_array`**: Stored alongside each catalog row for fast overlap comparisons
- **Index support**: GIN index on `version_tokens` plus optional trigram index for legacy searches

### Matching Algorithm

1. Find exact `hash_comercial` matches (same marca/modelo/anio/transmision)
2. Compute overlap = `|tokens_existing ∩ tokens_incoming| / max(|tokens_existing|, |tokens_incoming|)`
3. Apply configurable thresholds (default 0.92 same insurer, 0.50 cross insurer)
4. For overlaps above threshold: update availability
5. For overlaps below threshold: create new record and log warning

## Development Commands

### Testing n8n Workflows

No automated testing framework - workflows are tested by:

1. Running manually in n8n interface
2. Checking Supabase RPC response for success/error counts
3. Validating output in `catalogo_homologado` table

### Database Operations

Access Supabase via dashboard or direct PostgreSQL connection using credentials in `.env`

**Key Commands:**
```sql
-- Apply homologation functions
\i src/supabase/funciones-homologacion.sql

-- Test batch processing
SELECT procesar_batch_vehiculos('[{"hash_comercial":"...","version_limpia":"..."}]'::jsonb);

-- Check processing results
SELECT origen_aseguradora, COUNT(*) FROM catalogo_homologado GROUP BY origen_aseguradora;
```

### JavaScript Development

**n8n Code Node Testing:**
```javascript
// Test normalization locally (example for Qualitas)
const testRecord = { marca: "TOYOTA", modelo: "CAMRY", anio: 2023, version_original: "XLE AUT 2.5L" };
const normalized = processQualitasRecord(testRecord);
console.log(normalized);
```

### Data Validation

Use MCP agents for data profiling:

```sql
-- Example validation query
SELECT COUNT(*) total,
       COUNT(DISTINCT marca) d_marcas,
       COUNT(DISTINCT modelo) d_modelos
FROM [insurer_table];

-- Token overlap analysis
SELECT hash_comercial,
       array_length(version_tokens_array, 1) as token_count,
       version_tokens_array
FROM catalogo_homologado
WHERE marca = 'TOYOTA' AND modelo = 'CAMRY'
ORDER BY token_count DESC;
```

## Key Principles

### Idempotent Processing

- Re-running the same batch produces identical results
- `id_canonico` serves as unique key preventing duplicates
- RPC function uses UPSERT pattern with conflict resolution

### Data Integrity

- All normalization preserves original data in `version_original` and `id_original`
- Traceability maintained through `string_comercial` and `string_tecnico`
- Audit trail via `fecha_actualizacion` timestamps

### Canonical Normalization

- Consistent marca/modelo standardization across all insurers
- Standardized transmission mapping (1=MANUAL, 2=AUTO, 0=null)
- ALL technical specs integrated into single `version` field as string
- No separate technical specification fields in master catalog
- Token overlap enables cross-insurer compatibility despite format variations
- Empty version stored as empty string, not "BASE" or default values

## Working with New Insurers

1. Create directory: `/src/insurers/[name]/`
2. Analyze source data structure and create `[name]-analisis.md`
3. Develop extraction query: `[name]-query-de-extraccion.sql`
4. Build normalization logic: `[name]-codigo-de-normalizacion.js` following existing patterns
5. Ensure `version` field integrates ALL technical specifications
6. Create complete n8n workflow: `ETL - [Name].json`
7. Test token-overlap scores with existing data before full processing

## Important Notes

- The master catalog has NO separate technical specification fields
- ALL technical details must be integrated into the `version` string
- Token overlap algorithm compares normalized token sets stored with each record
- Los umbrales de similitud son configurables pero típicamente se fijan en 0.92 (misma aseguradora) y 0.50 (entre aseguradoras)
- Hash generation must be consistent across all insurers for proper grouping
- Always preserve original data for audit and debugging purposes
- The system handles missing/null values gracefully - avoid using default placeholders
- Security features (ABS, BA) and occupant info (5OCUP) should be excluded from version normalization

## Implementation Patterns

### Normalization Code Structure

Each insurer's normalization follows a consistent pattern:

1. **Dictionary-based cleaning**: Remove irrelevant tokens (comfort features, audio systems)
2. **Protected token handling**: Preserve hyphenated trims (A-SPEC, TYPE-S, S-LINE) during processing
3. **Engine specification normalization**: Standardize displacement (1.8L, 2.0L TURBO), power (150HP)
4. **Transmission mapping**: Convert codes to MANUAL/AUTO or infer from version text
5. **Hash generation**: SHA-256 of `marca|modelo|anio|transmision` for grouping
6. **Batch processing**: Process in chunks of 5,000 records to manage memory

### Supabase RPC Function Architecture

The `procesar_batch_vehiculos` function implements sophisticated matching:

1. **Multi-score calculation**: Combines Jaccard similarity, token overlap, trigram matching, and Levenshtein distance
2. **Threshold-based decisions**: Different thresholds for same-insurer (0.95) vs cross-insurer (0.35) matches
3. **JSONB availability tracking**: Stores per-insurer metadata including confidence scores and provenance
4. **Conflict resolution**: Handles duplicate exact matches through historical arrays
5. **Error isolation**: Failed records don't break batch processing

### Data Quality Patterns

- **Validation gates**: Records must pass structure validation before normalization
- **Token count requirements**: Minimum token thresholds prevent weak matches
- **Audit trails**: Original data preserved in `version_original` and `id_original` fields
- **Performance indexing**: GIN indexes on `version_tokens_array` for fast token overlap queries

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TendencIA-Systems) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
