## fyp2025-ai

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

University final year project (FYP) monorepo containing three independent Python microservices, all deployed to AWS Lambda as Docker container images (ECR). Each service uses AWS Bedrock for LLM inference and lives in its own directory with its own dependencies.

## Services

### meal-ai
Raw AWS Lambda function (no framework) that analyses meal photos. Two variants exist:
- `analyse_meal_fuzzy.py` — matches extracted ingredients to a Postgres/RDS nutrition database using fuzzy text matching
- `analyse_meal_vector.py` — matches using Amazon Titan v2 vector embeddings (pgvector)
- `migrate_embeddings.py` — one-off script to backfill embeddings into the `nutrition_items` table

Uses Bedrock Converse API for image analysis. Connects to an RDS Postgres database for nutrition lookups. The Lambda handler entrypoint is `handler.lambda_handler` (the Dockerfile copies one of the variants as `analyse_meal.py`).

### health-advisor
Stateless AWS Lambda chatbot. Completely rewritten — no longer uses FastAPI/Mangum, DynamoDB, or S3. Current architecture:

- **`handler.py`** — Lambda entrypoint. Verifies JWT, parses body, fetches context, calls Bedrock, saves turn.
- **`auth.py`** — Verifies Supabase JWTs locally using PyJWT (no network call).
- **`db.py`** — Supabase client (service role). Reads `health_profiles` and `meals` tables; writes to `conversation_history`.
- **`agent.py`** — Builds system prompt from profile + recent meals, calls Amazon Nova Lite 2 (`global.amazon.nova-2-lite-v1:0`) via the Bedrock `converse()` API.
- **`test_cli.py`** — Local interactive terminal test. Loads `.env` or `.env.example`, connects directly to Supabase and Bedrock, bypasses JWT.

Uses Supabase for all persistence (health profiles, meal logs, conversation history). Conversation history is passed in the request body by the client (last 20 messages / 10 turns); the Lambda saves each new turn to `conversation_history` but does not read it back.

Deployed as a Docker container via `push_to_ecr.sh` to ECR `us-east-2`.

Required environment variables: `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`, `SUPABASE_JWT_SECRET`.

### restaurant-summary
FastAPI + Mangum service that takes scraped OpenRice restaurant data (reviews, images, metadata) and produces a health-scored summary.

- **`app.py`** — FastAPI routes: `GET /health`, `POST /summarise`, `POST /restaurant-ai` (alias).
- **`bedrock_client.py`** — Calls Amazon Nova Pro via the Bedrock `converse()` API with optional image payloads.
- **`image_fetcher.py`** — Downloads images from URLs; detects format from `Content-Type` response header (falls back to URL extension). Supports jpeg, png, gif, webp.
- **`config.py`** — Loads `BEDROCK_REGION` (default `us-east-1`), `BEDROCK_MODEL_ID` (default `us.amazon.nova-pro-v1:0`), `MAX_IMAGES`, `IMAGE_TIMEOUT_SECS`.
- **`prompts.py`** — System prompt constant for the restaurant analyst.

**Important:** The model ID `us.amazon.nova-pro-v1:0` is a cross-region inference profile. `BEDROCK_REGION` must be a US region. The Lambda IAM role needs `bedrock:InvokeModel` on:
- `arn:aws:bedrock:us-east-1:<account>:inference-profile/us.amazon.nova-pro-v1:0`
- `arn:aws:bedrock:us-east-1::foundation-model/amazon.nova-pro-v1:0`
- `arn:aws:bedrock:us-east-2::foundation-model/amazon.nova-pro-v1:0`
- `arn:aws:bedrock:us-west-2::foundation-model/amazon.nova-pro-v1:0`

## Common Commands

Each service is independent. Always `cd` into the service directory first.

```bash
# Install dependencies (each service has its own requirements.txt)
cd health-advisor && pip install -r requirements.txt
cd meal-ai && pip install -r requirements.txt
cd restaurant-summary && pip install -r requirements.txt

# Test health-advisor locally (reads .env or .env.example)
cd health-advisor
python test_cli.py <supabase-user-id>

# Run restaurant-summary locally
cd restaurant-summary
uvicorn app:app --reload --port 8081

# Build and push Docker image to ECR us-east-2 (requires AWS CLI + Docker)
cd <service-dir>
./push_to_ecr.sh
```

There are no test suites, linters, or formatting tools configured.

## Architecture Notes

- **Bedrock client pattern**: Each service has its own lazily-initialised boto3 singleton (`_client = None` / `_get_client()`). All services now use the `converse()` API.
- **Lambda packaging**: All three services use Docker containers pushed to ECR `us-east-2`. health-advisor switched from zip+deploy.sh to Docker+push_to_ecr.sh.
- **No shared code**: The three services share no modules.
- **Persistence**: health-advisor uses Supabase (Postgres) for all storage. meal-ai uses RDS Postgres for nutrition lookups. restaurant-summary is stateless.
- **Auth**: health-advisor verifies Supabase JWTs in-process (PyJWT, HS256, audience `authenticated`). The `sub` claim is the Supabase user UUID.
- **Cross-region inference profiles**: restaurant-summary uses `us.amazon.nova-pro-v1:0`. health-advisor uses `global.amazon.nova-2-lite-v1:0`. Direct model IDs (without the `us.`/`global.` prefix) will fail with `ValidationException` on on-demand throughput.
- **Image format detection**: restaurant-summary reads the `Content-Type` HTTP response header to determine image format before sending to Bedrock, avoiding MIME mismatch errors when URLs don't reflect the actual format (e.g. OpenRice serving WebP from `.jpg` URLs).

## Deployment

All three services: `push_to_ecr.sh` builds a `linux/amd64` Docker image and pushes to ECR in `us-east-2`. Update the Lambda function image in the AWS console or via CLI after pushing.

---
> Source: [KedahMY/FYP2025_AI](https://github.com/KedahMY/FYP2025_AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
