## infra

> InfraGuard is a microservice-oriented backend optimized for platform engineering. It provides RESTful APIs to manage cloud infrastructure provisioning via Terraform and processes real-time server monitoring metrics.

# InfraGuard - Developer Guidelines for Jules AI

## Project Overview
InfraGuard is a microservice-oriented backend optimized for platform engineering. It provides RESTful APIs to manage cloud infrastructure provisioning via Terraform and processes real-time server monitoring metrics.

## Tech Stack
- **Framework**: Python 3.11+ / FastAPI
- **Relational DB**: PostgreSQL (via SQLAlchemy 2.0 / async) - used for relational entities (Servers, Alerts, Users).
- **NoSQL DB**: MongoDB (via Motor/Pymongo) - used for high-throughput time-series metrics.
- **Infrastructure**: Terraform, Docker, Docker Compose.
- **Testing**: pytest (with async support).

## Architectural & Coding Principles (CRITICAL)
1. **Clean Architecture & SOLID**: Strictly separate API routes, Business Logic (Services), and Data Access (Repositories). Routes must NEVER execute raw DB queries directly.
2. **Performance**: Write highly efficient async queries. Use proper indexing strategies for PostgreSQL and MongoDB.
3. **Clean Code**: Follow PEP8 strictly. Include type hints for all function arguments and return types.
4. **Testing**: All core services must be testable. Mock external calls (like Terraform subprocess execution).

---
> Source: [KarelNig/infra](https://github.com/KarelNig/infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
