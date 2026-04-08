---
name: auth0
description: Manage users, applications, and authentication via Auth0 Management API. Use when this capability is needed.
metadata:
  author: openclaw
---
# Auth0
Identity platform.
## Environment
```bash
export AUTH0_DOMAIN="your-tenant.auth0.com"
export AUTH0_MGMT_TOKEN="xxxxxxxxxx"
```
## List Users
```bash
curl "https://$AUTH0_DOMAIN/api/v2/users" -H "Authorization: Bearer $AUTH0_MGMT_TOKEN"
```
## Create User
```bash
curl -X POST "https://$AUTH0_DOMAIN/api/v2/users" \
  -H "Authorization: Bearer $AUTH0_MGMT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "Pass123!", "connection": "Username-Password-Authentication"}'
```
## Get User
```bash
curl "https://$AUTH0_DOMAIN/api/v2/users/{userId}" -H "Authorization: Bearer $AUTH0_MGMT_TOKEN"
```
## List Applications
```bash
curl "https://$AUTH0_DOMAIN/api/v2/clients" -H "Authorization: Bearer $AUTH0_MGMT_TOKEN"
```
## Links
- Dashboard: https://manage.auth0.com
- Docs: https://auth0.com/docs/api/management/v2

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/openclaw)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/openclaw)
<!-- tomevault:4.0:skill_md:2026-04-08 -->
