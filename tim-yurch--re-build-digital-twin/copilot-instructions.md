## re-build-digital-twin

> A digital twin app deployed on AWS. The "twin" is an AI persona that responds as a specific person, using conversational memory to maintain context across sessions.

# Digital Twin App

## What This Is

A digital twin app deployed on AWS. The "twin" is an AI persona that responds as a specific person, using conversational memory to maintain context across sessions.

## Deployment Architecture

```
Frontend
  S3 (static site) <-> CloudFront distribution

Backend
  API Gateway <-> Lambda (business logic)
      |
      +----> Amazon Bedrock (LLM inference)
      +----> S3 (conversational memory / history)
```

## Tech Stack

- **Frontend**: React with Next.js App Router (static export to S3)
- **Backend**: AWS Lambda (Python), API Gateway (HTTP API)
- **AI**: Amazon Bedrock
- **Memory**: S3 (JSON conversational history per session)
- **Package manager**: uv (Python)

## Project Structure

```
re-build-digital-twin/
├── CLAUDE.md
├── PROGRESS.md     # Step-by-step build checklist (5 days)
├── backend/        # Lambda functions, business logic
└── frontend/       # Next.js App Router app
```

## Build Plan

See [PROGRESS.md](PROGRESS.md) for the full step-by-step checklist across all 5 days. Work through it sequentially, checking off steps as they are completed.

---
> Source: [tim-yurch/re-build-digital-twin](https://github.com/tim-yurch/re-build-digital-twin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
