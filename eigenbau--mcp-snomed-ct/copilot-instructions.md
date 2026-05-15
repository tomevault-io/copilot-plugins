## mcp-snomed-ct

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SNOMED CT MCP server — provides LLM-accessible tools for searching, retrieving, and navigating SNOMED CT clinical terminology concepts via FHIR R4 terminology servers. Single-file server (`server.py`, ~660 lines) built with FastMCP.

## Running the Server

```bash
# Install dependencies (using venv)
pip install -r requirements.txt

# Run
python server.py
```

The server reads `.env` for configuration. Key variables:
- `SNOMED_BACKEND` — `"local"` (Snowstorm) or `"remote"` (hosted FHIR server)
- `SNOWSTORM_LOCAL_URL` / `FHIR_REMOTE_URL` — backend endpoint
- `SNOMED_DOMAINS` — comma-separated list to restrict searchable domains (default: all 19)

## Architecture

Everything lives in `server.py`. The flow:

1. **`SNOMED_HIERARCHIES` dict** (single source of truth) maps domain names to SNOMED CT root concept IDs and human-readable labels
2. **`SNOMED_DOMAINS` env var** is parsed at startup to filter which domains are active
3. **`SnomedDomain` enum** is dynamically generated from active domains — this is what the LLM sees in the tool schema
4. **Tool descriptions** are dynamically built from the registry so the LLM knows what each domain contains

Request path for lookup: domain name → `_domain_to_ecl()` builds ECL expression (e.g., `<<404684003`) → `_expand()` calls FHIR `ValueSet/$expand` via httpx → `_parse_concept()` extracts code, preferred term, FSN, synonyms from response.

Request path for hierarchy navigation: `snomed_get_related` runs concurrent ECL queries via `_expand_ecl()` — self (`{code}`), parents (`>!{code}`), children (`<!{code}`), then siblings (children of each parent, minus self). Uses `asyncio.gather` for concurrency; siblings are computed in a second phase since they depend on the parents result. Nested ECL (e.g., `<!( >!{code} )`) is avoided because remote FHIR backends don't reliably support it.

### MCP Interface

**Tools:**
- `snomed_lookup` — search by clinical term, scoped to a domain
- `snomed_get_by_code` — fetch full concept details by numeric ID
- `snomed_get_related` — navigate the SNOMED CT hierarchy (parents, children, siblings) for a concept

**Prompts:**
- `normalize_clinical_term` — structured 5-step workflow guiding the LLM to normalize lay/informal language into a verified SNOMED CT concept with confidence scoring
- `explore_concept` — multi-step workflow to explore concepts related to a starting concept, guided by semantic intent (combines hierarchy navigation with targeted searches across domains)

### Key Design Decisions

- FHIR API uses **prefix matching only** — tool descriptions instruct the LLM to convert lay language to formal clinical terminology before searching
- Timeouts differ by backend: 5s local, 15s remote
- No-results responses include structured retry suggestions for the LLM
- All domain metadata (concept IDs, labels, enum members, tool descriptions) derives from the single `SNOMED_HIERARCHIES` dict — add a domain there and everything else follows
- `domain` field has a `mode="before"` Pydantic validator that does **startswith prefix matching** to recover from truncated LLM tool calls (e.g., `"procedur"` → `"procedure"`). Only resolves when exactly one domain matches; ambiguous prefixes are rejected
- `_expand_ecl()` swallows exceptions and returns `([], None)` so that partial failures in concurrent hierarchy queries don't break the entire response
- `snomed_get_related` docstring directs the LLM to the `explore_concept` prompt for semantic (cross-branch) exploration rather than pure hierarchy navigation

## No Tests

There are currently no automated tests. Validation is done via Pydantic models with field validators and regex constraints.

---
> Source: [eigenbau/mcp-snomed-ct](https://github.com/eigenbau/mcp-snomed-ct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
