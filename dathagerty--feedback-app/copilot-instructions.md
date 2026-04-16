## feedback-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Rust web application for collecting user feedback via shareable links. Built with Axum web framework, SQLite database, and Askama templates.

**Key workflow:**
1. Admin creates feedback prompts (title + description)
2. System generates unique shareable URL per prompt
3. Users submit feedback via the URL
4. Admin views all feedback responses for each prompt

## Development Commands

### Build and Run
```bash
# Build project
cargo build

# Run application (starts on http://localhost:3000)
cargo run

# Run in release mode
cargo build --release
cargo run --release
```

### Testing
```bash
# Run all tests
cargo test

# Run specific test
cargo test test_name

# Run tests with output
cargo test -- --nocapture

# Run tests for specific module
cargo test db::tests
cargo test tests::test_admin_list
```

### Code Quality
```bash
# Format code
cargo fmt

# Check formatting without making changes
cargo fmt -- --check

# Run linter
cargo clippy

# Run linter with warnings as errors
cargo clippy -- -D warnings
```

## Architecture

### Application Structure

**Two-module design:**
- `src/main.rs`: Web server, routes, handlers, templates, and integration tests
- `src/db.rs`: Database layer with SQLite operations and unit tests

**Data model:**
- `Prompt`: Feedback prompt with UUID, title, description, timestamp
- `Feedback`: User submission linked to prompt with UUID, content, timestamp

### Web Framework (Axum)

**State management:**
- `AppState` holds SQLite connection pool (`SqlitePool`)
- Shared via `Arc<AppState>` across handlers
- Extracted using `State(state): State<Arc<AppState>>`

**Route structure:**
```
GET  /                      → Redirect to /admin
GET  /admin                 → List all prompts
GET  /admin/new             → New prompt form
POST /admin/new             → Create prompt
GET  /admin/prompt/:id      → View prompt and feedback responses
GET  /feedback/:id          → Public feedback form
POST /feedback/:id          → Submit feedback
```

**Handler patterns:**
- Use `impl IntoResponse` for return types
- Extract path params with `Path(id): Path<String>`
- Extract forms with `Form(form): Form<FormStruct>`
- Extract host header with `Host(host): Host`

### Templates (Askama)

Templates in `templates/` directory use Askama's Jinja2-like syntax:
- `base.html`: Base layout with embedded CSS
- Admin templates: `admin_list.html`, `admin_new.html`, `admin_detail.html`
- Public templates: `feedback_form.html`, `feedback_success.html`

**Template usage:**
```rust
#[derive(Template)]
#[template(path = "template_name.html")]
struct TemplateName {
    field: Type,
}
```

### Database (SQLx + SQLite)

**Connection:**
- Uses `SqlitePool` for connection pooling
- Compile-time checked queries via sqlx macros
- Database URL: `DATABASE_URL` env var or `sqlite:feedback.db?mode=rwc`

**Query patterns:**
```rust
// Parameterized queries (SQL injection safe)
sqlx::query("INSERT INTO table (col) VALUES (?)")
    .bind(value)
    .execute(pool)
    .await?

// Fetch with mapping
sqlx::query_as::<_, Struct>("SELECT * FROM table WHERE id = ?")
    .bind(id)
    .fetch_optional(pool)  // or fetch_all() or fetch_one()
    .await?
```

### Testing Strategy

**Two test suites:**

1. **Database tests** (`src/db.rs`):
   - Use in-memory SQLite: `sqlite::memory:`
   - Test CRUD operations in isolation
   - Verify ordering, isolation between prompts

2. **Integration tests** (`src/main.rs`):
   - Test full HTTP request/response cycle
   - Use `tower::ServiceExt::oneshot()` for request simulation
   - Parse response bodies to verify HTML content
   - Setup helper: `setup_test_app()` creates fresh in-memory database

**Test patterns:**
```rust
#[tokio::test]
async fn test_name() {
    let (app, state) = setup_test_app().await;

    let response = app
        .oneshot(
            Request::builder()
                .uri("/path")
                .body(Body::empty())
                .unwrap(),
        )
        .await
        .unwrap();

    assert_eq!(response.status(), StatusCode::OK);
}
```

## Important Implementation Details

### UUID Generation
All IDs use UUID v4 via `uuid::Uuid::new_v4().to_string()`

### Timestamp Format
All timestamps use RFC3339 format via `chrono::Utc::now().to_rfc3339()`

### URL Generation
Admin detail page generates shareable URLs dynamically:
- Detects localhost/127.0.0.1 → uses `http://`
- Otherwise uses `https://`
- Format: `{protocol}://{host}/feedback/{prompt_id}`

### Router Creation
`create_router()` function is extracted for testability - both main app and tests use it with different state instances.

## Database Schema

```sql
CREATE TABLE prompts (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    created_at TEXT NOT NULL
);

CREATE TABLE feedback (
    id TEXT PRIMARY KEY,
    prompt_id TEXT NOT NULL,
    content TEXT NOT NULL,
    created_at TEXT NOT NULL,
    FOREIGN KEY (prompt_id) REFERENCES prompts(id)
);
```

Both tables order results by `created_at DESC` (newest first).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dathagerty) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
