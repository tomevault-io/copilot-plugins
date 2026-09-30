## website-ai-chatbot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Monday" is an AI chatbot API for Ricky Martin's personal portfolio website. It runs as an AWS Lambda function (via Mangum/FastAPI), uses AWS Bedrock (Claude model) as the LLM, and maintains per-session conversation history in process memory.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally (requires AWS credentials and env vars set)
uvicorn src.main:app --host 0.0.0.0 --port 8000

# Run locally with Docker Compose (recommended)
docker compose up

# Rebuild and run
docker compose up --build

# Run all tests
pytest tests/

# Run a single test file
pytest tests/test_api.py -v

# Build Lambda Docker image
docker buildx build --platform linux/amd64 --tag chatbot:latest .
```

## Required Environment Variables

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION                  # default: us-east-1
API_KEY                     # required for /chat endpoint
ALLOWED_ORIGINS             # CORS whitelist, default: http://127.0.0.1:5500
BEDROCK_MODEL_ID            # default: anthropic.claude-3-haiku-20240307-v1:0
LANGSMITH_API_KEY           # optional, enables LangSmith tracing
LANGSMITH_TRACING           # optional, set to "true" to activate
LANGSMITH_ENDPOINT          # optional, LangSmith endpoint URL
LANGSMITH_PROJECT           # optional, LangSmith project name
S3_BUCKET_NAME              # optional, for logging
ECR_REPOSITORY              # CI/CD only — ECR repo URI used by deploy workflow
```

## Architecture

**Request flow:**
```
Client → FastAPI (main.py)
  → verify_api_key + rate limit (5 req/min) + CORS check
  → ChatAgent.chat() (agent.py, LanggGraph StateGraph)
    → MemoryStore.get(session_id) — fetches history from in-process dict
    → build prompt: SystemMessage + history + HumanMessage + "Assistant:"
    → bedrock.get_bedrock_response() — boto3 invoke_model, Anthropic format
    → MemoryStore.update() — appends AIMessage to history
  → {"message": response_text}
```

**Key files:**
- `src/main.py` — FastAPI app, `/chat` endpoint, Mangum Lambda handler
- `src/agent.py` — `ChatAgent` class wrapping a single-node LanggGraph StateGraph
- `src/bedrock.py` — raw boto3 call to Bedrock; returns response string
- `src/memory.py` — `MemoryStore`: `Dict[session_id → List[BaseMessage]]`
- `src/config.py` — all env var loading + the full Monday system prompt
- `src/logger.py` — `configure_logging()` + `get_logger()`; stdout always, rotates `chatbot.log` when running locally (skipped on Lambda)
- `src/utils.py` — `log_to_s3()` helper; no-ops unless `LOG_BUCKET` env var is set
- `src/chat.py` — `ChatSession` wrapper (defined but not wired into main API)

**Deployment:** GitHub Actions (`.github/workflows/deploy.yml`) triggers on push to `main`: runs pytest, builds a `linux/amd64` Docker image, pushes to ECR, and updates the Lambda function code.

## Key Design Notes

- **Session memory is ephemeral** — stored in process memory and lost on Lambda cold start or redeployment. The client is responsible for providing a consistent `session_id`; the API generates a UUID if none is given.
- **Two Dockerfiles** — `Dockerfile.local` runs uvicorn with hot reload for local development; `Dockerfile` builds the production Lambda image (`linux/amd64`).
- **Tests require real AWS credentials** — the live tests in `tests/` hit Bedrock directly. Mock-based variants are commented out in both test files.
- `ChatSession` (`src/chat.py`) is unused in the live API — `ChatAgent` in `agent.py` handles everything.
- `src/reporting/` is completely commented out (dead code).
- `python-jose` and `passlib` are in `requirements.txt` but not used.
- The system prompt (Monday's personality and Ricky's portfolio context) lives entirely in `src/config.py` — edit there to change chatbot behavior.

---
> Source: [RickyMartin-dev/Website_AI_Chatbot](https://github.com/RickyMartin-dev/Website_AI_Chatbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
