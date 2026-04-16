## truthlayer

> - Production serverless REST API for AI hallucination detection

# TruthLayer — Cursor IDE Rules

## Project Identity
- Production serverless REST API for AI hallucination detection
- AWS Lambda + API Gateway + DynamoDB + Amazon Bedrock Titan Embeddings V2
- NOT frontend-heavy — core value is the verification engine
- AWS 10,000 AIdeas Competition — Top 1,000 Semi-Finalist

## Stack
- Python 3.9 (Lambda runtime) — all backend code
- Next.js 16 + TypeScript (dashboard only)
- AWS SAM (IaC) — template.yaml is the single source of truth
- pytest (87 unit tests, MockEmbeddingProvider, no AWS needed)

## Code Conventions — Python

### Imports in Lambda Handlers
Always include at the TOP before other imports:
```python
import sys
sys.path.insert(0, '/opt/python/python')
sys.path.insert(0, '/opt/python')
```

### API Key Auth
Every handler EXCEPT health must call:
```python
from src.utils.auth import validate_api_key
is_valid, error_response = validate_api_key(event)
if not is_valid:
    return error_response
```

### DynamoDB Responses
Always use `Decimal` for float values stored in DynamoDB:
```python
from decimal import Decimal
item["score"] = Decimal(str(float_value))
```

### DynamoDB Reserved Words
`content`, `name`, `status`, `value` are reserved words. Always alias with `ExpressionAttributeNames`:
```python
table.get_item(
    Key={"document_id": doc_id},
    ProjectionExpression="#c",
    ExpressionAttributeNames={"#c": "content"},
)
```

### Error Handling
```python
def build_response(status_code, body):
    return {
        "statusCode": status_code,
        "body": json.dumps(body, default=str),
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Headers": "Content-Type,X-Api-Key,Authorization",
        }
    }
```

## Code Conventions — TypeScript (Dashboard)

### API Calls
All API calls go through `dashboard/src/lib/api.ts`.
Never hardcode API_URL or API_KEY directly in components.
Always use `process.env.NEXT_PUBLIC_API_URL` and `process.env.NEXT_PUBLIC_API_KEY`.

### Component Structure
- Pages in `dashboard/src/app/`
- Shared API client in `dashboard/src/lib/api.ts`
- No separate components folder (all inline for simplicity)

## File Structure Rules

### NEVER commit:
- `dashboard/.env.local` (contains real API key)
- Real API keys anywhere (`tl_` prefixed with real token)
- `layer/` build artifacts (auto-generated)
- `.aws-sam/` directory

### ALWAYS ensure:
- `src/` is copied to `layer/python/src/` before `sam build`
- New Lambda handlers have `sys.path.insert` lines
- New endpoints (except `/health`) call `validate_api_key()`
- DynamoDB policies updated in `template.yaml` for new table access

## DynamoDB Tables
- `TruthLayerApiKeys` — key: `api_key_hash` (SHA-256 of raw key)
- `TruthLayerDocuments` — key: `document_id`
- `TruthLayerEmbeddings` — key: `document_id` + `chunk_index`
- `TruthLayerVerifications` — key: `verification_id`

## Classification Thresholds (from src/config.py)
- VERIFIED: cosine similarity >= 0.80
- UNCERTAIN: cosine similarity >= 0.55
- UNSUPPORTED: cosine similarity < 0.55

## Deploy Checklist
1. `python -c "import shutil; shutil.copytree('src', 'layer/python/src', dirs_exist_ok=True)"`
2. `sam build`
3. `sam deploy`
4. Test: `curl .../prod/health` → `{"status": "healthy"}`
5. Test auth: no key → 401, valid key → 200

## API Key Rules
- Format: `tl_{secrets.token_urlsafe(32)}` (46 chars total)
- Storage: SHA-256 hash stored in `TruthLayerApiKeys.api_key_hash`
- Raw key NEVER stored anywhere — only the hash
- Generate: `python scripts/generate_api_key.py "OwnerName"`

## Testing
- All tests use `MockEmbeddingProvider` — no AWS credentials needed
- Run: `pytest tests/ -v`
- 87 tests must pass before any deploy
- Test files in `tests/` directory

## What NOT to do
- Do not add `--use-container` to `sam build`
- Do not add `--guided` to `sam deploy`
- Do not use class name `TruthLayerClient` (correct name: `TruthLayer`)
- Do not call `verify()` without `source_documents` or `document_ids` — raises ValueError
- Do not commit anything in `layer/python/` — gitignored build artifact
- Do not skip copying `src/` to `layer/` before building
- Do not use `ProjectionExpression` with reserved words without `ExpressionAttributeNames`

## Key Files Added (Steps 1–4)
- `src/embeddings/cached_provider.py` — DynamoDB cache wrapper (SHA-256, 7-day TTL)
- `src/utils/auth.py` — now includes rate limit check + atomic usage_count increment
- `lambda/verify/handler.py` — now accepts `document_ids` alongside `source_documents`
- `sdk/python/truthlayer.py` — `verify(document_ids=...)`, `upload/delete_document()`
- `examples/` — 3 integration demos (customer support, document QA, legal analyzer)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Prakhar2025) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
