## leptos-store

> Enterprise-grade, type-enforced state management for Leptos (Rust). Use when implementing stores, getters, mutators, actions, selectors, persistence, SSR hydration, middleware, audit trails, or store composition in a Leptos application.


# leptos-store — Agent Skill

State management for Leptos with enforced Enterprise Mode: getters (public read) → actions (public write) → mutators (private write) → state.

## Requirements

- Leptos 0.8+
- Rust 1.92+ (2024 edition)
- leptos-store 0.7

## Critical Rules

1. **Use `this` not `self`** in `store!` macro bodies — Rust 2024 edition macro hygiene requires it.
2. **Import explicitly** — never `use leptos::prelude::*;` in files using selectors. Leptos 0.8 has a deprecated `create_selector` that causes ambiguity.
3. **Enterprise Mode is the core invariant** — components call actions (public) → actions call mutators (private) → mutators update `RwSignal<State>`. Never bypass.
4. **Always `provide_store()` in a parent** before `use_store()` in children.
5. **Wrap store access in `move ||` closures** inside `view!` macros for reactivity.

## Quick Start

```rust
use leptos::prelude::*;
use leptos_store::prelude::{Store, provide_store, use_store};

store! {
    pub CounterStore {
        state CounterState {
            count: i32 = 0,
        }
        getters {
            count(this) -> i32 {
                this.read(|s| s.count)
            }
        }
        mutators {
            set_count(this, value: i32) {
                this.mutate(|s| s.count = value);
            }
        }
        actions {
            increment(this) {
                let current = this.count();
                this.set_count(current + 1);
            }
        }
    }
}

#[component]
fn App() -> impl IntoView {
    provide_store(CounterStore::new());
    view! { <Counter /> }
}

#[component]
fn Counter() -> impl IntoView {
    let store = use_store::<CounterStore>();
    let inc = store.clone();
    view! {
        <p>{move || store.count()}</p>
        <button on:click=move |_| inc.increment()>"+"</button>
    }
}
```

## Feature Gates

```
csr                — Client-only SPA (no server rendering)
ssr                — Server-side rendering (default)
hydrate            — SSR + client hydration (adds serde + web-sys ~50KB WASM)
persist-web        — localStorage / sessionStorage persistence
persist-idb        — IndexedDB persistence (implies persist-web)
persist-server     — Server-side persistence (implies ssr)
middleware         — Mutation interception, audit trails (adds js-sys)
devtools           — Browser devtools integration (implies middleware)
templates          — Feature flag system (implies hydrate)
tracing            — OpenTelemetry integration (implies middleware)
full               — All features enabled
```

### Dependency Chain

```
templates ──→ hydrate ──→ serde, serde_json, web-sys, wasm-bindgen
persist-idb → persist-web → web-sys, wasm-bindgen, serde, serde_json, base64, js-sys
devtools ───→ middleware ──→ js-sys
tracing ────→ middleware ──→ js-sys
```

### Cargo.toml Examples

```toml
# Client-only SPA
leptos-store = { version = "0.7", features = ["csr"] }

# Standard SSR app
leptos-store = { version = "0.7", features = ["ssr", "hydrate"] }

# With persistence
leptos-store = { version = "0.7", features = ["ssr", "hydrate", "persist-web"] }

# Enterprise (full stack)
leptos-store = { version = "0.7", features = ["full"] }
```

## Decision Guide

### How Many Stores?

- **Small app** → One store via `store!` macro → [creating-a-store](docs/skills/01-creating-a-store.md)
- **Multiple domains** (auth, cart, UI) → `RootStore` composition → [store-composition](docs/skills/07-store-composition.md)
- **Team boundaries** → `RootStore` + `namespace!` macro → [store-composition](docs/skills/07-store-composition.md)

### What Features?

- **Derived/computed values** → Selectors (`create_selector`, `combine_selectors`) → [selectors](docs/skills/06-selectors.md)
- **API calls / loading states** → Async actions (`spawn_local`) → [async-actions](docs/skills/03-async-actions.md)
- **Save state across sessions** → Persistence adapters → [persistence](docs/skills/04-persistence.md)
- **Server-rendered + interactive** → SSR hydration → [ssr-hydration](docs/skills/05-ssr-hydration.md)
- **Feature flags / A/B testing** → Template system → [feature-flags](docs/skills/02-feature-flags.md)
- **Audit trail / debugging** → Middleware + AuditTrail → [middleware](docs/skills/08-middleware.md)
- **Cross-store reactions** → EventBus + StoreCoordinator → [middleware](docs/skills/08-middleware.md)

## Skills Index

| Skill | Topic |
|-------|-------|
| [01-creating-a-store](docs/skills/01-creating-a-store.md) | `store!` macro, state, getters, mutators, actions |
| [02-feature-flags](docs/skills/02-feature-flags.md) | Template system, `<Feature>` component, A/B testing |
| [03-async-actions](docs/skills/03-async-actions.md) | `AsyncAction`, `spawn_local`, loading states |
| [04-persistence](docs/skills/04-persistence.md) | localStorage, IndexedDB, `PersistentStore` |
| [05-ssr-hydration](docs/skills/05-ssr-hydration.md) | `HydratableStore`, server → client state transfer |
| [06-selectors](docs/skills/06-selectors.md) | `create_selector`, `map_selector`, `combine_selectors` |
| [07-store-composition](docs/skills/07-store-composition.md) | `RootStore`, `StoreDependency`, `MultiStoreSelector` |
| [08-middleware](docs/skills/08-middleware.md) | `MiddlewareStore`, `AuditTrail`, `StoreCoordinator` |
| [architecture-guide](docs/skills/architecture-guide.md) | Decision trees, feature selection, Cargo.toml templates |
| [troubleshooting](docs/skills/troubleshooting.md) | Common errors indexed by symptom |

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `ambiguous import create_selector` | Glob import from `leptos::prelude` | Use `use leptos_store::prelude::create_selector;` |
| `cannot find value self` in `store!` | Rust 2024 macro hygiene | Use `this` instead of `self` |
| `Store not found in context` | Missing `provide_store()` in parent | Add `provide_store()` before `use_store()` |
| State not updating in UI | Missing reactive closure | Wrap in `move \|\|` inside `view!` |
| WASM bundle too large | `hydrate` feature when not needed | Use `csr` for client-only apps |

---
> Source: [nyvorin/leptos-store](https://github.com/nyvorin/leptos-store) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
