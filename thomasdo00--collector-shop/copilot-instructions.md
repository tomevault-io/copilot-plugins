## collector-shop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Collector.shop is a marketplace web platform for selling collectible items between individuals (C2C). The platform handles secure transactions with a 5% commission, real-time chat, personalized recommendations, and fraud detection.

**Target items**: Limited edition sneakers, signed posters, original figurines, vintage cassettes, and other collectibles (excluding luxury goods and classic antiques).

## Architecture

### Hexagonal Architecture with SOLID Principles

The codebase follows a modular monolith structure designed for eventual microservices extraction:

```
module/
├── domain/           # Pure business logic (no external dependencies)
│   ├── entities/     # Value Objects
│   ├── usecases/     # Business use cases (SRP)
│   └── ports/        # Interfaces (DIP)
├── infrastructure/   # External implementations (OCP, LSP)
└── adapters/         # REST API routes
```

### Business Modules (Bounded Contexts)

- **User**: Registration, authentication, profile management
- **Catalog**: Articles, categories, search
- **Payment**: Stripe integration, transactions
- **Chat**: WebSocket messaging with content filtering
- **Notification**: Email and in-app notifications
- **Recommendation**: Personalized article suggestions
- **Fraud**: Anomaly detection (price variations, suspicious sellers)

### Event-Driven Communication

Modules communicate via an internal event bus (V1), with Kafka planned for V2.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 18 + Vite + TypeScript + Redux Toolkit |
| Backend | Fastify 4 + TypeScript + Knex.js |
| Database | PostgreSQL 16 |
| Cache | Redis 7 |
| Auth | JWT (Keycloak planned for V2) |
| Payments | Stripe |
| Storage | S3 / MinIO |
| Tests | Vitest + Testcontainers |

## Project Structure (Monorepo)

```
collector.shop/
├── frontend/          # React 18 + Vite + TypeScript + Redux Toolkit
├── backend/           # Fastify 4 + TypeScript + Knex.js
├── docker-compose.yml # PostgreSQL + Redis + MinIO
└── .github/workflows/ # CI/CD GitHub Actions
```

## Build and Development Commands

```bash
# Install dependencies (from root)
npm install

# Start Docker services (PostgreSQL, Redis, MinIO)
npm run docker:up

# Run database migrations
npm run db:migrate

# Development (starts both frontend and backend)
npm run dev

# Or separately:
npm run dev:frontend   # http://localhost:5173
npm run dev:backend    # http://localhost:3000 (API docs: /docs)

# Build
npm run build

# Tests
npm test
npm run test:coverage -w frontend
npm run test:coverage -w backend

# Linting
npm run lint
```

## Quality Gates (SonarQube)

- Coverage: > 80%
- Duplications: < 3%
- Code Smells: 0 blockers, < 5 major
- Bugs: 0 critical
- Vulnerabilities: 0
- Technical Debt: < 5%
- Cognitive Complexity: < 15 per function

## Key Business Rules

- All payments must go through the platform (no direct exchanges)
- 5% commission on each transaction
- Articles require admin validation before publication (except trusted sellers: score > 4.5/5 with 20+ sales)
- Chat messages are filtered for personal contact information (email, phone)
- Price changes are tracked and trigger fraud detection notifications
- Sellers receive payment 7 days after delivery confirmation
- Maximum 100 active articles per seller
- Articles expire after 90 days without sale

## Security Requirements

- bcrypt for password hashing (cost 12)
- Rate limiting: 100 requests/min
- TLS 1.3 mandatory
- 3D Secure for payments
- RGPD compliant (data stored in France via Scaleway)
- OWASP Top 10 protection (SQL injection, XSS, CSRF)

## Accessibility

WCAG 2.1 Level AA compliance required:
- Keyboard navigation for all interactive elements
- Screen reader support with ARIA attributes
- Minimum 4.5:1 contrast ratio
- Responsive design up to 200% zoom

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThomasDo00) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
