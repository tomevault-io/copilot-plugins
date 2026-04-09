## kana

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## What Kana Is

Kana is a **spec-driven platform engineering studio** that lets enterprises visually design, generate, and operate cloud-native architectures (primarily Kubernetes workloads).

Kana Studio is NOT a diagramming tool. Every action on the canvas produces an executable, versionable **Kana spec** — the single source of truth for the architecture.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Frontend | React + TypeScript |
| Graph rendering | React Flow |
| Backend | Kotlin (preferred) or Node.js |
| Local execution | Docker / docker-compose |
| Messaging broker | Kafka (Redpanda acceptable for local) |
| Spec format | Kana Spec YAML |
| Storage (MVP) | Local filesystem |
| Storage (Phase 2+) | Cloud DB |

---

## Project Phases

### Phase 1 — MVP (current focus)
Goal: Developer drags Spring Boot services onto a canvas, connects them via API or Kafka, generates an Kana spec, and runs the architecture locally via Docker.

**MVP component types:**
- Spring Boot Service
- Kafka Cluster (logical)
- External Service (HTTP endpoint)

**MVP interaction types:**
- API (REST over HTTPS/mTLS/OAuth)
- Messaging (Service → Kafka topic)

**MVP deliverables:**
- Visual canvas with drag & drop
- Side panel configuration
- Kana spec generation
- `docker-compose.yml` generation from spec
- One-click local run (stubs + mock OAuth + TLS)
- Git export of spec

### Phase 2 — Platform Orchestration
Environment promotion (Dev → Test → Prod), live Kubernetes cluster visualization, replay engine, logical domain events abstracted from Kafka topics.

### Phase 3 — Enterprise Control Plane
Policy engine, multi-tenant SaaS, RBAC/SSO, CLI + VS Code extension, component marketplace.

---

## Core Architectural Decisions

1. **Kana spec (YAML) is the source of truth.** The UI is a write-head into the spec. Never store state only in the UI layer.
2. **Declarative UI.** Every UI action maps deterministically to a spec mutation. No hidden magic.
3. **Secure-by-default (non-negotiable).** All API connections default to HTTPS + mTLS + OAuth2 client credentials. All messaging defaults to SASL/TLS + schema validation. Security can only be disabled in an explicit advanced mode.
4. **Spec must be environment-aware.** The same spec drives local Docker, test, and production environments via environment overlays.
5. **Local and cloud share the same spec engine.** No divergence between local execution and cloud SaaS behaviour.
6. **Spec must carry replay metadata.** Even in MVP, design the schema to support future replay/simulation without breaking changes.

---

## Kana Spec Schema Shape

The spec is persisted as YAML (`kana-spec.yaml`) and is the single source of truth.

```yaml
# kana-spec.yaml (top-level structure)
kanaSpec: "1.0"
info:
  name: string
  version: string
  environment: dev | test | prod

services:
  - id: string
    type: spring-boot | kafka | external
    runtime: java21          # default for spring-boot
    ports: []
    env: {}
    resources: {}            # abstracted limits

interactions:
  - id: string
    type: api | messaging
    source: serviceId
    target: serviceId

    # For type: api
    api:
      method: GET | POST | PUT | DELETE | PATCH
      path: string
      auth:
        type: mTLS+OAuth2    # default
        oauth: client_credentials
      timeout: duration
      retry:
        enabled: true        # default
        policy: {}

    # For type: messaging
    messaging:
      broker: kafkaClusterId
      topic: string          # auto-generated default; physical in MVP
      format: avro | json-schema
      delivery: at-least-once  # default
      deadLetterTopic: string  # auto-generated

security:
  mtls:
    enabled: true
    certStrategy: auto       # auto-generate certs
  oauth:
    type: client_credentials

deployment:
  target: local | kubernetes
  localExecution:
    compose: docker-compose.yml  # generated path
```

---

## Backend Architecture (Spec Engine)

The backend is responsible for:

1. **Spec Engine** — accepting incremental graph mutations from the UI and persisting to `kana-spec.yaml`. Must be deterministic (same inputs → same YAML).
2. **Docker Composer** — translating Kana spec → `docker-compose.yml`, including:
   - Service containers
   - Redpanda for Kafka
   - Mock OAuth2 server
   - Auto-generated TLS certificates
3. **Git Export** — committing/pushing the spec to a configured Git remote.
4. **Validation Layer** — enforcing security defaults and policy rules before spec is accepted.

Expose these as REST endpoints consumed by the React frontend.

---

## Frontend Architecture (Canvas)

Key UI areas:

- **Canvas** — React Flow infinite canvas; components are React Flow nodes, connections are edges with protocol badges.
- **Component Palette** — sidebar listing draggable component types.
- **Side Panel** — context-sensitive config panel that appears when a node or edge is selected; writes back to the spec engine on save.
- **Toolbar** — Generate Spec, Run Locally, Export to Git actions.

State management: keep the **canonical state in the Kana spec** (fetched from backend). Local UI state (drag position, panel open/closed) is ephemeral.

---

## Security Defaults — Enforcement Rules

- Never generate an API interaction without `auth.type: mTLS+OAuth2` unless user has explicitly toggled advanced mode.
- Never generate a Kafka interaction without SASL/TLS and schema validation.
- Always auto-generate a dead letter topic for every consumer.
- `docker-compose.yml` generation must include a mock OAuth2 server and cert generation step.

---

## Key Design Questions (Resolved for MVP)

| Question | Resolution |
|---|---|
| Physical vs logical topic names | Physical in MVP; logical in Phase 2 |
| Where is canonical state stored? | Backend / `kana-spec.yaml` on disk |
| Can security be disabled? | Only via explicit advanced mode toggle |
| Local broker | Redpanda (Kafka-compatible, lighter) |
| Auth for local run | Auto-generated mock OAuth2 server |
| TLS for local run | Auto-generated self-signed certs |

---

## MVP Success Criteria

The MVP is complete when a developer can:

1. Drag `OrderService` (Spring Boot) and `PaymentService` (Spring Boot) onto the canvas.
2. Connect them with an API interaction (POST, mTLS+OAuth2).
3. Connect `PaymentService` to a Kafka cluster with a `payment.processed` topic (Avro).
4. Click **Generate** → valid `kana-spec.yaml` produced.
5. Click **Run Locally** → docker-compose spins up both services, Redpanda, mock OAuth2 server, with TLS.
6. Services communicate securely.
7. Generated YAML is viewable and exportable.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aarunbala)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aarunbala)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
