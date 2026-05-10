## burnsville-mn-analysis

> Large-scale analysis and audit of public documents from Burnsville, MN's CivicWeb document site. Extract, store, and analyze textual content from various document formats using LLM-powered insights.

# Burnsville MN CivicWeb Document Analysis Project

## Overview
Large-scale analysis and audit of public documents from Burnsville, MN's CivicWeb document site. Extract, store, and analyze textual content from various document formats using LLM-powered insights.

**Source**: `https://burnsville.civicweb.net/document/{id}/`
**Document Types**: HTML, PDF, Images
**Document IDs**: Pre-collected list (provided by user)

## Architecture: Dockerized RAG Pipeline

### System Design
```
┌─────────────────────────────────────────────────────────────┐
│                     Docker Compose Stack                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Postgres   │  │   ChromaDB   │  │  Processing  │      │
│  │   Database   │  │    Vector    │  │   Worker     │      │
│  │              │  │    Store     │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Analysis API / Interface                 │   │
│  │          (FastAPI + Claude RAG System)                │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    ┌──────────────┐
                    │  Claude API  │
                    │   (External) │
                    └──────────────┘
```

### Components

1. **PostgreSQL** - Primary data store
   - Document metadata
   - Extracted text
   - Analysis results
   - Better than SQLite for this scale and concurrent access

2. **ChromaDB** - Vector database
   - Document embeddings
   - Semantic search
   - RAG context retrieval

3. **Processing Worker** - Python service
   - Document download
   - Text extraction (PDF, HTML, images)
   - Tesseract OCR included in container
   - Embedding generation
   - Queue-based processing

4. **Analysis API** - FastAPI application
   - RESTful API for queries
   - Interactive chat interface
   - Batch analysis endpoints
   - Web UI for exploration

## Why This Approach

### Advantages
- **Reproducible**: Docker ensures consistent environment
- **Scalable**: Can process documents in parallel
- **Production-Ready**: Easy to deploy, monitor, and maintain
- **Isolated**: OCR and dependencies containerized
- **Queryable**: SQL + vector search capabilities
- **API-First**: Can build multiple frontends (CLI, web, notebooks)

### Tech Stack

```yaml
Services:
  postgres:14-alpine      # Lightweight, fast, reliable
  chromadb/chroma:latest  # Official ChromaDB image
  python:3.11-slim        # Processing worker + API

Python Libraries:
  # Document Processing
  - pymupdf               # PDF extraction
  - pytesseract           # OCR wrapper
  - beautifulsoup4        # HTML parsing
  - html2text             # HTML to text
  - pillow                # Image processing
  - pdf2image             # PDF to images for OCR

  # Storage
  - psycopg2-binary       # PostgreSQL driver
  - chromadb              # Vector DB client
  - sqlalchemy            # ORM

  # API & Processing
  - fastapi               # API framework
  - uvicorn               # ASGI server
  - celery                # Task queue (optional)
  - httpx                 # Async HTTP

  # LLM Integration
  - anthropic             # Claude API
  - tiktoken              # Token counting

  # Utilities
  - python-magic          # File type detection
  - tqdm                  # Progress tracking
  - pydantic              # Data validation
```

## Data Pipeline

### Stage 1: Document Download
```
Input: document_ids.txt (list of IDs)

Process:
  1. Read document IDs from file
  2. For each ID, fetch from https://burnsville.civicweb.net/document/{id}/
  3. Detect content type (HTML/PDF/Image)
  4. Save to /data/raw/{id}/
  5. Insert metadata into PostgreSQL

Database: documents table
  - id, url, content_type, file_path,
    download_timestamp, status, error_message
```

### Stage 2: Text Extraction
```
Process by type:

HTML:
  - Parse with BeautifulSoup
  - Extract main content
  - Remove scripts, styles, navigation
  - Convert to clean markdown

PDF:
  - Attempt direct text extraction with PyMuPDF
  - Calculate text density per page
  - If density < threshold, convert to images
  - Run Tesseract OCR on images
  - Combine text with page numbers

Images:
  - Preprocess (deskew, contrast)
  - Run Tesseract OCR
  - Extract text with confidence scores
  - Store low-confidence pages for review

Output: text_content in PostgreSQL + /data/text/{id}.txt
```

### Stage 3: Chunking & Embedding
```
Process:
  1. Split text into semantic chunks (~1000 tokens)
  2. Preserve document structure (pages, sections)
  3. Add overlap between chunks (200 tokens)
  4. Generate embeddings:
     - Option A: Local model (sentence-transformers)
     - Option B: Claude embeddings API
  5. Store in ChromaDB with metadata

Metadata per chunk:
  - document_id
  - chunk_index
  - page_numbers
  - section_title
  - token_count
  - embedding_model
```

### Stage 4: Analysis & Querying
```
RAG Query Flow:
  1. User submits question
  2. Embed question using same model
  3. Query ChromaDB for top-k relevant chunks
  4. Retrieve full context from PostgreSQL
  5. Construct prompt with context
  6. Send to Claude API
  7. Return answer with source citations

Batch Analysis:
  1. Define analysis task (JSON config)
  2. Query relevant documents
  3. Process in batches (rate limiting)
  4. Store results in analysis_results table
  5. Generate summary report
```

## Project Structure

```
burnsville-mn-analysis/
├── CLAUDE.md
├── README.md
├── docker-compose.yml        # Service orchestration
├── .env.example
├── document_ids.txt          # List of IDs to process
│
├── data/
│   ├── raw/                  # Mounted volume for downloads
│   ├── text/                 # Extracted text files
│   ├── postgres/             # PostgreSQL data (volume)
│   └── chroma/               # ChromaDB data (volume)
│
├── worker/                   # Processing service
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py              # Entry point
│   ├── downloaders/
│   │   └── civicweb.py
│   ├── extractors/
│   │   ├── pdf_extractor.py
│   │   ├── html_extractor.py
│   │   └── image_ocr.py
│   ├── storage/
│   │   ├── postgres.py
│   │   └── vectordb.py
│   └── utils/
│       ├── chunking.py
│       └── config.py
│
├── api/                      # FastAPI service
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py
│   ├── routers/
│   │   ├── query.py         # RAG queries
│   │   ├── analysis.py      # Batch analysis
│   │   └── documents.py     # Document management
│   ├── services/
│   │   ├── rag.py           # RAG implementation
│   │   └── claude.py        # Claude API wrapper
│   └── models/
│       └── schemas.py       # Pydantic models
│
├── scripts/
│   ├── init_db.sql          # PostgreSQL schema
│   ├── run_pipeline.sh      # Full pipeline runner
│   └── test_extraction.py   # Test on sample docs
│
└── notebooks/
    └── exploration.ipynb     # Jupyter for analysis
```

## Database Schema

### PostgreSQL Tables

```sql
-- Documents metadata
CREATE TABLE documents (
    id VARCHAR PRIMARY KEY,              -- Document ID from CivicWeb
    url TEXT NOT NULL,
    content_type VARCHAR(50),            -- html, pdf, image
    title TEXT,
    file_path TEXT,
    download_timestamp TIMESTAMP,
    extraction_timestamp TIMESTAMP,
    status VARCHAR(50),                  -- pending, processing, completed, failed
    error_message TEXT,
    page_count INTEGER,
    word_count INTEGER,
    extraction_method VARCHAR(50)        -- text, ocr, hybrid
);

-- Extracted text with chunks
CREATE TABLE text_chunks (
    id SERIAL PRIMARY KEY,
    document_id VARCHAR REFERENCES documents(id),
    chunk_index INTEGER,
    text_content TEXT,
    page_numbers INTEGER[],
    section_title TEXT,
    token_count INTEGER,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Analysis results
CREATE TABLE analysis_results (
    id SERIAL PRIMARY KEY,
    analysis_name VARCHAR(255),
    document_id VARCHAR REFERENCES documents(id),
    analysis_type VARCHAR(100),
    result JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Processing queue
CREATE TABLE processing_queue (
    id SERIAL PRIMARY KEY,
    document_id VARCHAR,
    task_type VARCHAR(50),              -- download, extract, embed
    status VARCHAR(50),                  -- pending, processing, completed, failed
    priority INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    error_message TEXT
);

-- Indexes
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_chunks_document ON text_chunks(document_id);
CREATE INDEX idx_queue_status ON processing_queue(status, priority);
```

## Docker Compose Configuration

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: burnsville_docs
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - ./data/postgres:/var/lib/postgresql/data
      - ./scripts/init_db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  chromadb:
    image: chromadb/chroma:latest
    volumes:
      - ./data/chroma:/chroma/chroma
    environment:
      - CHROMA_SERVER_AUTH_CREDENTIALS=${CHROMA_AUTH}
    ports:
      - "8000:8000"

  worker:
    build: ./worker
    environment:
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/burnsville_docs
      - CHROMA_HOST=chromadb
      - CHROMA_PORT=8000
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    volumes:
      - ./data/raw:/app/data/raw
      - ./data/text:/app/data/text
      - ./document_ids.txt:/app/document_ids.txt
    depends_on:
      postgres:
        condition: service_healthy
      chromadb:
        condition: service_started
    command: python main.py process

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/burnsville_docs
      - CHROMA_HOST=chromadb
      - CHROMA_PORT=8000
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - chromadb
    command: uvicorn main:app --host 0.0.0.0 --port 8080
```

## Implementation Phases

### Phase 1: Infrastructure Setup
```bash
# Create project structure
# Write docker-compose.yml
# Create Dockerfiles
# Set up PostgreSQL schema
# Configure environment variables
```

### Phase 2: Document Processing
```bash
# Implement downloader
# Build extractors (PDF, HTML, image)
# Set up processing pipeline
# Test on small batch (10 docs)
```

### Phase 3: Storage & Indexing
```bash
# Store documents in PostgreSQL
# Implement chunking strategy
# Generate embeddings
# Index in ChromaDB
```

### Phase 4: Analysis Layer
```bash
# Build RAG query system
# Create FastAPI endpoints
# Implement batch analysis
# Add simple web UI
```

### Phase 5: Production Hardening
```bash
# Add error handling and retries
# Implement rate limiting
# Add monitoring/logging
# Create backup scripts
```

## Usage

### Starting the System
```bash
# Set up environment
cp .env.example .env
# Edit .env with API keys

# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f worker
```

### Processing Documents
```bash
# Run full pipeline on all documents
docker-compose exec worker python main.py process --all

# Process specific batch
docker-compose exec worker python main.py process --batch 0-100

# Reprocess failed documents
docker-compose exec worker python main.py reprocess-failed
```

### Querying Documents
```bash
# API endpoints
curl http://localhost:8080/query -d '{"question": "What are the main themes in 2024 documents?"}'

# Interactive chat
curl http://localhost:8080/chat -d '{"message": "Summarize budget discussions"}'

# Batch analysis
curl http://localhost:8080/analyze -d '{"analysis_type": "theme_extraction", "filters": {"year": 2024}}'
```

### Analysis Examples

1. **Thematic Analysis**: Identify recurring topics
2. **Entity Extraction**: Find people, orgs, locations
3. **Compliance Checks**: Search for required disclosures
4. **Temporal Trends**: Track topics over time
5. **Document Comparison**: Find similar documents
6. **Anomaly Detection**: Flag unusual documents

## Configuration

### Environment Variables (.env)
```bash
POSTGRES_USER=burnsville
POSTGRES_PASSWORD=secure_password_here
ANTHROPIC_API_KEY=sk-ant-...
CHROMA_AUTH=token_here

# Processing settings
MAX_CONCURRENT_DOWNLOADS=5
OCR_THRESHOLD=0.5
CHUNK_SIZE=1000
CHUNK_OVERLAP=200

# LLM settings
CLAUDE_MODEL=claude-sonnet-4-5-20250929
MAX_CONTEXT_CHUNKS=10
```

## Cost Estimation

### Infrastructure
- All local, Docker-based
- No cloud costs

### Storage (1000 documents)
- Raw files: ~5GB
- PostgreSQL: ~500MB
- ChromaDB: ~100MB
- Total: ~6GB

### API Costs
- Embeddings: Use local model (free) or ~$0.0001/doc
- Queries: ~$0.001-0.01 per query
- Batch analysis: Depends on depth

## Next Steps

1. Wait for document_ids.txt upload
2. Create project structure
3. Write docker-compose.yml
4. Implement downloader
5. Build extractors
6. Test on sample documents
7. Scale to full corpus
8. Build analysis interface

---
> Source: [AnthonyHerman/burnsville-mn-analysis](https://github.com/AnthonyHerman/burnsville-mn-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
