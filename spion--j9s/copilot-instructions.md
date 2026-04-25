## j9s

> - Rust formatting: 100 columns, 2-space indentation (see `rustfmt.toml`)

# j9s Development Notes

## Code Style

- Rust formatting: 100 columns, 2-space indentation (see `rustfmt.toml`)
- Run `cargo fmt` before committing

## Coding approach

- Concise: Highly prioritize brevity, but without sacrificing clarity.
  - Existing code may be more verbose than desired. When writing new code or refactoring, prefer the concise style over matching surrounding patterns.
- Abstractions: valued when they are very general, powerful and/or have practical benefit.
  - Its okay if some abstractions are forward-looking: the code is evolving and sometimes we build a bit of the foundations before the need.
  - Prefer using less powerful method of describing abstractions if its still possible to get great clarity and benefit, but aim for a concise and clear language of expression.
  - The abstraction quality bar is high - if you would be embarassed to put the abstraction in a README.md its probably not worth it.
  - Modularity. Abstractions should be able to stand largely on their own, whenever possible.
- Comments: prefer self-documenting code.

## Project Structure

- `src/app.rs` - Main app state and event loop
- `src/event.rs` - Event types (Key, Tick)
- `src/query.rs` - Query<T> for async data fetching
- `src/config.rs` - XDG-compliant config loading
- `src/ui/` - All UI code (see below)
- `src/jira/` - Gouqi wrapper and types
- `src/db/` - SQLite connection management
- `src/cache/` - Generic caching layer (see below)

## UI Architecture

```
src/ui/
├── mod.rs           # Main draw function
├── view.rs          # View trait and ViewAction enum
├── components/      # Stateful components with co-located rendering and event handling
├── views/           # Toplevel views (screens) structs with co-located rendering, stackable
└── renderfns/       # Purely stateless render functions
```

**Delegation chain:** App → View → Component

- **App** owns command mode (`:` prefix), delegates to current view
- **Views** own their modes (e.g. search with `/`), implement `View` trait
- **Components** are reusable input handlers with `KeyResult<T>` pattern
- **renderfns** are stateless - just take data and render, no state

When adding a new view: create in `ui/views/`, implement `View` trait, co-locate rendering.
When adding a new component: create in `ui/components/`, co-locate rendering with the component.

## Component Key Handling with KeyResult<T>

Components use `KeyResult<T>` for consistent key event handling:

```rust
pub enum KeyResult<T> {
  Handled,      // Key consumed, no event for parent
  Event(T),     // Key consumed, here's an event for parent
  NotHandled,   // Key not consumed, try next handler
}
```

Each component defines its own event enum (e.g. `FilterBarEvent`, `SearchEvent`).

**Views use or_else chains** to delegate keys through the component stack:

```rust
fn handle_key(&mut self, key: KeyEvent) -> ViewAction {
  self
    .handle_overlays(key)
    .or_else(|| self.handle_navigation(key))
    .or_else(|| self.handle_toggles(key))
    .or_else(|| self.handle_actions(key))
    .unwrap_or(ViewAction::None)
}

fn handle_overlays(&mut self, key: KeyEvent) -> Option<ViewAction> {
  match self.search.handle_key(key) {
    KeyResult::Handled => return Some(ViewAction::None),
    KeyResult::Event(SearchEvent::Submitted(q)) => { /* use q */ }
    KeyResult::Event(SearchEvent::Cancelled) => return Some(ViewAction::None),
    KeyResult::NotHandled => {}
  }
  // ... more components
  None
}
```

**When adding a new component:**
1. Define an event enum: `pub enum FooEvent { Selected(T), Cancelled }`
2. Implement `handle_key(&mut self, key: KeyEvent) -> KeyResult<FooEvent>`
3. Component returns `NotHandled` when inactive, handles keys when active
4. Parent view matches on the result in its `handle_overlays` method

## Navigation

Uses k9s-style navigation:
- View stack with push/pop semantics
- `:` commands replace root view
- `Enter` pushes detail views
- `Escape`/`q` pops back

## Data Fetching with Query<T> and Fetched<T>

Views own their data loading via `Query<T>` (inspired by TanStack Query):

```rust
// View creates and owns its query — closures returning Result<T, String> or Fetched<T> both work
let mut query = Query::new(move || {
    let jira = jira.clone();
    async move { jira.search_issues(&jql).await }
});
query.fetch();  // Start loading immediately

// In tick() - poll for results
fn tick(&mut self) {
    self.query.poll();
}

// In render() - use query.data() unconditionally, query.state() for indicators
match self.query.state() {
    QueryState::Loading => render_spinner(),
    QueryState::Success => render_data(query.data()),
    QueryState::Error(e) => render_error(e),
    QueryState::Idle => {}
}
```

**Error + cached data coexistence**: `Fetched<T>` expresses three outcomes:
- `Fresh(data)` — network success → `cached_data = data`, `state = Success`
- `Stale(data, error)` — cache hit + network error → `cached_data = data`, `state = Error`
- `Error(msg)` — no data available → `cached_data` unchanged, `state = Error`

Query preserves `cached_data` in all error cases. Views should render `query.data()` unconditionally and use `query.state()` to show error/loading indicators. The cache layer returns `Stale` (not swallows errors) so Query can track and display them.

`From<Result<T, String>>` is implemented for `Fetched<T>`, so closures returning `Result` auto-convert.

**Key points:**
- Views take `JiraClient` in constructor and create their own queries
- `Query<T>` handles loading/success/error states internally
- App calls `tick()` on all views each tick to poll queries
- No event routing needed - views are self-contained
- Testable: create view with mock fetcher, call methods, check state

## Caching with CacheLayer

The `src/cache/` module provides transparent caching with offline support:

```
src/cache/
├── mod.rs      # Module exports
├── traits.rs   # Cacheable trait
├── layer.rs    # CacheLayer<S> - orchestrates caching logic, returns Fetched<T>
└── storage.rs  # CacheStorage trait, SqliteStorage
```

**Core concepts:**

- **Cacheable trait** - Entities must implement `cache_key()`, `updated_at()`, `entity_type()`
- **CacheLayer<S>** - Wraps a storage backend, always goes to network, returns `Fetched<T>`
- **CacheStorage trait** - Abstraction for storage backends (SQLite)

**Fetch strategies** — all return `Fetched<T>`:

```rust
// List fetch (network-first, cache fallback)
cache.fetch_list("boards:PROJECT", || async { jira.get_boards().await }).await

// Incremental fetch (only fetches entities updated since last sync)
cache.fetch_incremental("issues:jql", |since| async move {
    jira.search_issues_since(jql, since).await
}).await

// Single entity fetch
cache.fetch_one("issue:PROJ-123", || async { jira.get_issue(key).await }).await
```

**Key behaviors:**

- Network-first: always calls the fetcher, returns `Fresh(data)` on success
- Offline fallback: on network failure, returns `Stale(cached_data, error)` if cache exists
- Error propagation: returns `Error(msg)` when both network and cache fail
- Incremental sync: uses `updated_at > max_cached_updated_at` for efficient updates
- Storage errors are non-fatal (logged with `tracing::warn!`)

## Environment

- `J9S_JIRA_TOKEN` - Jira API token (required)
- Config at `~/.config/j9s/config.yaml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spion) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
