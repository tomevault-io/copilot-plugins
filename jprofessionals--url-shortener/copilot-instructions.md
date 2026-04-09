## url-shortener

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A serverless URL shortener built in Rust using **Hexagonal Architecture** (Ports & Adapters). Designed for AWS Lambda/DynamoDB free tier, with portability to GCP, self-hosted, or local development.

## Build Commands

```bash
# Build default members (domain + api-server)
cargo build

# Build entire workspace
cargo build --workspace

# Build Lambda artifacts for AWS deployment
make build-lambdas

# Run tests
cargo test --workspace --all-features

# Check formatting
cargo fmt --all -- --check

# Run clippy
cargo clippy --workspace --all-targets -- -D warnings
```

## Running Locally

```bash
# Option A: No Google setup (fastest, uses X-Debug-User header for auth)
make run-api-local-none     # Backend on :3001
make run-admin-frontend     # Frontend on :8000

# Option B: With Google Sign-In
export ALLOWED_DOMAIN=example.com
export GOOGLE_OAUTH_CLIENT_ID=your-client-id.apps.googleusercontent.com
make run-api-local-google
make run-admin-frontend
```

## Deployment

**IMPORTANT:** Always use Makefile targets for deployment. Never run `sam deploy` or `wrangler` commands directly. If a deployment command is missing from the Makefile, add it there first.

### Available Deployment Targets

```bash
make deploy-dev        # Deploy backend to dev environment
make deploy-prod       # Deploy backend to prod (requires confirmation)
make deploy-frontend   # Deploy admin UI to Cloudflare Pages
make deploy-all-dev    # Deploy backend + frontend to dev
make deploy-all-prod   # Deploy backend + frontend to prod
```

### First-time Frontend Setup

```bash
cd admin-frontend
npm install          # Install wrangler (pinned in package.json)
npx wrangler login   # Authenticate with Cloudflare
```

### Environment Detection

The frontend auto-detects the environment from hostname:
- `dev-admin-sc.jpro.dev` → uses dev backend
- `admin-sc.jpro.dev` → uses prod backend

## Architecture

Three-tier workspace following hexagonal architecture:

```
domain/                 # Pure business logic - NO cloud SDKs, minimal deps
├── Types: Slug, ShortLink, NewLink, UserEmail
├── Traits (Ports): LinkRepository, SlugGenerator, Clock
└── Service: LinkService (orchestrates create/resolve)

adapters/               # Infrastructure implementations
├── aws-dynamo/        # DynamoDB adapter (production)
├── sqlite-adapter/    # SQLite adapter (local dev)
└── google-auth/       # OIDC token verification

apps/                   # Entry points
├── api-server/        # Axum HTTP server (local dev)
├── lambda-redirect/   # AWS Lambda: GET /{slug} → 308 redirect
└── lambda-admin/      # AWS Lambda: POST/GET /api/links (authed)

admin-frontend/         # Static Svelte SPA
```

**Key principle:** The `domain/` crate has zero cloud dependencies. All cloud-specific code lives in `adapters/`.

## Key Files

| Path | Purpose |
|------|---------|
| `domain/src/lib.rs` | Core types and traits |
| `domain/src/service.rs` | LinkService business logic |
| `domain/src/base62.rs` | Base62 slug encoding |
| `apps/api-server/src/main.rs` | Local dev server (Axum) |
| `apps/lambda-admin/src/main.rs` | Admin API Lambda handler |
| `adapters/aws-dynamo/src/lib.rs` | DynamoRepo implementation |
| `adapters/google-auth/src/lib.rs` | Google OIDC verification |
| `infra/sam/template.yaml` | AWS SAM/CloudFormation template |

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `STORAGE_PROVIDER` | `memory`, `sqlite`, or `aws` |
| `AUTH_PROVIDER` | `none` or `google` |
| `ALLOWED_DOMAIN` | Google Workspace domain for auth |
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth2 client ID |
| `DB_PATH` | SQLite file path (default: `./data/shortlinks.db`) |
| `RUST_LOG` | Log level filter (e.g., `info,api_server=debug`) |
| `LOG_FORMAT` | `pretty` or `json` |

## Testing a Single Crate

```bash
cargo test -p domain
cargo test -p api-server
cargo test -p google-auth
```

## Workspace Structure

Default members (`cargo build` without flags): `domain`, `api-server`

All members: `domain`, `api-server`, `lambda-redirect`, `lambda-admin`, `aws-dynamo`, `google-auth`, `sqlite-adapter`

## Code Style

- `unsafe_code = forbid` at workspace level
- Clippy warnings treated as errors in CI
- Feature flags for optional adapters: `sqlite`, `dynamo`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jprofessionals)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jprofessionals)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
