## playeros-sdk

> Operate a PlayerOS casino marketing platform from an agent — list/segment players, send compliant email & SMS campaigns, run journeys, and handle TCPA/CCPA/GDPR — via the REST API, the @playeros/sdk, or the MCP server.


# PlayerOS

PlayerOS is a casino marketing platform for gaming operators. Use it to manage a player
database (gaming-native fields: Player ID, tier, ADT, coin-in), send multi-channel
campaigns (email, SMS, AI voice, web push, Player Inbox), run automation journeys, and
enforce TCPA/CCPA/GDPR compliance.

## Authenticate

Create a property-scoped API key (`pk_…`) at https://playeros.ai/dashboard/mcp. Send it as
`X-Api-Key: pk_…` (or `Authorization: Bearer pk_…`). Keys carry `read`/`write` scopes.

## Three ways to call it

1. **MCP server** (best for agents): `https://api.playeros.ai/api/mcp` — Streamable HTTP,
   25 tools, discoverable resources. Manifest: https://playeros.ai/.well-known/mcp.json
2. **SDK**: `npm i @playeros/sdk` →
   ```ts
   import { PlayerOS } from "@playeros/sdk";
   const playeros = new PlayerOS({ apiKey: process.env.PLAYEROS_API_KEY! });
   await playeros.players.list({ limit: 50 });
   ```
3. **CLI**: `npx @playeros/sdk players list --limit 25`

## Key operations

- Players: list, search, get, create, update, delete, batch-validate
- Campaigns: send email / SMS (pass an `Idempotency-Key`; every send runs a pre-send
  compliance check)
- Compliance: pre-send check, record consent
- Privacy: export / delete player data (CCPA / GDPR)
- Webhooks: list, create

## References

OpenAPI https://playeros.ai/openapi.json · Docs https://playeros.ai/api-docs.md · Auth
https://playeros.ai/auth.md

## Safety

Sends reach real players and some operations delete data. Treat campaign sends,
`players delete`, and data deletion as destructive; never bypass the compliance check.

---
> Source: [movaMedia-Inc/playeros-sdk](https://github.com/movaMedia-Inc/playeros-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
