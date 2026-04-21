## convex

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a customer support ticket analysis and response generation system that uses RAG (Retrieval Augmented Generation) to provide contextual support responses. The system combines:

- **Supabase backend** for data storage and serverless functions
- **Python FastAPI** for machine learning inference and ticket processing
- **Vector search** using FAISS for semantic ticket matching
- **LLM integration** for generating support responses

## Architecture

### Core Components

1. **Supabase Backend** (`supabase/`)
   - PostgreSQL database with custom extensions
   - Edge functions for serverless execution
   - Authentication and API management
   - Database migrations handle ticket storage and processing functions

2. **Python ML Service** 
   - `main.py`: FastAPI server with RAG-based ticket response generation
   - `rag_chatbot.py`: Gradio interface for testing the chatbot
   - Vector embeddings using SentenceTransformer models
   - FAISS index for semantic search across historical tickets

3. **Data Processing**
   - Large datasets: `bq-results-*.jsonl`, `tickets.csv`, `quoted.csv`
   - SQL scripts for data analysis and ticket processing
   - Python utilities for data transformation

### Key Database Functions (from migrations)

- `format_ticket_for_embedding()`: Prepares ticket data for vector search
- `format_ticket_for_prompt()`: Formats tickets for LLM prompts  
- `build_llm_prompt()`: Constructs prompts with relevant context
- `call_together_chat()`: Integrates with Together AI for LLM calls
- `fetch_front_ticket()`: Retrieves tickets from Front API

## Common Development Commands

### Supabase Local Development

```bash
# Start local Supabase stack
supabase start

# Stop local stack
supabase stop

# Reset database with fresh migrations
supabase db reset

# Generate new migration
supabase migration new <migration_name>

# Push schema changes
supabase db push
```

### Python Services

```bash
# Run the FastAPI server
python main.py
# Serves on http://0.0.0.0:8000

# Run the Gradio chatbot interface  
python rag_chatbot.py

# Run with uvicorn directly
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

## Environment Configuration

The project uses environment variables managed through:
- `.env` file for local development (contains API keys for HuggingFace, Together AI, Front API)
- `supabase/config.toml` for Supabase-specific configuration
- Database vault for secure credential storage

**Important**: The `.env` file contains sensitive API keys. Never commit changes to this file.

## Data Flow

1. **Ticket Ingestion**: Historical support tickets are processed and embedded
2. **Vector Search**: New tickets are matched against historical ones using semantic similarity
3. **Context Retrieval**: Most relevant historical tickets provide context for response generation
4. **LLM Generation**: Language models generate appropriate support responses using retrieved context
5. **Category Boosting**: Tickets from the same category get preference in similarity matching

## Development Notes

- The system processes customer support tickets from various categories: Authentication, Data Sync, Stability, Export/Import, Billing
- Vector embeddings use the 'all-MiniLM-L6-v2' model for efficient similarity search
- Database functions are security definer functions requiring proper permissions
- The project includes both batch processing scripts and real-time API endpoints
- Large datasets suggest this system handles significant volumes of support tickets

## Testing the System

Use the Gradio interface (`rag_chatbot.py`) to test ticket response generation locally. The system requires:
- Proper environment variables for API access
- Local Supabase instance running
- Historical ticket data loaded for effective similarity matching

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dventimisupabase) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
