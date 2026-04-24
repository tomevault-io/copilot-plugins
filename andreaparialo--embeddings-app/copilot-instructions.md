## embeddings-app

> This is a high-performance visual product search engine that processes 29,136 product images using state-of-the-art AI models on 4x NVIDIA A100 GPUs.

# GPU-Accelerated Visual Product Search System

This is a high-performance visual product search engine that processes 29,136 product images using state-of-the-art AI models on 4x NVIDIA A100 GPUs.

## Core Technologies

- **Framework**: FastAPI (Python web framework)
- **AI Models**: GME-Qwen2-VL-7B with LoRA adapters + OpenCLIP
- **Vector Search**: FAISS (Facebook AI Similarity Search) with GPU acceleration
- **GPUs**: 4x NVIDIA A100 (40GB each)
- **Database**: Pandas DataFrames with enriched AWS shape data
- **Frontend**: Jinja2 templates with modern UI

## Key Features

1. **Multiple Search Modes**:
   - Image similarity search (upload image → find similar products)
   - SKU code search (exact/partial matching)
   - Filter-based search (brand, gender, product type, etc.)
   - Batch processing via Excel upload (100+ images/second)

2. **Advanced Filtering**:
   - Pre-filtering before FAISS search for efficiency
   - Post-filtering for fine-grained control
   - Range-based filtering for numeric values
   - Baseline filters (status codes, release dates)

3. **Performance Optimizations**:
   - Batch processing: 1000x faster than sequential
   - GPU acceleration: 10-50x faster than CPU
   - Float16 precision: 50% memory reduction
   - Multi-GPU distribution across 4 A100s

## Project Structure

```
old_app/
├── app.py                      # Main FastAPI application
├── search_engine.py           # Core search orchestration
├── data_loader.py             # FAISS index and CSV data management
├── gme_model.py               # GME-Qwen2-VL model wrapper
├── batch_processor_optimized.py # Pre-filtering batch processor
├── optimized_faiss_search.py # Pre-filtering FAISS search
├── faiss_gpu_utils.py         # GPU management utilities
├── config_filtering.py        # Centralized filter configuration
├── indexes/                   # FAISS indexes (369MB+)
├── pictures/                  # 29,104 product images
├── loras/                     # LoRA checkpoints
└── database_results/          # Enriched product database
```

## Critical Components

- **Main Entry**: [app.py](mdc:app.py) - FastAPI server with all endpoints
- **Search Logic**: [search_engine.py](mdc:search_engine.py) - Orchestrates all search types
- **Data Loading**: [data_loader.py](mdc:data_loader.py) - Manages 34,431 SKUs → 29,136 embeddings mapping
- **GPU Utilities**: [faiss_gpu_utils.py](mdc:faiss_gpu_utils.py) - Handles GPU transfers and memory

## Running the Application

```bash
# Ensure conda environment is activated
conda activate faiss_env

# Start with GPU acceleration
./start_gpu.sh

# Or start with specific options
python app.py  # Default
./start_openclip_gpu.sh  # OpenCLIP variant
```

The application runs at `http://127.0.0.1:8080`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andreaparialo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
