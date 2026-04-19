## aletheia-ai

> Aletheia project context, deployment, and migration guidelines


# Aletheia AI — Project Context

## Project
Enterprise Agent Trust Framework: QTSP-ready AI agent governance, EU AI Act compliance, post-quantum crypto (ML-DSA), delegation chains, multi-tenant RBAC.

## Deployment (Ansible)
- University VM: `cd deploy/ansible && ./deploy.sh` (inventory `inventory.yml`, group `university`; код с **GitHub** через `git` на сервере; перед деплоем нужен `git push`)
- AWS: **`./deploy/aws-eatf.sh`** из корня репо (= `deploy/ansible/deploy.sh -i aws_inventory.yml`). `group_vars/aws.yml`: **https://eatf.duckdns.org** (Nginx + DuckDNS + Let’s Encrypt), ngrok выключен; секреты в корневом `.env` (`DEPLOY_DUCKDNS_TOKEN`, `DEPLOY_CERTBOT_EMAIL`, OAuth/JWT — см. `.env.example`). Без прокси: `-e deploy_reverse_proxy=false`
- University / ngrok: `https://kaia-uncharacterized-unorbitally.ngrok-free.dev` (в `group_vars/university.yml` — `deploy_ngrok_enabled: true`)
- Pass `-e openai_api_key=... -e google_client_id=... -e google_client_secret=... -e nextauth_secret=... -e ai_aletheia_jwt_secret=...` (optional: `-e ai_aletheia_oauth_auto_provision=false -e ai_aletheia_invite_secret=...` for closed demo + invite signup)
- Next.js BFF → Spring JWT: set **`AI_ALETHEIA_BFF_MINT_KEY`** (same value backend + frontend) for mint-by-userId; Docker/Ansible template includes it. Details: `docs/developers/en/bff-api-jwt-mint.md`

## Default Admin
- **Organization:** `default` (or leave empty)
- **Email:** `admin@default.local`
- **Password:** `Admin123!`

## DB Migrations (PostgreSQL)
Production uses PostgreSQL. Migrations must be PostgreSQL-compatible:
- Use `DOUBLE PRECISION`, not `DOUBLE`
- No `ON UPDATE CURRENT_TIMESTAMP` (MySQL-only)
- Avoid reserved words as column names (`references` → `attestation_references`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sapsan14) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
