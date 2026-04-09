## hybrid-decentralized-aml-framework-integrating-ml-authority-with

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Production-grade **Blockchain-Based KYC/AML Fraud Detection System** (thesis project). Combines Hyperledger Fabric for immutable audit trails with ensemble ML fraud detection (XGBoost, LightGBM, Random Forest, GNN, Autoencoder) trained on the Elliptic Bitcoin dataset.

## Common Commands

All developer workflows go through `make`. Run `make help` for the full list.

### Bootstrap
```bash
bash scripts/setup.sh   # One-command idempotent full local setup
cp .env.example .env    # Configure environment variables
```

### Infrastructure
```bash
make infra-up           # Start Docker stack (postgres, mongo, redis, kafka, vault, etc.)
make infra-down         # Stop containers (keep volumes)
make infra-clean        # Destroy containers + volumes (destructive)
make infra-status       # Show container status
```

### Build
```bash
make build                        # Build all 9 Go services
make build-svc SVC=iam-service    # Build a single Go service
make build-ml                     # Install Python ML dependencies (Poetry)
make proto                        # Regenerate gRPC stubs (Go + Python) from /proto/*.proto
```

### Run
```bash
make run                          # Start all services via docker-compose
make run-svc SVC=iam-service      # Run a single service locally
make fabric-up                    # Start Hyperledger Fabric network separately
```

### Testing
```bash
make test                         # All unit tests
make test-unit-go                 # Go unit tests with coverage
make test-unit-python             # Python unit tests
make test-unit-chaincode          # Chaincode tests
make test-integration             # Integration tests (requires Docker)
make test-e2e                     # E2E tests (Postman collections)
make test-perf                    # Locust performance tests
make test-security                # gosec + bandit security scan
make test-coverage                # Open coverage reports
```

### Lint & Format
```bash
make lint                         # All linters (golangci-lint + flake8)
make lint-go                      # Go only (.golangci.yml config)
make lint-python                  # flake8 + black + mypy
make fmt                          # Auto-format (gofmt + black + isort)
```

### Database
```bash
make migrate                      # Run all migrations
make migrate-down                 # Rollback last migration
make seed                         # Seed development data
make seed-ml                      # Download Elliptic Bitcoin dataset
```

### ML
```bash
make ml-train                             # Train all models
make ml-train-model MODEL=xgboost         # Train a specific model
make ml-evaluate                          # Generate model comparison table
make ml-serve                             # Run ML service locally (FastAPI + gRPC)
```

### Blockchain
```bash
make fabric-up                    # Start Hyperledger Fabric (3-org network)
make chaincode-deploy             # Deploy kyc-contract, alert-contract, audit-contract
```

## Architecture

### Services (`/services/`)
10 microservices communicating over **gRPC** (proto definitions in `/proto/`):

| Service | Language | Responsibility |
|---|---|---|
| `api-gateway` | Go | Traefik + JWT auth, rate limiting, CORS |
| `iam-service` | Go | Authentication, RBAC, JWT, TOTP MFA |
| `kyc-service` | Go | Customer onboarding, OCR, face verification |
| `transaction-service` | Go | Kafka consumer, feature extraction |
| `ml-service` | Python | FastAPI + gRPC, ensemble fraud models, SHAP/LIME |
| `blockchain-service` | Go | Hyperledger Fabric SDK wrapper |
| `alert-service` | Go | Alert generation, WebSocket notifications |
| `case-service` | Go | Case management, SAR generation |
| `analytics-service` | Go | Reports, metrics aggregation |
| `encryption-service` | Go | HashiCorp Vault Transit wrapper (AES-256-GCM) |

### Request Flow
1. **API Gateway** (Traefik) → JWT auth + rate limit → injects `RequestMetadata` (trace ID, user ID, role) into gRPC headers
2. **Transaction Service** consumes Kafka events → extracts features → calls **ML Service** via gRPC
3. **ML Service** returns fraud probability + SHAP explanations
4. If fraud: **Alert Service** publishes to Kafka → WebSocket notifications
5. **Blockchain Service** anchors KYC records and alerts on Hyperledger Fabric
6. **Case Service** generates SARs; all actions logged to Kafka audit topic

### Blockchain (`/blockchain/`)
- **Hyperledger Fabric 2.x**, 3 organizations (Primary Bank / Regulator / Partner)
- 3 channels: `kyc-channel`, `alert-channel`, `audit-channel`
- 3 Go chaincodes: `kyc-contract`, `alert-contract`, `audit-contract`

### ML Pipeline (`/ml/`)
- **Dataset**: Elliptic Bitcoin Transactions (203K txs, 166 features, temporal graph)
- **Models**: Random Forest, XGBoost, LightGBM, GraphSAGE GNN, Autoencoder, Ensemble (weighted voting)
- **Targets**: >94% precision, >90% recall, >0.98 AUC-ROC
- **Explainability**: SHAP (top-5 features) + LIME per prediction
- **Registry**: MLflow at `http://localhost:5000`

### Event Bus (Kafka Topics)
`kyc.events` · `transactions.raw` · `alerts.created` · `audit.events` · `blockchain.events` · `notifications.outbound`

### Data Stores
- **PostgreSQL 15** — KYC, IAM, alerts, cases (relational)
- **MongoDB 6** — transaction time-series
- **Redis 7** — sessions, caching, rate limiting, bloom filters

### Security Model
- All PII encrypted via **Encryption Service → Vault Transit** before DB storage
- JWT: 15min access tokens + 7-day refresh tokens stored in Redis
- Rate limit: 100 req/min public; 5 failed logins → 15min lockout
- All DB queries parameterized; TLS 1.3 enforced in production

### Shared Go Libraries (`/shared/go/`)
- `logger/` — structured logging (zerolog)
- `tracing/` — OpenTelemetry instrumentation
- `middleware/` — gRPC interceptors (auth, logging, tracing)

## Key Files
- `go.work` — Go workspace (10 modules, one per service + shared)
- `pyproject.toml` — root Python config; `/ml/pyproject.toml` — ML dependencies (Poetry)
- `.golangci.yml` — Go linter configuration
- `docker-compose.yml` — full local dev stack (13 containers)
- `docker-compose.test.yml` — isolated test environment
- `.env.example` — all required environment variables (copy to `.env`)

## Local Dev Endpoints (after `make run`)
| Service | URL |
|---|---|
| API Gateway | http://localhost:8080 |
| Grafana | http://localhost:3000 |
| Jaeger | http://localhost:16686 |
| Prometheus | http://localhost:9090 |
| MLflow | http://localhost:5000 |
| Vault UI | http://localhost:8200 |
| Kafka UI | http://localhost:8090 (dev-tools profile) |

## 15-Phase Roadmap Status
- **Phases 1–4** (Complete): Foundation, infrastructure, proto contracts, IAM, encryption
- **Phases 5–9** (In Progress): KYC, Hyperledger Fabric, ML service, transaction monitoring, alerts
- **Phases 10–15** (Pending): Case management, API Gateway, analytics, testing, Kubernetes, CI/CD

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mashu-wq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mashu-wq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
