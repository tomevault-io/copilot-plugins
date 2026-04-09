## ai-bridge-router

> - "Always use Axum for HTTP server setup."


- "Always use Axum for HTTP server setup."
- "Use Tokio as async runtime; avoid spawning blocking tasks without tokio::task::spawn_blocking."
- "Use SQLx with Postgres driver and connection pooling."
- "Organize code into modules: config, db, cache, routes, services, models, utils."
- "Use dotenvy and config crate for env management, never hardcode secrets."
- "All APIs must return JSON via serde::Serialize models."
- "Error handling should use thiserror + anyhow for context-rich errors."
- "Logging must be via tracing, not println!."
- "Tests should be in /tests with integration focus (health_check, routes)."
- "Use feature flags in Cargo.toml for optional components (ml_scoring, telemetry)."
- "When you implement and it doesn't work, break it down to why it is not working and don't restort to simpler approach"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/am-miracle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/am-miracle)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
