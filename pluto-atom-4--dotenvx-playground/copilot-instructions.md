## dotenvx-playground

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **dotenvx showcase project** using pnpm. The primary purpose is to demonstrate best practices for managing environment variables and secrets using dotenvx, while ensuring sensitive values are never exposed.

## Critical Security Guidelines

**NEVER commit or expose:**
- Actual API keys, credentials, or usernames
- Real secret values from `.env` or `.env.local` files
- Plaintext passwords or authentication tokens in code, comments, or outputs

**When working with environment variables:**
- Always use placeholder values in example files (e.g., `API_KEY=xxx`, `PASSWORD=changeme`)
- Never print or log environment variable values
- Use `.env.local` for local development secrets (must be in `.gitignore`)
- Demonstrate the concept without using real credentials
- If creating examples, clearly mark them as "EXAMPLE" or use obviously fake values

**When providing code or responses:**
- Do not expand or reveal actual secret values if they appear in code
- Do not suggest logging sensitive environment variables for debugging
- Always recommend using dotenvx's features to manage secrets securely

## Development Setup & Commands

### Prerequisites
- Node.js (LTS recommended)
- pnpm (latest version recommended)

### Common Commands

```bash
# Install dependencies
pnpm install

# Development server
pnpm dev

# Build project
pnpm build

# Run linting
pnpm lint

# Run tests
pnpm test

# Run a single test file
pnpm test -- <test-file>

# View dotenvx status
pnpm dotenvx status

# Decrypt .env files (for demonstration)
pnpm dotenvx decrypt
```

### Environment Setup

1. Copy `.env.example` to `.env.local` for local development
2. Fill in placeholder values (never use real credentials)
3. Use dotenvx to manage and encrypt sensitive values in production
4. `.env.local` should be in `.gitignore` and never committed

## Project Architecture

The project structure demonstrates:
- **Configuration management** using dotenvx
- **Local vs. production secrets** handling
- **pnpm workspaces** (if applicable)
- **Build and development workflows**

### Key Files & Directories

- `.env` - Default environment variables (safe to commit, no secrets)
- `.env.local` - Local overrides (gitignored, for development only)
- `.env.*.encrypted` - Encrypted environment files (production secrets, safe to commit)
- `pnpm-workspace.yaml` - Workspace configuration (if monorepo)
- `.gitignore` - Ensures `.env.local` is never committed

## dotenvx Integration

dotenvx is used to:
- Load environment variables from `.env` files at runtime
- Encrypt sensitive values for production use
- Manage different configurations per environment
- Prevent accidental exposure of secrets

When demonstrating or modifying:
- Show how dotenvx loads variables
- Use example/placeholder values
- Never hardcode real credentials
- Demonstrate encryption/decryption concepts without real secrets

## Testing

Write tests that validate:
- Environment variables are loaded correctly
- Configuration fallbacks work as expected
- Secrets are not leaked in logs or errors

Use mock values and fixtures for testing, never real credentials.

## Deployment Notes

- Encrypted `.env.*.encrypted` files can be safely committed
- Use dotenvx CLI to decrypt at runtime in production
- Document required environment variables in `.env.example`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pluto-atom-4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
