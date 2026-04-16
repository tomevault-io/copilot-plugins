## ultimate-asepharyana-tech-rust

> Internal technical overview of the **Scraping & CDN Service** (`apps/rust`) for automated data extraction and image persistence.

# GEMINI.md - Codebase Architecture & Structure

Internal technical overview of the **Scraping & CDN Service** (`apps/rust`) for automated data extraction and image persistence.

## 🌍 Context

- **`apps/rust`**: Specialized backend engine (Axum).
- **`apps/solidjs`**: Frontend consumer.
- **`packages/services`**: Shared logic.

## 🤖 AI Assistant Guidelines

AI assistants (like Claude, Gemini, GPT) interacting with this codebase **MUST** adhere to the following protocols defined in **[AGENT.md](file:///mnt/code/bp3/ultimate-asepharyana.tech/apps/rust/AGENT.md)**:

1. **Professional Tone**: Maintain a cold, technical, and objective tone.
2. **No Hyperbole**: **PROHIBITED** from using marketing-speak or exaggerated praise (e.g., "amazing", "unparalleled", "powerful", "revolutionary").
3. **Technical Accuracy**: Focus purely on implementation facts, data structures, and performance metrics.
4. **Minimalist Adherence**: Always prioritize the **Zero-Bloat Policy**. If a request introduces unnecessary dependencies or logic, challenge the user and suggest a leaner alternative.
5. **Documentation Consistency**: Ensure any generated documentation follows the established professional and objective style of the `docs/` folder.

## 🦀 `apps/rust` - Backend Service

An asynchronous service for scraping and image proxying. All secondary web framework features (Authentication, Social, GraphQL) have been removed to reduce complexity.

### 📊 Tech Stack

- **Framework**: [Axum](https://github.com/tokio-rs/axum) (0.8.8) - Asynchronous Rust HTTP.
- **ORM**: [SeaORM](https://www.sea-ql.org/SeaORM/) (MySQL) - Database abstraction.
- **Caching**: `deadpool-redis` & `redis` - In-memory cache mapping.
- **Observability**: [Prometheus](https://prometheus.io/) - Performance instrumentation.
- **Scraping**: `scraper` (CSS Selectors) & remote Chrome via HTTP.

### 📂 Directory Structure (`apps/rust/src`)

Organized as a hybrid of Vertical Slice and Clean Architecture.

| Directory | Description |
| :--- | :--- |
| **`bin/`** | Binary entry points and CLI tools. |
| **`config/`** | Strongly-typed environment configuration. |
| **`entities/`** | **SeaORM Entities**. Database schema mapping. |
| **`routes/`** | **API Handlers**. Automatic routing system. |
| **`services/`** | Business logic (e.g., `ImageCache` service). |
| **`scraping/`** | Data extraction engines and parsers. |
| **`helpers/`** | Shared utilities and cache helpers. |
| **`middleware/`** | Axum layers (CORS, Compression, Metrics). |
| **`events/`** | Internal event bus for repair state updates. |
| **`jobs/`** | Background task processing. |
| **`scheduler/`** | Periodic tasks (Daily CDN audit). |
| **`observability/`** | Prometheus metrics and latency tracking. |

### 🔑 Logic Flows

1. **Scraping**:
   `Request` -> `Router` -> `Handler` -> `Scraper Engine` -> `Redis` -> `Response`.
2. **Image Proxy/CDN**:
   `Request` -> `ImageCache` -> `Cache Lookup` -> `Picser Upload (on miss)` -> `CDN URL`.
3. **Metrics**:
   `GET /metrics` -> `Prometheus Exporter` -> `Text Response`.

### 📜 Commands

- **Standard Run**: `cargo run`
- **Optimized Build**: `cargo build --release`
- **External Audit**: `POST /api/proxy/image-cache/audit`

## 🏗 Maintenance Constraints

- **Minimalist Approach**: New dependencies require impact evaluation.
- **Lint Compliance**: Suppression flags (`#[allow]`) are prohibited.
- **Performance-First**: Latency regressions are monitored via `/metrics`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MythEclipse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
