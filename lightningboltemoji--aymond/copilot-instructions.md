## aymond

> **aymond** is a type-safe, batteries-included Rust client wrapper for AWS DynamoDB, inspired by the DynamoDB Enhanced Java Client. It uses procedural macros to generate compile-time-safe CRUD and query builders from annotated Rust structs.

## Project Overview

**aymond** is a type-safe, batteries-included Rust client wrapper for AWS DynamoDB, inspired by the DynamoDB Enhanced Java Client. It uses procedural macros to generate compile-time-safe CRUD and query builders from annotated Rust structs.

## Workspace Structure

Three crates:
- **`aymond/`** — Main library: `HighLevelClient`, trait definitions, shim re-exports
- **`aymond-derive/`** — Procedural macro crate that generates all table/item code
- **`aymond-test/`** — Integration tests and compile-fail (trybuild) tests

## Commands

Don't declare anything 'finished' before running the following:

```bash
# Build all crates
cargo build
# Format/lint
cargo fmt
cargo clippy --fix --allow-dirty
# Integration tests
cargo test -p aymond-test
```

## Architecture

### Macro → Generated Code Flow

The `#[aymond(...)]` attribute macro (in `aymond-derive/src/lib.rs`) generates all boilerplate:

- `#[aymond(item)]` → Item trait impl + From/Into HashMap<String, AttributeValue> converters
- `#[aymond(nested_item)]` → From/Into converters only (no key requirements)
- `#[aymond(item, table)]` → All of the above + `{Name}Table` struct with typed builders

### Generated Types per Item

For a struct named `Car` with `#[aymond(item, table)]`:
- `CarTable(client, table_name)` — implements the `Table` trait with CRUD + query methods
- `CarGetItemHashKey` → optional `CarGetItemSortKey` → `send()` returns `Option<Car>`
- `CarPutItem` → optional `CarPutItemCondition` → `send()`
- `CarQuery` → `CarQueryHashKey` → optional `CarQuerySortKey` → async `Stream<Car>`

The multi-stage builder pattern enforces correctness at **compile time** (e.g., hash key must be set before sort key).

### Core Traits (`aymond/src/traits.rs`)

- `Item`: Top-level DynamoDB items; provides `key_schemas()` and `key_attribute_definitions()`
- `NestedItem`: Marker trait for embeddable structs (stored as DynamoDB `M` map)
- `Table<'a, T, G, GHK, P, Q, QHK>`: Generic over item type and all builder types

### Derive Macro Internals (`aymond-derive/src/`)

| File | Purpose |
|------|---------|
| `definition.rs` | Parses struct attributes into `ItemDefinition` / `ItemAttribute` |
| `item.rs` | Generates `Item` trait impl |
| `marshal.rs` | Generates From/Into HashMap converters |
| `table.rs` | Generates `{Name}Table` struct + `Table` trait impl |
| `get_item.rs` | Generates GET builder chain |
| `put_item.rs` | Generates PUT builder + condition expression builder |
| `query.rs` | Generates QUERY builder chain with streaming pagination |
| `nested_item.rs` | Generates NestedItem converters |

### Attribute Annotations

```rust
#[aymond(item, table)]
struct Car {
    #[aymond(hash_key)]
    make: String,
    #[aymond(sort_key)]
    model: String,
    #[aymond(attribute(name = "yr"))]  // custom DynamoDB attribute name
    year: i32,
    color: Option<String>,     // Option<T> = optional DynamoDB attribute
}

#[aymond(nested_item)]
struct Address {
    street: String,
}
```

Rules enforced by macro:
- `#[aymond(hash_key)]` field is required for `item`; cannot be `Option<T>`
- `#[aymond(sort_key)]` field is optional; cannot be `Option<T>`
- `nested_item` structs have no key requirements

### Type Mapping

| Rust | DynamoDB |
|------|----------|
| `String` | `S` |
| `i8`–`i128`, `u8`–`u128` | `N` |
| `Vec<T>` | `L` |
| `Option<T>` | conditional insert/absence |
| `struct` with `#[aymond(nested_item)]` | `M` |

### Streaming Queries

Query results return `impl Stream<Item = Result<T>>` using `futures::unfold` for lazy pagination — not a `Vec`. Callers iterate with `StreamExt::next()` or collect.

### Transactions

```rust
client.tx()
    .put(table.put(item1))
    .put(table.put(item2))
    .send()
    .await?;
```

`PutItem` builders implement `Into<Put>` for use in `transact_write_items`.

### Shim Module

`aymond/src/shim.rs` re-exports AWS SDK crates so the generated macro code can reference `aymond::shim::aws_sdk_dynamodb` without requiring users to add those deps directly.

## Testing

Two-tier approach in `aymond-test/`:

1. **Integration tests** (`integ/`): Run against DynamoDB Local on port 8000 — cover CRUD, queries, streaming, option attributes, numeric keys
2. **Compile-fail tests** (`shouldnt_compile/`): Use `trybuild` to assert that invalid macro usage produces expected compiler errors

---
> Source: [lightningboltemoji/aymond](https://github.com/lightningboltemoji/aymond) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
