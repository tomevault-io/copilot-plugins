## intentforge

> IntentForge v2 is a high-performance, resource-efficient discovery engine designed to identify and rank content based on user intent signals. It achieves sub-50ms P95 query latency and <10s trending freshness through a combination of a Rust-based core and specialized microservices.

# IntentForge v2: Instructional Context

IntentForge v2 is a high-performance, resource-efficient discovery engine designed to identify and rank content based on user intent signals. It achieves sub-50ms P95 query latency and <10s trending freshness through a combination of a Rust-based core and specialized microservices.

## Project Overview

- **Purpose:** Intent-first content discovery, crawling, and hybrid search.
- **Core Technology:** Rust (Edition 2021) with Tokio, Axum, and Meilisearch-SDK.
- **Architecture:** 
    - **Rust Core (`intentforge`):** Orchestrates crawling, RSS/GitHub trending monitoring, and indexing.
    - **Extraction Service (`trafilatura`):** A Python/FastAPI microservice for high-quality text extraction from HTML.
    - **Query Layer (`query-layer`):** A Python/FastAPI service providing a semantic search interface using ONNX embeddings.
    - **Inference:** Uses ONNX Runtime with the `all-MiniLM-L6-v2` model for generating 384-dimensional embeddings.
    - **Storage:** 
        - **Meilisearch (v1.13+):** Main search index with support for binary quantization (8x vector compression).
        - **Redis (Redis Stack 7.2):** Used for SimHash deduplication (RedisBloom) and compressed full-text storage (zstd).
    - **Monitoring:** Prometheus for metrics and Grafana for dashboards.

## Building and Running

### Prerequisites
- Rust 1.75+
- Docker & Docker Compose
- Python 3.10+ (for model export and microservices)

### Infrastructure Setup
```bash
# Start Meilisearch, Redis, and microservices
docker-compose up -d

# Initialize Meilisearch index with optimized settings
# (Includes binary quantization and ranking rules)
./scripts/init_meilisearch.sh
```

### Model Preparation
```bash
# Export the ONNX model for embeddings
pip install optimum[onnxruntime] sentence-transformers
python scripts/export_model.py
```

### Development Commands
- **Build:** `cargo build --release`
- **Run:** `cargo run`
- **Test:** `cargo test`
- **Lint:** `cargo fmt` and `cargo clippy`
- **Load Test:** `locust -f tests/load_test.py --host=http://localhost:9100`

## Development Conventions

- **Hybrid Search:** Defaults to a `semanticRatio: 0.7` (70% semantic, 30% keyword).
- **Vector Optimization:** Always use **binary quantization** for embeddings to reduce memory usage and speed up search.
- **Crawler Policy:** 
    - Respects `robots.txt` using the `robotstxt` crate.
    - Static-first fetching (95% no JS) for performance.
    - Adaptive backoff and rate-limiting (default 10 req/s).
- **Logging:** Uses `tracing` and `tracing-subscriber` for structured logging. Set `RUST_LOG=info` for standard operation.
- **Error Handling:** Uses `anyhow` for application-level errors and `thiserror` for library-level errors.
- **Deduplication:** Uses SimHash combined with Bloom filters in Redis for efficient cross-backend deduplication.

## Key Files
- `src/main.rs`: Entry point, orchestrates all Rust components.
- `config.yaml`: Main configuration for crawler, indexer, and inference.
- `sources.yaml`: Defines RSS/Atom feeds for the trending monitor.
- `docker-compose.yml`: Defines the multi-service architecture.
- `scripts/init_meilisearch.sh`: Critical for setting up the Meilisearch index with the correct vector settings.
- `services/trafilatura/app.py`: Content extraction logic.
- `services/query_layer/app.py`: Semantic search implementation.

## API Ports
- **Rust API:** `9100` (Search, Crawl, Health, Metrics)
- **Query Layer:** `8000` (Semantic Search)
- **Extraction Service:** `8080` (Batch Extraction)
- **Meilisearch:** `7700`
- **Redis:** `6379` (Data), `8001` (Insight)
- **Prometheus:** `9090`
- **Grafana:** `3000`

---
> Source: [oxiverse-labs/intentforge](https://github.com/oxiverse-labs/intentforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
