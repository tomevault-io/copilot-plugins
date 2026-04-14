## connections-helper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NYT Connections Helper - a web service that displays today's puzzle words with definitions.

- **CLI tool**: `get_connections.js` - local script for testing
- **AWS deployment**: Lambda + API Gateway + DynamoDB at `connections-helper.anystupididea.com`

## Commands

```bash
# Local CLI
npm start                    # Run local CLI script

# AWS Deployment
./deploy.sh                  # One-click deployment to AWS

# CDK Commands (from cdk/ folder)
cd cdk && npm run cdk synth  # Synthesize CloudFormation
cd cdk && npm run cdk diff   # Preview changes
cd cdk && npm run cdk deploy # Deploy stack

# Manual Lambda Invocation
aws lambda invoke --function-name connections-helper-fetch /dev/stdout
```

## Architecture

```
EventBridge (12:05 AM ET) → Fetch Lambda → DynamoDB
Route 53 → API Gateway → Serve Lambda → DynamoDB → HTML Response
S3 Bucket → CloudFront → Static Assets (favicon, og-image, etc.)
```

**Key Components:**
- `lambdas/fetch/` - Node.js Lambda, fetches puzzle via NYT API daily
- `lambdas/serve/` - Returns cached HTML from DynamoDB (references CloudFront for assets)
- `assets/icons/` - Brand assets deployed to S3 via CDK
- `cdk/` - TypeScript CDK infrastructure (us-east-1)

## Project Structure

```
connections_helper/
├── get_connections.js      # CLI script (local testing)
├── deploy.sh               # One-click AWS deployment
├── assets/
│   ├── logos/              # Source brand images
│   └── icons/              # Generated assets (deployed to S3)
│       ├── favicon.ico
│       ├── og-image.png    # Social sharing image (1200x630)
│       └── ...
├── cdk/                    # CDK infrastructure
│   ├── lib/connections-helper-stack.ts  # Main stack
│   └── bin/app.ts          # CDK app entry
└── lambdas/
    ├── fetch/              # NYT API fetch Lambda
    │   ├── index.js
    │   └── cultural-refs.js
    └── serve/              # HTML serving Lambda
        └── index.js
```

## Extending Cultural References

Add entries to `lambdas/fetch/cultural-refs.js` for words the dictionary API doesn't cover:

```javascript
module.exports = {
  'WORD': 'Cultural meaning; alternate usage',
};
```

## DNS

Uses existing Route 53 hosted zone `Z6ED78QGBFL6M` for `anystupididea.com`. DNS is automatically configured by CDK.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bobbu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Bobbu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
