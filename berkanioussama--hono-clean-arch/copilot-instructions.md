## hono-clean-arch

> We are building a backend API using Clean Architecture


## Architecture

We are building a backend API using Clean Architecture

---

## Core Folders structure

- **Modules** : contains feature modules (user, quotes, etc.), each module has its own clean architecture layers.
- **Shared**  : contains shared code across modules, with clean architecture layers.

---

## Architecture Layers

- **Api layer**            : contains controllers, validators, routes, middlewares, webhooks, depends on Application and Infrastructure layers, external frameworks and libraries.
- **Application layer**    : contains use cases, dto, mappers, services and business rules. depends on Domain layer only, no external frameworks and libraries.
- **Domain layer**         : contains entities, value objects, and repository interfaces, must not depend on any other layer, no external frameworks and libraries.
- **Infrastructure layer** : contains concrete implementations of repositories, mappers, depends on Domain layer only, external frameworks and libraries.

---

## Technology Stack

- **Runtime**          : Bun v1.3
- **Framework**        : Hono v4 + TypeScript v5
- **ORM**              : Drizzle ORM v0.45
- **Database**         : Neon (PostgreSQL)
- **Auth**             : Clerk v3
- **Validation**       : Zod v4
- **Other**            : svix
- **Hosting / Deploy** : Vercel

---

## Database Tables

- **users**  : (id, providerId, firstName, lastName, email, image, role, createdAt, updatedAt)
- **quotes** : (id, userId, author, description, createdAt, updatedAt)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/berkanioussama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
