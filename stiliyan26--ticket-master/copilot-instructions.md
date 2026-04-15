## ticket-master

> Full project context, goals, and Trading212 benchmark for the Ticket Master learning project.


## Project Overview

**Ticket Master Lite** is a learning project designed to bridge the skill gap from "Mid-level Engineer" to "Distributed Systems Engineer" by **Jan 5th**. This is NOT a commercial product - it's a deliberate practice ground for high-frequency, high-concurrency backend patterns.

## The Trading212 Benchmark (The Goal)

The benchmark is a friend working on **High-Frequency Trading / Fintech systems** at Trading212 involving distributed locks and microservices. Ticketmaster mirrors Fintech challenges:

| Trading212 Concept | Ticketmaster Equivalent | Why We're Building It |
|--------------------|------------------------|----------------------|
| Liquidity / Balance | Ticket Inventory | Both are finite numbers. You cannot sell what you don't have. |
| Double Spending | Overselling | If 2 users click "Buy" at the same ms, a bad system sells 2 items. We must prevent this. |
| Order Execution | Booking Transaction | Money must move ONLY if the ticket is secured. This requires ACID properties. |
| Market Fairness | First-Come-First-Serve | We need Distributed Locks (Redis) to queue users fairly without killing the database. |

## Architecture: High-Frequency Booking Engine

**Focus**: Pure Backend API (no UI) using Nest.js and PostgreSQL.

### Tech Stack
- **Framework**: NestJS (TypeScript)
- **Database**: PostgreSQL with TypeORM
- **Cache/Locking**: Redis (Redlock pattern)
- **Queue**: RabbitMQ/BullMQ
- **Testing**: Jest (Unit + E2E)
- **Infrastructure**: Docker

## The Roadmap

### Phase 1: The Foundation (CURRENT)
**Goal**: Build a "Safe" Schema that rejects bad data.
- Strict DTO validation (no negative numbers)
- DB Constraints (`CHECK > 0`)
- Optimistic Locking (`@VersionColumn`)
- TypeORM Integration

### Phase 2: Distributed Locking
**Goal**: Move locking logic out of the Database and into Redis.
- Redlock pattern implementation
- Strategy pattern for lock providers
- High traffic handling without DB bottleneck

### Phase 3: Resilience
**Goal**: Handle network failures gracefully.
- Idempotency keys (prevent double-charging if internet cuts out)
- Event Queues (RabbitMQ)
- Circuit Breaker pattern
- Saga workflows for multi-step transactions

## Current Status

You are in **Phase 1**:
- ✅ DTOs created (`CreateBookingDto`) to validate inputs
- ✅ Entities created (`Event` and `Booking`) to enforce data integrity
- ✅ Database module with TypeORM configured
- ✅ Config validation with Zod
- 🔄 Next: Service Logic to handle actual transactions (taking lock + decrementing ticket count)

## Key Learning Objectives

1. **Concurrency Control** - Optimistic vs Pessimistic locking
2. **Distributed Systems** - Redis-based distributed locks
3. **Transaction Safety** - ACID compliance, preventing race conditions
4. **Resilience Patterns** - Circuit breakers, retries, idempotency
5. **NestJS Advanced Patterns** - Custom providers, dynamic modules, interceptors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Stiliyan26)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Stiliyan26)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
