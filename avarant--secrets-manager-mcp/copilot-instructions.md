## secrets-manager-mcp

> AWS Secrets Manager MCP server. Python + FastMCP, hosted on AWS App Runner, Okta OAuth 2.1 PKCE.

# CLAUDE.md

## Project

AWS Secrets Manager MCP server. Python + FastMCP, hosted on AWS App Runner, Okta OAuth 2.1 PKCE.

## Key Files

- `server/main.py` — the entire server (MCP tools, OAuth endpoints, secret entry form)
- `infra/app_runner.tf` — App Runner service + IAM + auto-scaling config
- `infra/ecr.tf` — ECR repo + `null_resource` that builds and pushes the Docker image
- `infra/main.tf` — provider, auth locals (Okta vs Cognito switch)
- `infra/terraform.tfvars` — **gitignored** — contains Okta issuer, client ID, audience

## Deploy

```bash
cd infra
AWS_PROFILE=platforms-forge-prod terraform apply -auto-approve

# terraform apply builds and pushes the Docker image automatically (null_resource triggers on main.py / requirements.txt / Dockerfile md5)
# but App Runner won't pull it until you trigger a deployment:
aws apprunner start-deployment \
  --service-arn arn:aws:apprunner:us-east-2:080403790243:service/secrets-mcp/69b9ab15680a4bd79466c9cfad880e14 \
  --region us-east-2 \
  --profile platforms-forge-prod
```

## Important Constraints

**`max_size=1` on App Runner auto-scaling is not negotiable.** The secret entry token store (`_pending_ops`, `_completed`) is in-memory. Multiple instances would cause form POSTs to land on a different instance than the one that holds the token, returning "link expired" to the user. Do not increase `max_size` without replacing the in-memory store with something external (e.g. ElastiCache, DynamoDB TTL).

**`auto_deployments_enabled = false`.** Terraform pushing a new image to ECR does not automatically redeploy App Runner. Always run `aws apprunner start-deployment` after apply.

## Secret Entry Flow

`create_secret` / `update_secret` do not collect the secret value themselves. They return a one-time URL. The user opens it in their browser, submits the form, and App Runner stores the value in `_completed[token]`. `finalize_secret(token)` then reads it and calls Secrets Manager.

The MCP spec defines `ctx.elicit_url()` for this pattern (value stays out of the MCP protocol), but as of April 2026 neither Claude Code nor Cursor implement client-side URL mode elicitation. See `docs/implementation-notes.md`.

## OAuth

The server acts as an OAuth authorization server proxy — it exposes `/.well-known/oauth-authorization-server` pointing at Okta's endpoints, plus a static `/register` DCR endpoint that returns the pre-configured Okta client ID. The actual token issuance and JWKS verification all go through Okta.

Each MCP client uses a different OAuth redirect URI. All must be registered in your Okta app:

| Client | Redirect URI |
|---|---|
| Claude Code | `http://localhost:3000/callback` |
| Cursor native HTTP | `cursor://anysphere.cursor-mcp/oauth/callback` |
| VS Code Copilot | `http://127.0.0.1:33418/` |
| Replit | `https://replit.com/connectors/oauth/callback` |

## AWS Profile

Use `platforms-forge-prod` for all AWS and Terraform commands.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avarant) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
