## agentic-authz

> This project demonstrates OpenFGA-powered authorization for AI agents. It shows fine-grained access control at team, project, and operation levels.

# Agentic AuthZ - Project Guidelines

## Overview

This project demonstrates OpenFGA-powered authorization for AI agents. It shows fine-grained access control at team, project, and operation levels.

## Quick Commands

```bash
make setup          # Start services and seed data
make demo           # Run demo scenarios
make dev-gateway    # Run gateway in mock mode
make dev-demo       # Run Demo UI
make dev-admin      # Run Admin UI
```

## Architecture

- **Gateway** (Go): Authorization proxy that checks OpenFGA before forwarding tool calls
- **Demo UI** (Next.js): Interactive interface for testing authorization scenarios
- **Admin UI** (Next.js): Permission management dashboard
- **OpenFGA**: Authorization engine with ReBAC model

## Key Files

- `gateway/internal/authz/client.go` - OpenFGA client with mock mode
- `gateway/internal/handler/handler.go` - HTTP handlers for tool calls
- `openfga/model.fga` - Authorization model in FGA DSL
- `openfga/tuples.json` - Seed data for demo

## Authorization Model

Three levels:
1. **Team Level**: Teams → Tool Categories (Engineering → code tools)
2. **Project Level**: Projects → Tools (auth-service → postgres)
3. **Operation Level**: Roles → Operations (leads → delete_repo)

## Testing

```bash
# Test a tool call
curl -X POST http://localhost:9000/v1/tools/call \
  -H "Content-Type: application/json" \
  -d '{"tool":"github","operation":"create_pr","params":{},"context":{"user":"alice","project":"auth-service","team":"engineering"}}'
```

## Development Notes

- Gateway runs in mock mode by default (`MOCK_MODE=true`)
- Set `MOCK_MODE=false` to use real OpenFGA checks
- Demo UI connects to gateway on port 9000
- Admin UI shows static permission matrix

---
> Source: [Siddhant-K-code/agentic-authz](https://github.com/Siddhant-K-code/agentic-authz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
