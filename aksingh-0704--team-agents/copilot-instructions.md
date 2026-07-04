## team-agents

> AI-powered platform covering the full health insurance lifecycle:

# PolicyAI — Health Insurance Intelligence Platform

## Project Overview
AI-powered platform covering the full health insurance lifecycle:
1. Natural Language Policy Discovery (find the right policy)
2. Conversational Policy Comparison (side-by-side comparison)
3. Hybrid RAG Policy Q&A + Hidden Conditions Detector (the unique feature)
4. Medical Report → Smart Policy Matching
5. Existing Policy Upload → Claim Advisory
6. Coverage Gap Analyzer

## Tech Stack
- **Frontend:** Next.js 14 + Tailwind CSS → deployed on Vercel
- **Backend:** FastAPI (Python 3.11) → deployed on Render
- **Vector DB:** Supabase pgvector (1536-dim embeddings)
- **Keyword Search:** PostgreSQL tsvector (GIN index) in same Supabase DB
- **Embeddings:** OpenAI text-embedding-3-small
- **LLM:** OpenAI GPT-4o-mini (structured JSON output)
- **PDF Parsing:** PyMuPDF (fitz)
- **RAG:** Hybrid (semantic + keyword via RRF fusion) + section-aware enrichment

## Key Commands
```bash
# Frontend
cd frontend && npm install && npm run dev

# Backend
cd backend && pip install -r requirements.txt
uvicorn main:app --reload --port 8000

# Data seeding (run once)
cd backend && python scripts/seed_db.py         # Seed structured catalog
cd backend && python scripts/startup_seeder.py  # Embed all PDFs from policies/ folder

# Tests
cd backend && python -m pytest tests/
```

## Environment Variables
```
# backend/.env
OPENAI_API_KEY=sk-...
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_SERVICE_KEY=eyJ...

# frontend/.env.local
NEXT_PUBLIC_API_URL=http://localhost:8000
```

## Key File Paths
```
backend/
  main.py                    — FastAPI app entry point + lifespan startup seeder
  services/
    tools.py                 — 9 tool implementations + OpenAI function-call schemas
    skills.py                — HiddenConditionsDetector, CoverageGapScanner, PolicyRanker
    agents.py                — 5 agent definitions (system prompts + tool lists)
    pdf_parser.py            — Section-aware PDF chunking (PyMuPDF)
    embedder.py              — OpenAI embedding wrapper with retry
    vector_store.py          — Supabase: semantic_search, keyword_search, rrf_fusion, section_search
    medical_extractor.py     — MedicalExtractorAgent (conditions from text/PDF)
    llm.py                   — GPT-4o-mini structured response helpers
  routers/
    discovery.py             — /api/discover, /api/compare
    qa.py                    — /api/upload, /api/policies, /api/ask
    claim.py                 — /api/claim-check, /api/extract-conditions, /api/match-conditions, /api/gap-analysis/{id}
  data/
    seed_policies.json       — 10-insurer structured catalog metadata
  scripts/
    seed_db.py               — Populate insurance_policies catalog table
    startup_seeder.py        — Scan policies/ dir → embed all PDFs

frontend/
  app/
    page.tsx                 — Landing / mode selector
    discover/page.tsx        — Feature 1+2: Discovery + Comparison
    qa/page.tsx              — Feature 3: Policy Q&A
    claim/page.tsx           — Feature 5+6: Claim Advisory + Gap Analysis
  components/
    AnswerCard.tsx           — Shows verdict, GREEN/AMBER/RED badge, hidden conditions
    HiddenConditionBadge.tsx — Individual hidden trap card
  lib/api.ts                 — All API client functions

policies/                    — Real PDF policy wordings (organized by insurer)
  tata/                      — 30 Tata AIG PDFs (already downloaded)
```

## Policy PDF Folder Convention
```
policies/{insurer_slug}/{policy_filename}.pdf
```
The startup seeder scans this folder recursively. To add new insurer PDFs:
1. Create `policies/{insurer_name}/` folder
2. Drop PDFs in — seeder handles the rest on next startup

## Supabase Tables
- `insurance_policies` — structured catalog (premiums, waiting periods, exclusions)
- `uploaded_policies` — tracks embedded PDF documents
- `policy_chunks` — text chunks with `embedding VECTOR(1536)` + `content_tsv TSVECTOR` + `section_type`

## RAG Architecture (Hybrid — 3 Layer)
1. **Layer 1:** Hybrid search = semantic (pgvector) + keyword (tsvector) → RRF fusion → top-5
2. **Layer 2:** Section-filtered search for `definitions` section → top-3
3. **Layer 3:** Section-filtered search for `exclusions + conditions + limits` → top-3
4. **Synthesis:** GPT-4o-mini with Hidden Conditions Detector prompt → structured JSON

## Hidden Conditions Types
room_rent_trap | pre_auth_required | proportional_deduction | definition_trap |
waiting_period | sub_limit | documentation | network_restriction

## Practical Claimability
- GREEN — Clearly covered, straightforward claim
- AMBER — Technically covered but with conditions that complicate claiming
- RED — Not covered or likely to be denied

---
> Source: [AKSINGH-0704/Team_Agents](https://github.com/AKSINGH-0704/Team_Agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
