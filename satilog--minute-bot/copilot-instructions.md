## minute-bot

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

Minute Bot is a meeting memory system that processes audio into speaker-attributed transcripts and knowledge graphs. Two audio ingest paths:

1. **File upload** — audio file POSTed to `/meetings/upload`, processed offline. Works without Redis.
2. **Live streaming** — microphone audio streamed to `/meetings/stream` via Redis pub/sub pipelines.

After transcription and diarization, three KG extraction engines can run in parallel:
- **Triplet** — hybrid spaCy NER + LLM gap-fill + constrained triplet extraction
- **MinuteBot** — single-pass entity+relationship extraction via Claude tool_use
- **KG-Gen** — third-party kg-gen library

A query endpoint (`POST /meetings/<id>/query`) provides hybrid vector search + graph traversal with LLM-generated structured responses.

## Build & Run

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
python -m spacy download en_core_web_sm
minute-bot
```

API at `http://localhost:5000`. Frontend (separate repo: `minute-bot-web`) at `http://localhost:3000`.

## Project Structure

```
src/minute_bot/
├── api/                        # Flask blueprints
│   ├── meetings.py             # Upload, start/stop, transcripts, speakers
│   ├── graph.py                # KG extraction triggers, graph retrieval
│   ├── query.py                # Hybrid search query endpoint
│   ├── profiles.py             # Speaker enrollment (record + file upload)
│   ├── events.py               # SSE stream
│   └── health.py
├── core/
│   ├── transcriber.py          # faster-whisper
│   ├── diarizer.py             # Pyannote diarization + voice embeddings
│   ├── speaker_attribution.py  # Post-meeting speaker matching
│   ├── file_processing.py      # Offline upload pipeline
│   ├── kg_extraction.py        # Engine dispatcher (triplet/minutebot/kggen)
│   └── kg_engine/
│       ├── models.py           # Pydantic models, entity/event/relationship taxonomy
│       ├── prompts.py          # LLM prompt templates (extraction, cleanup, disambiguation)
│       ├── triplet_extractor.py    # Hybrid NER + constrained triplet extraction
│       ├── triplet_pipeline.py     # Triplet engine end-to-end pipeline
│       ├── extractor.py        # MinuteBot single-pass extraction
│       ├── pipeline.py         # MinuteBot engine pipeline
│       ├── embedder.py         # OpenAI text-embedding-3-small (1536-dim)
│       ├── disambiguator.py    # Cross-meeting entity disambiguation
│       ├── merger.py           # Subgraph dedup and merge
│       └── query.py            # Vector search + BFS traversal + LLM response
├── audio/                      # Encoding, processing, file I/O
├── db/                         # Database layer (one class per table)
│   ├── kg.py                   # KgNodes, KgEdges, KgGraphs, KgProvenance, KgEdgeMeta, KgExcerpts
│   ├── speaker_profiles.py     # Multi-sample embeddings, majority-vote matching
│   └── ...
├── models/                     # Pydantic data models
├── pubsub/                     # Redis pub/sub (live streaming only)
├── services.py                 # ML model init (Whisper, Pyannote)
└── config.py                   # pydantic-settings configuration

supabase/migrations/            # 16 SQL files (apply in order, 001-016)
data/meetings/                  # 12 sample meeting scripts (not committed)
data/speakers/                  # Speaker enrollment scripts (not committed)
```

## Database Tables

| Table | Purpose |
|-------|---------|
| meetings | Meeting metadata + processing status |
| transcripts | Whisper segments with speaker FK |
| speakers | Per-meeting speakers with 512-dim voice embeddings |
| speaker_profiles | Global enrolled profiles |
| speaker_profile_embeddings | Multi-sample voice embeddings per profile |
| audio_files | Recording metadata + Supabase Storage paths |
| kg_graphs | One row per meeting per engine (status tracking) |
| kg_nodes | Entities/events with 1536-dim embeddings |
| kg_edges | Typed relationships (58 relationship types) |
| kg_edge_meta | Key-value metadata on edges |
| kg_excerpts | Transcript evidence for nodes/edges |
| kg_provenance | Links nodes/edges back to source meetings |

## Key Configuration

Environment variables in `.env`:

| Variable | Required | Purpose |
|----------|----------|---------|
| SUPABASE_URL, SUPABASE_KEY | Yes | Database + storage |
| ANTHROPIC_API_KEY | Yes | KG extraction, transcript cleanup, query responses |
| OPENAI_API_KEY | Yes | Embeddings (text-embedding-3-small, 1536-dim) |
| HF_TOKEN | Yes | Pyannote model access |
| WHISPER_MODEL | No | Default: "base" |
| PORT | No | Default: 5000 |

## API Endpoints

### Core

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/meetings/upload` | POST | Upload audio file |
| `/meetings/<id>/transcripts` | GET | Speaker-attributed transcripts |
| `/meetings/<id>/extract-graph` | POST | Trigger KG extraction |
| `/meetings/<id>/graph?engine=` | GET | Get graph nodes + edges |
| `/meetings/<id>/query` | POST | Hybrid search query |
| `/profiles/enroll` | POST | Enroll speaker (live recording) |
| `/profiles/enroll-file` | POST | Enroll speaker (file upload) |
| `/health` | GET | Server health |

---
> Source: [satilog/minute-bot](https://github.com/satilog/minute-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
