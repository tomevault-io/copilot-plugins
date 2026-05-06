## rust-webapp-rule

> Here’s an improved and expanded version of your **Rust Web Application Development Rule**, with refinements for clarity, consistency, and completeness. I’ve also added several **recommended practices** based on real-world Rust web development experience and modern ecosystem trends:

Here’s an improved and expanded version of your **Rust Web Application Development Rule**, with refinements for clarity, consistency, and completeness. I’ve also added several **recommended practices** based on real-world Rust web development experience and modern ecosystem trends:

---

# 📦 Rust Web Application Development Guidelines

## 📁 Project Structure

Follow a modular, domain-driven architecture with clearly separated concerns:

```
app/
├── src/
│   ├── main.rs              # App entry point: initialize server, tracing, config
│   ├── router.rs            # Centralized routing logic
│   ├── handler/             # HTTP request handlers per domain
│   ├── service/             # Business logic and application services per domain
│   ├── models/              # Database models (input/output structs, schema use)
│   ├── extensions/          # Middlewares, extractors, guards, utils
│   └── config.rs            # Typed configuration loading/validation
├── migrations/              # Diesel migrations (organized per domain if needed)
├── crates/                  # Workspace crates (e.g., shared types, db schemas)
│   └── database_schema/     
│       └── enum_definitions.rs  # Shared enums used in DB layer
└── .env                     # Environment variables
```

## ✅ Compilation and Linting

* All code **must compile** without warnings or errors (linter errors)
* Prefer `cargo fix` for resolving simple compiler suggestions

## 🗃️ Database & ORM

* Use **Diesel** for type-safe SQL and **diesel\_async** for async queries (`RunQueryDsl` trait)
* For complex queries or batch operations, consider **SeaQuery**
* Use **domain-specific services** (in `service/`) to encapsulate DB logic
* Migrations:

  * Use `diesel_migrations`
  * Separate logical concerns across migration files
* Never accept `created_at`, `updated_at`, `tenant_id` in user input models — they are:

  * Auto-generated in DB
  * Tenant ID is inserted via the connection pool for row-level security
* Enum definitions must live in `crates/database_schema/enum_definitions.rs` and be shared via workspace crates

## ❗ Error Handling

* Define a centralized `AppError` type (implement `std::error::Error`, `Display`, `From`)
* Use a type alias `pub type AppResult<T> = Result<T, AppError>;`
* Never use `unwrap()` or `expect()` in production code — prefer `?`
* Map errors to meaningful HTTP status codes in route handlers
* Log errors with structured tracing before returning to client

## 🔐 Row-Level Security

* `tenant_id` must not be manually inserted
* Inject `tenant_id` automatically into queries using a custom connection wrapper or Diesel helper

## ⚙️ Configuration Management

* Use `dotenv` crate for layered configuration
* Define strongly-typed config structs and validate them on startup
* Use [`secrecy::Secret<T>`](mdc:https:/docs.rs/secrecy) for sensitive fields like passwords or API keys
* Fail fast on invalid or missing configuration

## 🧪 Testing and Observability

* Organize tests:

  * Unit tests in the same file as module
  * Integration tests in `tests/` folder

---
> Source: [jmerelnyc/elkar-tasks](https://github.com/jmerelnyc/elkar-tasks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
