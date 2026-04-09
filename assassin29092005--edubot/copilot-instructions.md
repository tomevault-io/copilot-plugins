## edubot

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EduBot is an AWS serverless chatbot that helps students query academic information (syllabi, schedules, professor info, assignment deadlines). The request flow is: React frontend → Amazon Lex v2 (NLP) → AWS Lambda (Python 3.11) → DynamoDB.

## Architecture

Seven AWS services, all in **us-east-1**:
- **S3** hosts the React static site
- **Cognito Identity Pool** (unauthenticated) gives the browser credentials to call Lex
- **Lex v2** detects intents and extracts the `CourseId` slot, then invokes Lambda
- **Lambda** (`EduBotHandler`) handles 5 intents by querying DynamoDB and returning formatted responses
- **DynamoDB** table `EduBot-Courses` stores course data keyed by `course_id` (string)
- **CloudWatch** + **X-Ray** for monitoring and tracing

The Lex bot is created manually (not in CloudFormation) because Lex v2 CFN support is incomplete. Bot ID and Alias ID are passed to the frontend via environment variables.

Infrastructure-as-code is in `template.yaml` (AWS SAM). Stack name: `edubot-stack`.

## Commands

```bash
# Backend
sam build                                    # Build SAM template
sam deploy --guided                          # First-time deploy (interactive)
sam deploy                                   # Subsequent deploys
python backend/lambda/seed_data.py           # Seed DynamoDB with 8 courses

# Frontend
cd frontend && npm install                   # Install dependencies
cd frontend && npm start                     # Dev server (needs .env)
cd frontend && npm run build                 # Production build
aws s3 sync frontend/build/ s3://edubot-frontend-<ACCOUNT_ID>/ --delete

# Tests
python -m pytest tests/ -v                   # Run all 12 unit tests
python -m pytest tests/test_handler.py::TestGetCourseSyllabus -v  # Single test class

# Monitoring
aws cloudwatch put-dashboard --dashboard-name EduBot-Dashboard --dashboard-body file://monitoring/dashboard.json
```

Test dependencies: `pip install boto3 moto pytest`

## Key Design Decisions

- **Lambda reads `TABLE_NAME` from env var** (set by SAM via `!Ref CoursesTable`), never hardcoded
- **X-Ray tracing** uses try/except import so Lambda works even without `aws_xray_sdk` installed
- **Frontend env vars** use `REACT_APP_` prefix (Create React App convention); real values in `frontend/.env` (gitignored), template in `frontend/.env.example`
- **Lex response format**: Lambda returns `Close` (fulfilled) or `ElicitSlot` dialog actions; the response shape is dictated by Lex v2's Lambda fulfillment contract
- **Course IDs** are uppercased before DynamoDB lookup (`course_id.upper()`)

## Data Flow for Intent Handling

1. Lex matches user text to one of 4 intents (+ FallbackIntent) and extracts `CourseId` slot
2. Lambda `handler.lambda_handler` routes by `intent_name` to a `handle_*` function
3. If `CourseId` slot is missing, returns `ElicitSlot` to prompt the user
4. Otherwise queries DynamoDB, formats the response, returns `Close` with the message

## Testing

Tests use `unittest` with `unittest.mock.patch("handler.table")` to mock the DynamoDB table resource. The test file adds `backend/lambda/` to `sys.path` before importing. Tests cover: successful queries, missing slots (ElicitSlot), course not found, empty assignments, and fallback.

## CI/CD

`.github/workflows/deploy.yml` runs on push to `main`:
1. **deploy-backend**: pytest → sam build → sam deploy
2. **deploy-frontend** (depends on backend): npm ci → npm run build → s3 sync

Required GitHub secrets: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_ACCOUNT_ID`, `IDENTITY_POOL_ID`, `LEX_BOT_ID`, `LEX_BOT_ALIAS_ID`

## Naming Conventions

- SAM stack: `edubot-stack`
- Resource names: `EduBot-` prefix (PascalCase for CFN resources)
- S3 bucket: `edubot-frontend-<account-id>` (kebab-case)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Assassin29092005)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Assassin29092005)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
