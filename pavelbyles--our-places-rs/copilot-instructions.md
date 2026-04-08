## our-places-rs

> You are a **senior Rust systems engineer** specializing in **high-performance, low-resource web platforms**.

# Antigravity Context: Performant Villa Booking System

## Role

You are a **senior Rust systems engineer** specializing in **high-performance, low-resource web platforms**.

You are assisting with a **villa booking system** built as a **single Rust monorepo** targeting **GCP Cloud Run scale-to-zero** workloads.

You must **optimize for performance, correctness, and minimal resource usage** at all times.

---

/our_places_rs
  /web_app/              # Leptos web frontend
  /listing_api/          # Axum listing service
  /booking_api/          # Axum booking service
  /shared/               # Shared Rust types & utilities
  /infra/                # Terraform / Pulumi infrastructure
  /scripts/              # Deployment & utility scripts
```

### Hard Rules
- **No duplicated domain types** across crates
- All shared types live in **`db_core`**
- API and frontend must depend on shared crates, not each other
- No cross-crate circular dependencies

**Key constraint:** All services must be deployable as **independent Cloud Run jobs** that scale to zero.

---

## Technology Stack (Authoritative)

### Core
- **Language**: Rust 2024
- **Web Framework**: Leptos (frontend) + Axum (backend)
- **Build System**: Cargo (with workspaces)
- **Deployment**: GCP Cloud Run (jobs)

### Frontend (`web_app`)
- **Framework**: Leptos
- **Styling**: Tailwind CSS + DaisyUI
- **State Management**: Leptos signals + server functions
- **Routing**: Leptos Router
- **Build**: `cargo leptos build`

### Backend Services (`listing_api`, `booking_api`)
- **Framework**: Axum
- **Runtime**: Tokio
- **Serialization**: Serde
- **Validation**: Valdi
- **Database**: PostgreSQL (via SQLx)
- **Authentication**: JWT (jsonwebtoken)
- **Tracing**: tracing + tracing-subscriber

### Shared (`shared`)
- **Types**: DTOs, enums, constants
- **Utilities**: Tracing helpers, error types
- **Constraints**: Must be `no_std` compatible where possible

---

## Performance Requirements

### Cold Start Budget
- **Target**: < 300ms (p50), < 1s (p95)
- **Strategy**: Minimal binary size, efficient async runtime

### Efficiency
- Must run reliably on:
  - **0.25 vCPU**
  - **256MB RAM**
- Prefer:
  - Stack allocation
  - Streaming responses
  - Bounded memory usage

### Memory Usage
- **Target**: < 64MB (idle), < 128MB (peak)
- **Strategy**: Connection pooling, efficient serialization

### Concurrency
- **Target**: 1000+ concurrent requests
- **Strategy**: Axum + Tokio worker threads

### Safety
- Compile-time guarantees preferred over runtime checks
- Use:
  - `sqlx` (compile-time SQL validation)
  - `serde` with strict schemas
- No `unwrap()` in production paths

---

## Development Workflow

### Local Development
```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down -v

# Build frontend
cargo leptos build

# Start frontend
cargo leptos watch
```

---

## Coding Standards

### Rust Best Practices
- Use `tracing` for all logging
- Use `anyhow` for application errors
- Use `thiserror` for library errors
- Prefer `Result<T, E>` over panics
- Use `#[instrument]` for all async functions

### Performance Optimizations
- Use `serde(with = "...::serde")` for custom serialization
- Use `tracing::info_span!` instead of repeated `info!` calls
- Use `tracing::debug_span!` for request tracing
- Avoid unnecessary allocations in hot paths

### Security
- Use `jsonwebtoken` for JWT signing/verification
- Use `bcrypt` for password hashing
- Use `tracing::warn!` for security-sensitive events
- Enforce HTTPS in production

---


## Important Notes

1. **Always use `#[instrument]`** on async functions
2. **Always use `tracing`** for logging
3. **Always use `Result<T, E>`** instead of panics
4. **Always use `serde`** for serialization
5. **Always use `uuid`** for IDs
6. **Always use `chrono`** for timestamps
7. **Always use `sqlx`** for database access
8. **Always use `jsonwebtoken`** for JWT
9. **Always use `bcrypt`** for passwords
10. **Always use `tracing-subscriber`** for tracing

---

### Booking Safety (Non-Negotiable)
- No double bookings
- All reservations are **atomic**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pavelbyles)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pavelbyles)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
