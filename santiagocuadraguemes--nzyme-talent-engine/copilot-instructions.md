## nzyme-talent-engine

> Serverless recruitment automation on AWS Lambda (Python 3.11, Docker). Processes CVs with AI (OpenAI GPT-5-mini + Pydantic structured output), manages candidates across Notion (UI/CRM) and Supabase (PostgreSQL + JSONB + Storage). Uses pdfplumber for PDF extraction, optional Exa.ai for LinkedIn enrichment, and Logfire for observability.

# CLAUDE.md - Nzyme Talent Engine

## Project Overview

Serverless recruitment automation on AWS Lambda (Python 3.11, Docker). Processes CVs with AI (OpenAI GPT-5-mini + Pydantic structured output), manages candidates across Notion (UI/CRM) and Supabase (PostgreSQL + JSONB + Storage). Uses pdfplumber for PDF extraction, optional Exa.ai for LinkedIn enrichment, and Logfire for observability.

Three workers — **Factory** (process setup), **Harvester** (CV ingestion), **Observer** (change monitoring) — each with dual triggers: scheduled (EventBridge) and event-driven (Notion workspace webhooks).

## Architecture References

Detailed architecture docs are in `.claude/rules/`:
- @.claude/rules/architecture.md — workers, data flow, identity resolution, AI-pending reprocessing, database schemas
- @.claude/rules/webhooks.md — webhook routing, feature flags, adding new handlers
- @.claude/rules/notion-schema.md — path-scoped rules for Notion property changes
- @.claude/rules/testing.md — local testing commands and simulation

---

## AWS Operations

The AWS CLI is authenticated globally (IAM user `nzyme-santiago-IAM`, account `416418941636`). All commands below work from this project without additional setup.

### Resource Map

| Resource | Name / Identifier | Region |
|----------|-------------------|--------|
| Lambda function | `nzyme-talent-management` (Python 3.11, 512 MB, 300 s, handler `main_lambda.lambda_handler`) | `eu-west-1` |
| Lambda Function URL (webhooks) | `https://vi6n7zvmytou7djtx7ixmobc4e0ittqz.lambda-url.eu-west-1.on.aws/` | `eu-west-1` |
| S3 deploy bucket | `nzyme-talent-engine-deploy` | `eu-west-1` |
| CloudWatch log group | `/aws/lambda/nzyme-talent-management` | `eu-west-1` |
| EventBridge: Factory | `nzyme-factory-schedule` — `cron(6/10 * * * ? *)` | `eu-west-1` |
| EventBridge: Harvester | `nzyme-harvester-schedule` — `cron(0/10 * * * ? *)` | `eu-west-1` |
| EventBridge: Observer | `nzyme-observer-schedule` — `cron(3/10 * * * ? *)` | `eu-west-1` |

All three schedules fire every 10 minutes, offset by 3 minutes so workers don't stampede.

### Deploy

```powershell
powershell -ExecutionPolicy Bypass -File scripts/deploy.ps1
```

The script caches pip dependencies (reinstalls only when `requirements.txt` SHA changes), builds `lambda.zip`, uploads to `s3://nzyme-talent-engine-deploy/lambda.zip`, then runs `aws lambda update-function-code`. Direct upload is avoided because the zip is ~46 MB (hits Lambda's direct-upload timeout).

**Verify a deploy actually changed code** — `update-function-code` prints `LastModified`, but the authoritative check is `CodeSha256`:
```bash
aws lambda get-function-configuration --function-name nzyme-talent-management \
  --region eu-west-1 --query CodeSha256 --output text
```
Capture before and after; if they match, the deploy didn't replace the code.

### Tail Logs

```bash
# Last 5 minutes, live follow (Ctrl+C to stop)
aws logs tail /aws/lambda/nzyme-talent-management --region eu-west-1 --since 5m --follow

# Filter for a specific worker run
aws logs tail /aws/lambda/nzyme-talent-management --region eu-west-1 --since 30m \
  --filter-pattern '"[Harvester]"'

# Errors only
aws logs tail /aws/lambda/nzyme-talent-management --region eu-west-1 --since 1h \
  --filter-pattern 'ERROR'
```

### Manually Invoke the Lambda

```bash
# Force a harvester run via EventBridge-style payload
aws lambda invoke --function-name nzyme-talent-management --region eu-west-1 \
  --payload '{"task":"harvester"}' --cli-binary-format raw-in-base64-out /tmp/out.json
cat /tmp/out.json

# Swap "harvester" for "observer" or "factory"
```

For webhook-shaped invocations, prefer hitting the Function URL with `curl` so the routing code runs end-to-end.

### Inspect Config / Env Vars

```bash
# Full config
aws lambda get-function-configuration --function-name nzyme-talent-management --region eu-west-1

# Just env vars (redacted secrets)
aws lambda get-function-configuration --function-name nzyme-talent-management \
  --region eu-west-1 --query 'Environment.Variables'
```

To flip a feature flag (e.g. enable a webhook handler) without redeploying, use `update-function-configuration` with `--environment`. **Note:** this replaces the entire env var map, so always GET first, modify, then PUT the full set back.

### Pause / Resume Scheduled Workers

```bash
aws events disable-rule --name nzyme-harvester-schedule --region eu-west-1
aws events enable-rule  --name nzyme-harvester-schedule --region eu-west-1
```

Useful when debugging locally to prevent EventBridge from racing with your manual runs.

### Rollback

Lambda keeps previous versions. To roll back:
```bash
aws lambda list-versions-by-function --function-name nzyme-talent-management --region eu-west-1
# Find the prior $LATEST's Version number, then:
aws lambda update-alias --function-name nzyme-talent-management \
  --name prod --function-version <N> --region eu-west-1
```
(Only applies if an alias is in use; otherwise re-deploy the previous zip from git.)

---

## Coding Guidelines

### Language Convention
- **Code/variables**: English
- **Logs/comments**: English, keep logs minimal
- **Notion properties**: English (defined in constants.py)

### Architecture Patterns

- **Dependency Injection** — Workers receive initialized clients via constructor
- **Lazy Initialization** — Lambda only instantiates clients needed for the current task
- **run_once() Pattern** — Workers execute a single pass, then exit (EventBridge)
- **Webhook Entry Points** — Workers also have single-page handlers (`run_from_webhook`, `handle_webhook_event`, `process_single_from_webhook`)
- **Feature Flag Pattern** — Webhook handlers gated by `WEBHOOK_*_ENABLED` env vars (default `false`)
- **Static + Dynamic Registry** — `WebhookRouter` resolves DB IDs via env vars first, then Supabase lookup

### Code Style

- **Early returns** on errors — don't nest, exit early
- **Numbered steps** in complex methods (`# 1. Download CV`, `# 2. Parse with AI`, etc.)
- **Constants for property names** — always import from `core/constants.py`, never hardcode strings
- **Handler name constants** — use `HANDLER_*` from `core/constants.py` for webhook routing
- **Logger per module** — `self.logger = get_logger("ModuleName")`

### Error Handling

- Log errors with context but don't crash the batch
- Mark items as processed even on partial failure (prevents infinite loops)
- Use try/except around external API calls (Notion, Supabase, OpenAI, Exa)

### Keeping Docs in Sync

After making code changes, update the relevant `.claude/rules/*.md` file if the change affects documented behavior. Specifically:

- **New/changed worker logic, data flow, DB tables, or identity resolution** → update `architecture.md`
- **New/changed webhook handlers, feature flags, or routing** → update `webhooks.md`
- **New/changed Notion properties** → follow the checklist in `notion-schema.md`
- **New/changed local testing commands or env vars** → update `testing.md`
- **New/changed AWS resources (Lambda config, EventBridge schedules, S3 buckets, function URL)** → update the `AWS Operations` section in this file (`CLAUDE.md`)
- **New architectural patterns, code style rules, or conventions** → update this file (`CLAUDE.md`)

If unsure whether a change warrants a docs update, ask: *"Would a future session make a mistake without knowing this?"* If yes, update the relevant file.

---
> Source: [santiagocuadraguemes/nzyme-talent-engine](https://github.com/santiagocuadraguemes/nzyme-talent-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
