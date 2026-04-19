## test

> - @task/ folder contains the description of the problem this project is solving with all extra files for the context.

### Description
- @task/ folder contains the description of the problem this project is solving with all extra files for the context.

### Requirements
- the solution has to be robust and battle tested on every edge-case

### Tools
1. Python `email` module (stdlib) - parse .eml files, extract MIME parts, headers, attachments
2. Docling (https://github.com/docling-project/docling) - convert DOCX/PPTX attachments to PDF/images for Gemini processing
3. Gemini 3.0 Flash API (google-genai) - agentic vision for PO extraction from images/PDFs, email body parsing, structured JSON output, validation cross-check
4. RapidFuzz (https://github.com/rapidfuzz/RapidFuzz) - fuzzy SKU and product title matching in entity resolution cascade
5. sentence-transformers (intfloat/e5-base-v2) - local embedding model for pre-filtering Gemini candidates with disk-cached embeddings
6. Supabase (supabase-py) - database client for reading product catalog and writing validated PO data
7. Pydantic - structured data models and validation at every pipeline boundary

### Architecture
- 5-stage pipeline: Email Parser → Format Router → Gemini Extractor → Entity Resolver → Validation Agent → Output
- Hybrid entity resolution: deterministic DB lookups (exact → supplier_sku → RapidFuzz fuzzy) → Gemini semantic fallback (pre-filtered by embedding candidates)
- Domain-based supplier matching as fallback when exact email match fails
- Self-learning: writes back learned SKU aliases to DB after AI resolution, enriches embedding model with known aliases
- Two-layer validation: deterministic business rules + Gemini cross-validation
- Agent-in-the-loop with human escalation for unresolvable items

### Project-specific rules
- ignore @archive/ folder

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/morozovdd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
