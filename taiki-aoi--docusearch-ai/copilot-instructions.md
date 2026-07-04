## docusearch-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DocuSearch_AI is a RAG (Retrieval-Augmented Generation) platform that transforms unstructured data (documents and images) into structured, searchable knowledge. The system enables natural language semantic search over various data sources.

**Deployment Pattern:** Pattern C (Lightweight Hybrid) - Docker Desktop + Gemini API

## Commands

### Start all services
```bash
docker compose up -d
```

### Stop services
```bash
docker compose down
```

### View logs
```bash
docker compose logs -f [service_name]
```

### Python scripts (in scripts/ directory)
```bash
# Install dependencies
pip install -r scripts/requirements.txt

# Test EXIF extraction
python scripts/exif_extractor.py <image_file>

# Test geocoding
python scripts/geocoder.py <latitude> <longitude>

# Process image (full pipeline)
python scripts/image_processor.py <image_file>
```

## Architecture

The system consists of three layers:

### 1. Orchestration Layer (n8n)
- Self-hosted workflow automation engine
- Monitors external data sources (Dropbox, local folders)
- Routes files by type (images vs documents)
- Coordinates API calls to Vision AI and RAG engine

### 2. Cognitive Processing Layer (Dify + LLM)
- **RAG Engine:** Dify (Open Source)
- **Vision Analysis:** Gemini 2.5 Flash
- **Vector Database:** Weaviate
- Handles chunking, embedding, semantic search, and answer generation

### 3. Data Source Layer
- Supported documents: PDF, DOCX, XLSX, TXT, MD
- Supported images: JPG, PNG (with EXIF metadata extraction)

## Key Files

| File | Purpose |
|------|---------|
| `docker-compose.yml` | All service definitions |
| `.env.example` | Environment variables template |
| `nginx/nginx.conf` | Reverse proxy configuration |
| `scripts/exif_extractor.py` | EXIF metadata extraction from images |
| `scripts/geocoder.py` | GPS to address conversion |
| `scripts/image_processor.py` | Combined image processing pipeline |

## Service Ports

| Service | Port | URL |
|---------|------|-----|
| Dify Web | 3000 | http://localhost:3000 |
| Dify API | 5001 | http://localhost:5001 |
| n8n | 5678 | http://localhost:5678 |
| Weaviate | 8080 | http://localhost:8080 |
| Nginx (unified) | 80 | http://localhost |

## Environment Variables

Required variables (copy `.env.example` to `.env`):
- `POSTGRES_PASSWORD` - Database password
- `REDIS_PASSWORD` - Redis password
- `DIFY_SECRET_KEY` - Dify secret key (32+ chars)
- `N8N_ENCRYPTION_KEY` - n8n encryption key (24+ chars)
- `GEMINI_API_KEY` - Gemini API key for vision analysis

## Document Processing Pipeline

The system processes files differently based on type:

### Text Files (.txt, .md, .csv, etc.)

- Direct text extraction
- Uploaded via Dify `create-by-text` API

### PDF Files (including scanned documents)

- **Gemini 2.5 Flash Vision API** processes PDFs directly
- Extracts text from both native PDFs and scanned image-based PDFs
- OCR is handled by Gemini (no additional OCR service needed)
- Extracted text uploaded via Dify `create-by-text` API

### Other Binary Files (.docx, .xlsx, etc.)

- Uploaded via Dify `create_by_file` API
- Dify handles parsing internally

### n8n Workflow: Document Processing

Location: `n8n/workflows/document-processing.json`

Flow:

```text
File → ドキュメント準備 → テキスト判定
                           ├─ Text → Difyテキスト登録
                           └─ Not Text → PDF判定
                                          ├─ PDF → Gemini PDF処理 → DifyにPDFテキスト登録
                                          └─ Binary → Difyファイル登録
```

## Image Processing Pipeline

1. **EXIF Extraction** (`exif_extractor.py`)
   - Extracts datetime, GPS coordinates, camera info
   - Converts GPS from DMS to decimal degrees

2. **Geocoding** (`geocoder.py`)
   - Converts GPS coordinates to addresses
   - Supports Nominatim (free) and Google Maps API

3. **Vision Caption** (`image_processor.py`)
   - Sends image to Gemini 2.5 Flash
   - Generates detailed Japanese description for search

4. **Indexing**
   - Combines metadata + caption into document
   - Uploads to Dify Knowledge Base

## Important Configuration Notes

### Embedding Model (Critical for Search Quality)

- **Recommended:** `gemini-embedding-001` for Dify Knowledge Base
- Default embedding models may have poor Japanese search accuracy
- Configure in: Dify → Knowledge → Settings → Embedding Model
- This significantly improves semantic search quality for Japanese content

## Design Documents

All specifications are in Japanese:
- `設計書/01_System_Architecture.md` - System components and data processing logic
- `設計書/02_Deployment_Patterns.md` - Three deployment tier patterns
- `設計書/03_Implementation_Specs.md` - Docker Compose config, n8n workflows, optimization
- `設計書/04_Implementation_Plan.md` - Implementation plan and phases

---
> Source: [taiki-aoi/DocuSearch_AI](https://github.com/taiki-aoi/DocuSearch_AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
