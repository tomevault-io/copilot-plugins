## khyzr-agents

> This repo contains AI agents built with the **AWS Strands SDK** deployed on **Amazon Bedrock AgentCore Runtime**.

# Khyzr Agents — Cursor Rules

## Project Overview
This repo contains AI agents built with the **AWS Strands SDK** deployed on **Amazon Bedrock AgentCore Runtime**.
Only 3 agents are currently active and deployed. Focus exclusively on these unless told otherwise.

---

## Active Agents (do not modify others)

### Agent 01 — Market Intelligence
- **Path:** `agents/01-market-intelligence-agent/`
- **Purpose:** Competitive intelligence reports — news search (GDELT), SEC EDGAR filings, market analysis
- **Runtime:** `khyzr_market_intelligence_demo-IXK91q23u1`
- **S3 reports bucket:** `khyzr-market-intelligence-demo-110276528370`
- **S3 code bucket:** `khyzr-market-intelligence-demo-code-110276528370`
- **IAM role:** `khyzr-market-intelligence-demo-agentcore-role`

### Agent 36 — AP Automation
- **Path:** `agents/36-ap-automation-agent/`
- **Purpose:** Accounts payable automation — invoice extraction, 3-way PO matching, auto-approval, ledger updates
- **Runtime:** `khyzr_ap_automation_demo-yXLiHZ39Ob`
- **S3 invoices bucket:** `khyzr-ap-automation-demo-invoices-110276528370`
- **DynamoDB table:** `khyzr-ap-automation-demo`
- **IAM role:** `khyzr-ap-automation-demo-agentcore-role`

### Agent 40 — AR Collections
- **Path:** `agents/40-ar-collections-agent/`
- **Purpose:** Accounts receivable collections — risk tiering, aging analysis, collection actions
- **Runtime:** `khyzr_ar_collections_demo-HZchkDGBs5`
- **S3 reports bucket:** `khyzr-ar-collections-demo-reports-110276528370`
- **DynamoDB table:** `khyzr-ar-collections-demo`
- **IAM role:** `khyzr-ar-collections-demo-agentcore-role`

---

## Tech Stack

### Runtime
- **Language:** Python 3.12
- **Agent framework:** `strands-agents` SDK
- **Deployment:** Amazon Bedrock AgentCore Runtime (Code Zip — no Docker/ECR)
- **Model:** `us.anthropic.claude-3-5-sonnet-20241022-v2:0` (cross-region inference profile)
- **Architecture:** ARM64 (aarch64) — deps must be built for this platform

### Infrastructure
- **IaC:** Terraform (HCL)
- **Provider:** `hashicorp/aws ~> 5.0`
- **AgentCore runtime:** deployed via `null_resource` + AWS CLI `aws bedrock-agentcore` commands (not native TF resource)
- **State:** local (no remote backend)

### Key AWS Services
- Bedrock AgentCore Runtime
- S3 (code zips + data storage)
- DynamoDB (agent state)
- IAM (roles + policies)

---

## Critical Rules

### IAM Policy — BedrockInvokeAll
The Bedrock invoke policy **must** include all 3 resource ARN patterns or the agent will get 403 errors:
```json
{
  "Sid": "BedrockInvokeAll",
  "Effect": "Allow",
  "Action": ["bedrock:InvokeModel", "bedrock:InvokeModelWithResponseStream", "bedrock:Converse", "bedrock:ConverseStream"],
  "Resource": [
    "arn:aws:bedrock:*::foundation-model/*",
    "arn:aws:bedrock:*:ACCOUNT_ID:inference-profile/*",
    "arn:aws:bedrock:*:*:inference-profile/*"
  ]
}
```

### Code Zip Bundling
- All Python dependencies **must be pre-installed** into the zip — AgentCore does NOT run `pip install` at runtime
- Build command:
```bash
pip install --target {build_dir} \
  --platform manylinux2014_aarch64 \
  --python-version 3.12 \
  --only-binary=:all: \
  strands-agents bedrock-agentcore boto3 openpyxl
```
- Then copy `agent.py` into the build dir and zip the whole thing

### IAM Role Propagation
- After creating a new IAM role, wait **60–90 seconds** before first invocation or you'll get 500 errors
- Pattern: create role → sleep 90s → create/recreate runtime

### boto3 Invocation Pattern
```python
resp = client.invoke_agent_runtime(
    agentRuntimeArn=runtime_arn,
    payload=json.dumps({"prompt": user_input})
)
body = resp['response'].read()  # NOT resp['body']
```

### Model ID
Always use: `us.anthropic.claude-3-5-sonnet-20241022-v2:0`
Never use: `anthropic.claude-sonnet-4-5` or any non-prefixed model ID

---

## Deployment
```bash
# Deploy all 3 active agents
./deploy.sh all

# Deploy individual agent
./deploy.sh agent01
./deploy.sh agent36
./deploy.sh agent40

# Requires env vars:
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...   # if using temp creds
```

---

## Demo UI
- **Path:** `demo-ui/server.py`
- Pure Python stdlib HTTP server + single-page app
- Run: `python3 server.py` from `demo-ui/` dir → opens on port 8080
- Lists all 3 deployed agents with per-agent chat history

---

## What NOT to Touch
- Agents 02–35, 37–39, 41–47 — scaffolding only, not deployed
- `.terraform/` dirs, `terraform.tfstate*`, `*.zip` — excluded from git, do not commit
- The `seo-posts/` and `skills/` dirs — separate concern, not agent infra

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yakatak5) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
