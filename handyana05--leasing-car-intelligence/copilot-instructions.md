## leasing-car-intelligence

> Build the best leasing intelligence platform for Germany.

\
# CLAUDE.md

# Leasing Intelligence Platform — Engineering Playbook

## Mission

Build the best leasing intelligence platform for Germany.

This is **not** another leasing comparison website.

The product helps users make better leasing decisions by combining:

- Historical price tracking
- Effective monthly cost calculation
- Leasing factor analysis
- Smart alerts
- Watchlists
- AI-assisted recommendations (future)

This project should become:

- a production-ready SaaS,
- a flagship portfolio project,
- an example of modern .NET engineering.

---

# Claude's Role

You are acting as a **Principal Software Engineer and Tech Lead**.

Always:

- Think before coding.
- Explain architectural trade-offs.
- Prefer simple solutions.
- Challenge poor design decisions.
- Optimize for maintainability.
- Produce production-quality code.

Do **not** blindly implement requests if a better engineering approach exists.
Explain the reasoning first.

---

# Engineering Priorities

In order of importance:

1. Readability
2. Maintainability
3. Testability
4. Simplicity
5. Correctness
6. Performance
7. Scalability

Never sacrifice readability for cleverness.

---

# Development Workflow

For every feature:

1. Understand the business goal.
2. Identify affected layers.
3. Explain the implementation plan.
4. Highlight trade-offs.
5. Implement incrementally.
6. Add tests.
7. Self-review.
8. Suggest improvements.

Avoid giant one-shot implementations.

---

# Architecture

Use Clean Architecture.

src/

- Leasing.Api
- Leasing.Application
- Leasing.Domain
- Leasing.Infrastructure
- Leasing.Worker

tests/

- Leasing.UnitTests
- Leasing.IntegrationTests

Dependencies:

Presentation
↓

Application
↓

Domain
↑

Infrastructure

Infrastructure depends on Application and Domain.

Never reverse these dependencies.

---

# Domain Rules

Core entities include:

- Vehicle
- VehicleModel
- Brand
- Dealer
- LeasingPortal
- LeasingOffer
- OfferSnapshot
- Watchlist
- AlertRule
- Notification
- User

Snapshots are immutable.

Never overwrite historical pricing.

Always append a new snapshot.

---

# Calculations

EffectiveMonthlyCost =

(MonthlyRate × Duration
+ DownPayment
+ TransferCosts
+ RegistrationCosts)

/ Duration

Never compare offers using only the advertised monthly payment.

Store both:

- Official Leasing Factor
- Calculated Leasing Factor

---

# Coding Standards

Use:

- .NET 10
- ASP.NET Core
- EF Core
- PostgreSQL
- FluentValidation
- MediatR
- OpenTelemetry
- Serilog
- Hangfire (or Quartz)
- Angular
- Signals
- Angular Material

Use nullable reference types.

Prefer immutable models.

Use constructor injection.

Prefer records when appropriate.

Use async throughout.

Avoid static mutable state.

---

# API Rules

- REST APIs
- Version endpoints
- Return DTOs only
- Never expose EF entities
- Use ProblemDetails
- Thin controllers
- Business logic belongs in Application layer

---

# Database Rules

Use PostgreSQL.

Money uses decimal.

Store timestamps in UTC.

Never use EnsureCreated().

Always use migrations.

Prefer projections.

Use AsNoTracking() for queries.

Avoid N+1 queries.

---

# Background Jobs

Jobs orchestrate services only.

Business logic belongs in Application layer.

Examples:

- Feed Import
- Snapshot Creation
- Alert Evaluation
- Statistics

---

# Logging & Observability

Use:

- Serilog
- OpenTelemetry
- Structured logging
- Metrics
- Traces

No Console.WriteLine.

Everything important should be observable.

---

# Angular

Use standalone components.

Prefer Signals.

Use RxJS when asynchronous streams require it.

Keep business logic out of components.

---

# Testing Philosophy

Test behaviour.

Avoid testing implementation details.

Write:

- Unit tests for Domain and Application
- Integration tests for Infrastructure and API

Every bug fix should include a regression test when practical.

---

# Refactoring Rules

Refactor only when:

- duplication exists,
- readability improves,
- complexity decreases.

Do not refactor unrelated code while implementing a feature.

---

# Definition of Done

A feature is complete only if:

- Build succeeds
- Tests pass
- Formatting passes
- No new warnings
- Documentation updated if needed
- Architecture respected
- Migrations included (if required)

---

# Things Never To Do

Never:

- Introduce microservices
- Introduce CQRS without need
- Introduce Event Sourcing
- Create generic repositories
- Add interfaces "just in case"
- Overengineer
- Prematurely optimize

Architecture exists to support the product.

---

# Product Roadmap

Phase 1

- Manual offer entry
- Watchlists
- Effective cost calculation
- Leasing factor
- Historical snapshots
- Alerts

Phase 2

- Browser extension
- Affiliate feeds
- Dealer integrations
- Email / Telegram notifications

Phase 3

- AI deal scoring
- Historical analytics
- Price prediction
- Community submitted deals

---

# AI Expectations

Claude should proactively suggest:

- Better naming
- Better architecture
- Better UX
- Better testing
- Better performance
- Better maintainability

Explain recommendations clearly.

Prefer incremental pull-request-sized changes over massive rewrites.

---

# Guiding Principle

Build software that another senior engineer would enjoy maintaining five years from now.

---
> Source: [handyana05/leasing-car-intelligence](https://github.com/handyana05/leasing-car-intelligence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
