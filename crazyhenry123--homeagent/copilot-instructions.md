## homeagent

> Family AI agent app — family members chat with Claude (via Amazon Bedrock) through a mobile app. Backend runs on ECS Fargate with Flask, data in DynamoDB, infra managed with AWS CDK. AgentCore provides runtime orchestration, persistent memory, tool gateway, and identity management.

# Family AI Agent (homeagent)

## Project Overview
Family AI agent app — family members chat with Claude (via Amazon Bedrock) through a mobile app. Backend runs on ECS Fargate with Flask, data in DynamoDB, infra managed with AWS CDK. AgentCore provides runtime orchestration, persistent memory, tool gateway, and identity management.

## Monorepo Structure
- `backend/` — Flask API (Python 3.12)
- `mobile/` — Expo React Native app (TypeScript, SDK 54)
- `webui/` — Debug web console (static HTML/CSS/JS, hosted on S3+CloudFront)
- `infra/` — AWS CDK stacks (Python)

## Conventions

### Python (backend + infra)
- Python 3.12, type hints on all function signatures
- Flask app factory pattern in `backend/app/__init__.py`
- Use `ulid` for user/conversation IDs, `uuid4` for device IDs
- DynamoDB access through helpers in `backend/app/models/dynamo.py`
- Tests with `pytest`; run: `cd backend && .venv/bin/python -m pytest tests/`
- Lint: `ruff check backend/`
- Format: `ruff format backend/`

### TypeScript (mobile)
- Strict TypeScript, no `any`
- Expo managed workflow (SDK 54)
- State management: React Context + useReducer
- API client in `mobile/src/services/api.ts`
- Secure token storage via `expo-secure-store`
- Test on device via Expo Go app (scan QR code)

### API
- All endpoints prefixed with `/api/`
- Auth via `Authorization: Bearer <device_token>` header
- SSE streaming for chat responses (`text/event-stream`)
- Pagination via cursor-based `?limit=N&cursor=X`

### Infrastructure
- CDK Python, one stack per concern (network, data, agentcore, security, service, webui, pipeline)
- CDK Pipelines self-mutating pattern for CI/CD
- DynamoDB on-demand billing, tables created by CDK (cloud) or auto-init (local)
- ECS Fargate with ALB, 300s idle timeout for SSE
- ECR repository `homeagent-backend` for Docker images
- S3 + CloudFront for debug web UI static hosting

## AgentCore Integration
The `/api/chat` endpoint routes through AgentCore Runtime when `AGENTCORE_RUNTIME_ARN` is set.
The routing logic in `_get_chat_stream()` (in `backend/app/routes/chat.py`) follows this fallback chain:
1. **AGENTCORE_RUNTIME_ARN set** → `_stream_via_agentcore()` → AgentCore Runtime orchestration
2. **USE_AGENT_ORCHESTRATOR=true** → local Strands Agent orchestrator
3. **Neither set** → direct Bedrock `converse_stream`

The backend integrates with Amazon Bedrock AgentCore for:
- **Runtime** — Agent orchestration via `agentcore_runtime.py` (invoke agents, manage sessions). When `AGENTCORE_RUNTIME_ARN` is configured, the `/api/chat` endpoint routes all chat through AgentCore Runtime using `_stream_via_agentcore()`, which creates sessions, resolves sub-agent tools, and streams responses in the same SSE event format.
- **Memory** — Persistent family and per-member memory via `agentcore_memory.py` and `family_memory.py`
- **Gateway** — MCP tool gateway for health and family tools via `agentcore_gateway.py`
- **Identity** — Cognito-based auth with AgentCore identity via `agentcore_security.py`
- **Performance** — Caching and optimization via `agentcore_performance.py`

### AgentCore Services
| Service | File | Purpose |
|---------|------|---------|
| Runtime | `agentcore_runtime.py` | Invoke orchestrator agent, manage sessions |
| Memory | `agentcore_memory.py` | Store/retrieve family and member memories |
| Family Memory | `family_memory.py` | Family-scoped shared memory operations |
| Gateway | `agentcore_gateway.py` | Route tool calls through MCP gateway |
| Security | `agentcore_security.py` | Cognito token validation, identity mapping |
| Performance | `agentcore_performance.py` | Response caching, connection pooling |
| Integration | `agentcore_integration.py` | Unified facade for all AgentCore services |

## CI/CD Pipeline (AWS CodePipeline)
```
GitHub (main) → Synth CDK → Run Tests → Deploy Infra → Docker Build+Push → Update ECS → Deploy Web UI
```
- **Source**: GitHub `Crazyhenry123/homeagent` repo via CodeStar Connection
- **Test**: CodeBuild runs pytest with DynamoDB Local in Docker
- **Deploy**: CDK Pipelines deploys Network, Data, AgentCore, Security, Service, WebUi stacks
- **Build**: CodeBuild builds Docker image, pushes to ECR, triggers ECS rolling deploy
- **WebUI**: Syncs `webui/` to S3 bucket and invalidates CloudFront cache

### Component Pipelines (independently deployable)
Each component has its own fast pipeline triggered by file path changes on `main`:
- `homeagent-backend-fast` — backend/ changes → Test → Docker Build → ECS Deploy (~5 min)
- `homeagent-webui-fast` — webui/ changes → S3 Sync → CloudFront Invalidation (~1 min)
- `homeagent-infra` — infra/ changes → CDK Synth → CDK Deploy (~5 min)
- `homeagent-mobile` — mobile/ changes → TypeCheck → Expo Publish → Test URL (~3 min)

### First-time setup
```bash
# 1. Create a CodeStar Connection to GitHub in AWS Console
#    (Developer Tools → Settings → Connections → Create connection)
# 2. Bootstrap CDK
cd infra
pip install -r requirements.txt
cdk bootstrap aws://ACCOUNT_ID/us-east-1

# 3. Deploy pipeline with connection ARN
cdk deploy HomeAgentPipeline \
  -c account=ACCOUNT_ID \
  -c region=us-east-1 \
  -c github_connection_arn=arn:aws:codeconnections:us-east-1:ACCOUNT_ID:connection/UUID

# 4. Push to GitHub to trigger the pipeline
git push origin main
```

## Local Development
```bash
# Backend
cp .env.example .env   # fill in values
docker-compose up       # Flask + DynamoDB Local + MinIO

# Mobile
cd mobile
npm install
npx expo start          # Scan QR code with Expo Go on your phone

# Web Debug Console
python -m http.server 8080 -d webui
# Then configure the API endpoint to http://localhost:5000
```

## Environment Variables
- `AWS_REGION` — AWS region (default: us-east-1)
- `DYNAMODB_ENDPOINT` — DynamoDB endpoint (local dev only: http://dynamodb-local:8000)
- `TABLE_PREFIX` — Optional prefix for DynamoDB table names
- `BEDROCK_MODEL_ID` — Claude model ID (default: us.anthropic.claude-opus-4-6-v1)
- `SYSTEM_PROMPT` — System prompt for Claude
- `ADMIN_INVITE_CODE` — Pre-seeded invite code for first admin
- `COGNITO_USER_POOL_ID` — Cognito User Pool ID (from AgentCore stack)
- `COGNITO_CLIENT_ID` — Cognito User Pool Client ID (from AgentCore stack)
- `AGENTCORE_RUNTIME_ARN` — AgentCore Runtime ARN; when set, `/api/chat` routes through AgentCore Runtime instead of local orchestrator/direct Bedrock
- `AGENTCORE_ORCHESTRATOR_AGENT_ID` — AgentCore Runtime orchestrator agent ID
- `AGENTCORE_RUNTIME_ENDPOINT` — AgentCore Runtime endpoint URL
- `AGENTCORE_FAMILY_MEMORY_ID` — AgentCore Memory store ID for family memories
- `AGENTCORE_MEMBER_MEMORY_ID` — AgentCore Memory store ID for member memories
- `AGENTCORE_GATEWAY_ID` — AgentCore Gateway ID
- `HEALTH_MCP_ENDPOINT` — MCP server endpoint for health tools
- `FAMILY_MCP_ENDPOINT` — MCP server endpoint for family tree tools

## Testing
- Backend tests: `cd backend && .venv/bin/python -m pytest tests/ -v`
- 267 tests across 8 AgentCore test files + existing test suite
- Property-based tests use Hypothesis with reduced `max_examples` for speed (~68s total)
- Tests use moto for AWS service mocking (no real AWS calls)

---
> Source: [Crazyhenry123/homeagent](https://github.com/Crazyhenry123/homeagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
