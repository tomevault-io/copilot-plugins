## web-scraper

> This document describes the agent responsibilities and workflows for the Hungarian Health Portal web scraping project. It defines how different components work together to achieve RAG-optimized data extraction.

# AGENTS.md

## Hungarian Health Portal Web Scraper - Agent Architecture

This document describes the agent responsibilities and workflows for the Hungarian Health Portal web scraping project. It defines how different components work together to achieve RAG-optimized data extraction.

---

## 1. Overview

The scraper uses a **multi-agent architecture** where specialized components handle different aspects of the scraping and processing pipeline:

```
┌─────────────────────────────────────────────────────────────────┐
│                    HealthScraper (Orchestrator)                  │
│  • Coordinates all agents                                        │
│  • Manages configuration and state                               │
│  • Handles output and error reporting                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      JinaClient (Extractor)                      │
│  • Handles Jina AI API communication                             │
│  • Rate limiting and retry logic                                 │
│  • Content extraction from web pages                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ContentProcessor (Analyzer)                   │
│  • Cleans and normalizes content                                 │
│  • Semantic chunking for RAG                                     │
│  • Summary generation                                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   TOONConverter (Formatter)                      │
│  • Converts JSON to TOON format                                  │
│  • Token optimization for LLM processing                         │
│  • Output formatting                                             │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Agent Descriptions

### 2.1 HealthScraper (Orchestrator Agent)

**Purpose**: Main coordinator for the entire scraping pipeline.

**Responsibilities**:
- Initialize and configure all other agents
- Manage crawl state (visited URLs, queue, depth tracking)
- Coordinate the workflow between agents
- Handle errors and recovery
- Generate output reports

**Input**:
- Configuration (from `.env` or environment variables)
- Command-line arguments

**Output**:
- Scraped pages (ScrapedPage objects)
- Processed RAG documents
- Output files (JSON, TOON)
- Error reports

**Workflow**:
```python
# Initialize
config = Config.load()
scraper = HealthScraper(config)

# Crawl phase
pages = scraper.crawl(start_url, max_pages, max_depth)

# Process phase  
output_files = scraper.process_results(pages)
```

**Error Handling**:
- Graceful degradation on individual page failures
- Retry logic for transient errors
- Fallback to error page representation

---

### 2.2 JinaClient (Extractor Agent)

**Purpose**: Handle all communication with the Jina AI Reader API.

**Responsibilities**:
- API authentication and request signing
- Rate limiting to respect API constraints
- Retry logic with exponential backoff
- Error translation and handling
- Response parsing and validation

**Input**:
- URLs to extract
- Extraction options (raw, info, links)

**Output**:
- Extracted content (markdown format)
- Page metadata (title, language, content type)
- Discovered links

**Configuration**:
```python
JinaConfig(
    api_key="your-api-key",
    base_url="https://api.jina.ai",
    timeout=30,
    max_retries=3,
    retry_delay=1.0,
)
```

**Rate Limiting**:
- Default: 60 requests per minute
- Configurable via `REQUESTS_PER_MINUTE` env var

**Error Handling**:
- `JinaClientError` for API failures
- Automatic retry on 5xx errors
- Immediate failure on 401 (auth error)
- Backoff on 429 (rate limit)

---

### 2.3 ContentProcessor (Analyzer Agent)

**Purpose**: Transform raw content into RAG-optimized chunks.

**Responsibilities**:
- Content cleaning and normalization
- Semantic section extraction
- Token-aware chunking
- Summary generation
- Metadata enrichment

**Input**:
- Raw markdown content
- URL and page metadata

**Output**:
- RAGChunk objects
- RAGDocument objects
- Token counts
- Section boundaries

**Processing Pipeline**:

```
Raw Content
    │
    ▼
┌───────────────┐
│  Clean/       │ → Remove excessive whitespace
│  Normalize    │ → Fix encoding issues
└───────────────┘
    │
    ▼
┌───────────────┐
│  Extract      │ → Find markdown headings
│  Sections     │ → Identify semantic sections
└───────────────┘
    │
    ▼
┌───────────────┐
│  Generate     │ → Create 2-3 sentence summary
│  Summary      │ → Capture key information
└───────────────┘
    │
    ▼
┌───────────────┐
│  Chunk        │ → Split by token limit
│  Content      │ → Preserve semantic boundaries
└───────────────┘
    │
    ▼
RAGChunks
```

**Configuration**:
```python
ContentProcessor(
    chunk_size_tokens=512,  # Max tokens per chunk
    chunk_overlap=50,        # Overlap between chunks
    language="hu",          # Content language
)
```

---

### 2.4 TOONConverter (Formatter Agent)

**Purpose**: Convert processed data to TOON format for optimal LLM processing.

**Responsibilities**:
- JSON to TOON conversion
- Token optimization
- Format validation
- File output

**Input**:
- JSON data (from ContentProcessor)
- Document collections

**Output**:
- TOON-formatted strings
- TOON files (.toon extension)

**TOON Format Advantages**:
- ~50% fewer tokens vs JSON
- Explicit structure for LLMs
- Human-readable
- Lossless conversion

**Example Conversion**:

```json
// JSON (152 tokens)
{
  "health_resources": [
    {"name": "Hospital A", "type": "emergency", "phone": "123"},
    {"name": "Hospital B", "type": "emergency", "phone": "456"}
  ]
}

// TOON (76 tokens - 50% savings!)
health_resources[2]{name,type,phone}:
  "Hospital A",emergency,123
  "Hospital B",emergency,456
```

---

## 3. Workflow Coordination

### 3.1 Main Pipeline Flow

```
1. Initialization
   ├── Load configuration
   ├── Validate API key
   └── Initialize agents

2. Crawl Phase
   ├── Start at base URL
   ├── Discover links
   ├── Extract content (JinaClient)
   └── Track visited URLs

3. Process Phase
   ├── Clean content (ContentProcessor)
   ├── Generate chunks
   ├── Create summaries
   └── Convert to TOON (TOONConverter)

4. Output Phase
   ├── Save raw JSON
   ├── Save TOON files
   ├── Log statistics
   └── Report errors
```

### 3.2 Error Handling Flow

```
Page Error
    │
    ▼
┌─────────────────┐
│  Log Error      │ → Record in failed_urls
└─────────────────┘
    │
    ▼
┌─────────────────┐
│  Continue       │ → Proceed with other pages
│  Crawling       │
└─────────────────┘
    │
    ▼
┌─────────────────┐
│  Final Report   │ → List failed URLs
└─────────────────┘
```

---

## 4. Configuration Management

### 4.1 Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `JINA_API_KEY` | Yes | - | Jina AI API key |
| `BASE_URL` | No | Health portal URL | Starting URL |
| `MAX_DEPTH` | No | 3 | Maximum crawl depth |
| `MAX_PAGES` | No | Unlimited | Maximum pages to scrape |
| `CHUNK_SIZE_TOKENS` | No | 512 | RAG chunk size |
| `CHUNK_OVERLAP` | No | 50 | Chunk overlap |
| `LOG_LEVEL` | No | INFO | Logging level |
| `REQUESTS_PER_MINUTE` | No | 60 | API rate limit |

### 4.2 .env File

```bash
# Copy .env.example to .env and fill in your values
JINA_API_KEY=your-api-key-here
BASE_URL=https://egeszsegvonal.gov.hu/hova-forduljak.html
MAX_DEPTH=3
CHUNK_SIZE_TOKENS=512
LOG_LEVEL=INFO
```

---

## 5. Performance Considerations

### 5.1 Rate Limiting

The scraper respects Jina API rate limits:
- Default: 60 requests/minute
- Configurable via environment
- Automatic backoff on 429 responses

### 5.2 Memory Management

- Streaming processing (no full-page loading)
- Chunk-based processing
- Periodic garbage collection hints

### 5.3 Progress Tracking

- Real-time logging (every 10 pages)
- Final statistics report
- Failed URL tracking

---

## 6. Scaling Considerations

### 6.1 Horizontal Scaling

For large-scale crawling:
- Run multiple instances with different URL ranges
- Use distributed queue (Redis, RabbitMQ)
- Coordinate via shared state

### 6.2 Vertical Scaling

For faster single-instance processing:
- Increase `REQUESTS_PER_MINUTE` (within limits)
- Reduce `CHUNK_SIZE_TOKENS` for faster processing
- Use faster machine/instance

---

## 7. Monitoring and Observability

### 7.1 Logging

- Structured JSON logs for machine parsing
- Human-readable logs for debugging
- Configurable verbosity

### 7.2 Metrics

Tracked metrics:
- Pages scraped successfully
- Pages failed
- Total chunks generated
- Total tokens processed
- API latency
- Error rates

### 7.3 Health Checks

- Jina API connectivity check
- Configuration validation
- Output directory validation

---

## 8. Security Considerations

### 8.1 API Key Management

- Store in `.env` file (never commit)
- Use environment variables in production
- Rotate keys periodically

### 8.2 Data Privacy

- No PII in logs
- Secure output directories
- Data retention policies

---

## 9. Future Enhancements

Potential agent improvements:
- **EmbeddingAgent**: Generate vector embeddings
- **DeduplicationAgent**: Remove duplicate content
- **QualityAgent**: Validate extracted data quality
- **SchedulerAgent**: Periodic rescrawling
- **MonitoringAgent**: Real-time dashboards

---

## 10. Quick Start

```bash
# 1. Set up environment
cp .env.example .env
# Edit .env with your JINA_API_KEY

# 2. Run scraper
python src/main.py

# 3. Check output
ls data/toon/
cat data/toon/*.toon
```

---

**Document Version**: 1.0.0  
**Last Updated**: 2026-01-14  
**Project**: Hungarian Health Portal Web Scraper

---
> Source: [kevsuin/web-scraper](https://github.com/kevsuin/web-scraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
