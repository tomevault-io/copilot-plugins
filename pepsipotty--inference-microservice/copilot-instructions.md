## inference-microservice

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FastAPI microservice for comparing base model (pythia-2.8b) and DPO fine-tuned model outputs. The service integrates RAG (Retrieval-Augmented Generation) using Pinecone for context retrieval.

## Common Commands

### Development
```bash
# Local setup
./setup.sh

# Run server locally
python3.10 main.py
# OR
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### Docker Operations
```bash
# Build image
docker build -t dpo-inference:latest .

# Build for specific platform (e.g., for deployment)
docker build --platform linux/amd64 -t dpo-inference:latest .

# Run container with GPU support
docker run --gpus all \
  -p 8000:8000 \
  -v $(pwd)/serviceAccountKey.json:/app/serviceAccountKey.json \
  -v $(pwd)/model_cache:/app/model_cache \
  -e FIREBASE_STORAGE_BUCKET=dpo-frontend.firebasestorage.app \
  -e PINECONE_API_KEY=your-key-here \
  dpo-inference:latest

# Push to Docker Hub
docker tag dpo-inference:latest username/dpo-inference:latest
docker push username/dpo-inference:latest

# Clean up Docker resources
docker system prune -af --volumes
```

### RAG Data Management
```bash
# Load Q&A data into Pinecone
export PINECONE_API_KEY='your-api-key'
python load_rag_data.py
```

### Testing API
```bash
# Health check
curl http://localhost:8000/health

# Run inference
curl -X POST http://localhost:8000/inference/run \
  -H "Content-Type: application/json" \
  -d '{
    "modelId": "policy_model.pt",
    "prompt": "What is machine learning?",
    "baseModel": "pythia_2_8b"
  }'
```

## Architecture

### Core Components

**main.py** - FastAPI application with startup/shutdown lifecycle:
- Initializes Firebase client, base model manager, fine-tuned model manager, and RAG retriever at startup
- Manages two endpoints: `/health` and `/inference/run`
- RAG is optional: service degrades gracefully if `PINECONE_API_KEY` is missing

**Model Management Strategy**:
- **Base Model** (`BaseModelManager`): Loaded once at startup, kept in VRAM throughout service lifetime
- **Fine-tuned Models** (`FineTunedModelManager`): Loaded on-demand per request, then immediately unloaded to conserve VRAM
- Both managers share the same tokenizer from the base model

**firebase_client.py** - Downloads fine-tuned model checkpoints from Firebase Storage:
- Implements local caching in `/app/model_cache` to avoid re-downloading
- Models are stored at path `policies/{modelId}` in Firebase bucket

**rag_retriever.py** - Pinecone integration for context retrieval:
- Uses `sentence-transformers/all-mpnet-base-v2` for embeddings
- Extracts knowledge base name from `modelId` (e.g., `policy_crypto.pt` → namespace `crypto`)
- Returns top-k Q&A pairs as context, formatted into prompt template

**load_rag_data.py** - Standalone script to populate Pinecone:
- Reads Q&A data from `sample_data.json`
- Groups items by knowledge base (dataset field)
- Generates embeddings and uploads to namespaced Pinecone index `dpo-qa-index`

### Request Flow

1. Request arrives at `/inference/run` with `modelId`, `prompt`, `baseModel`
2. Extract KB name from `modelId` and query Pinecone for relevant Q&A context
3. Format prompt with retrieved context into Q&A template
4. Generate text using base model (already loaded)
5. Download fine-tuned model from Firebase (if not cached)
6. Load fine-tuned checkpoint into `FineTunedModelManager`
   - Handles state-wrapped checkpoints: extracts from `checkpoint["state"]` if present
   - Handles vocab size mismatches: resizes model embeddings if checkpoint vocab differs
7. Generate text using fine-tuned model
8. Unload fine-tuned model immediately
9. Return both outputs with execution time

### Important Implementation Details

**Checkpoint Loading** (model_manager.py:115-154):
- Fine-tuned checkpoints may be wrapped in `{"state": {...}}` - extract if present
- Checkpoint vocab size may differ from base model - resize embeddings to match before loading state_dict
- Stop token `<|stop|>` is encoded and used as `eos_token_id` during generation

**Prompt Formatting** (main.py:184-212):
- RAG context is formatted as: `[Source N] (Community votes: N/A)\nQ: ...\nA: ...\n\n`
- Final prompt template: `Based on this knowledge:\n\n{context}Question: {prompt}\nAnswer:`
- If RAG fails or returns no results, falls back to: `Question: {prompt}\nAnswer:`

**Generation Parameters**:
- Configured via environment variables (see .env.example)
- Default: temperature=0.1, top_p=0.85, repetition_penalty=1.5, no_repeat_ngram_size=3
- Output is truncated at last complete sentence if it doesn't end with punctuation

**VRAM Management**:
- Base model stays loaded to avoid reload overhead
- Fine-tuned models are loaded/unloaded per request
- Explicit `torch.cuda.empty_cache()` called after unloading

## Environment Configuration

Required files:
- `serviceAccountKey.json` - Firebase service account credentials (not in repo)
- `.env` - Copy from `.env.example` and configure (not in repo)

Key environment variables:
- `FIREBASE_STORAGE_BUCKET` - Firebase bucket name for model storage
- `PINECONE_API_KEY` - Required for RAG functionality (service runs without it)
- `BASE_MODEL_NAME` - HuggingFace model identifier (default: `lambdalabs/pythia-2.8b-deduped-synthetic-instruct`)
- `DEVICE` - Inference device (`cuda` or `cpu`)
- Generation parameters: `MAX_NEW_TOKENS`, `TEMPERATURE`, `TOP_P`, `REPETITION_PENALTY`, `NO_REPEAT_NGRAM_SIZE`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pepsipotty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
