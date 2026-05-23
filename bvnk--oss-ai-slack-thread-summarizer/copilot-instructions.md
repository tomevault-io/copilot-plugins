## oss-ai-slack-thread-summarizer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Build
```bash
# Build with SAM (recommended - handles Gradle build automatically)
sam build

# Alternative using Make
make build

# Direct Gradle build (for local development)
./gradlew clean build
```

### Deploy
```bash
# Deploy with SAM (guided)
make deploy

# Deploy without confirmation
make deploy-no-confirm

# Direct SAM deployment
sam deploy --guided
```

### Testing
```bash
# Run tests
./gradlew test
# or
make test
```

### Monitoring
```bash
# Tail Lambda logs
make logs
# or
sam logs -n SlackEventFunction --stack-name slack-ai-assistant --tail
```

### Clean
```bash
make clean
```

## Architecture Overview

This is a serverless Slack bot that uses AWS Lambda to process Slack events and respond with AI-generated content from Amazon Bedrock.

### Request Flow
1. **Slack Event Reception**: API Gateway receives POST requests at `/slack/events`
2. **Event Validation**: `SlackEventHandler` verifies Slack signatures and handles URL verification
3. **Deduplication**: Events are checked against DynamoDB to prevent duplicate processing
4. **Async Processing**: Valid events trigger async Lambda invocation via `LambdaInvokeService`
5. **AI Processing**: `AsyncProcessorHandler` retrieves thread context, sends to Bedrock, and posts response
6. **Response**: Bot posts AI-generated response back to Slack thread

### Key Components

- **SlackEventHandler**: Main entry point, handles both API Gateway requests and async invocations
- **AsyncProcessorHandler**: Processes mentions asynchronously, orchestrates AI response generation
- **BedrockService**: Interfaces with Amazon Bedrock for AI responses (Claude Sonnet model)
- **SlackService**: Handles all Slack API interactions (messages, reactions, thread retrieval)
- **DynamoService**: Manages event deduplication using DynamoDB with TTL
- **LambdaInvokeService**: Handles async Lambda invocations

### Infrastructure (SAM)
- Lambda function with Java 21 runtime on ARM64
- DynamoDB table for event deduplication with TTL
- API Gateway for webhook endpoint
- IAM policies for Bedrock, DynamoDB, and Lambda invocations

### Configuration
Environment variables are managed through SAM template:
- `SLACK_SIGNING_SECRET`: For request verification
- `SLACK_BOT_TOKEN`: For Slack API calls
- `DYNAMO_TABLE`: Deduplication table name
- `BEDROCK_MODEL_ID`: AI model identifier

---
> Source: [bvnk/oss-ai-slack-thread-summarizer](https://github.com/bvnk/oss-ai-slack-thread-summarizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
