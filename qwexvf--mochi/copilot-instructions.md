## mochi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**mochi** is a Code First GraphQL library for Gleam. It allows developers to define GraphQL schemas using Gleam types and automatically generates TypeScript types and GraphQL SDL.

## Commands

```bash
gleam build              # Build the library
gleam test               # Run all tests
gleam run                # Run main demo
gleam format src test    # Format code
gleam check              # Check for warnings (CI fails on warnings)
```

### Run specific test files
```bash
gleam run -m typescript_codegen_test  # TypeScript generation demo
gleam run -m sdl_codegen_test         # SDL generation demo
```

## Architecture

### Execution Flow
```
GraphQL Query String
    ↓ parser.gleam (lexer.gleam)
AST (ast.gleam)
    ↓ validation.gleam
Validated Document
    ↓ executor.gleam (uses input_coercion.gleam)
ExecutionResult → response.gleam → JSON
```

### Two Schema Building APIs

1. **High-level Code First API** (`query.gleam` + `types.gleam`): The recommended approach for building schemas with type-safe extractors.

2. **Low-level Schema API** (`schema.gleam`): Direct schema manipulation for advanced use cases.

### Core Modules

| Module | Purpose |
|--------|---------|
| `query.gleam` | Code First API - Query/Mutation/Subscription builders |
| `types.gleam` | Type builders with field extractors |
| `schema.gleam` | Core schema types, ExecutionContext, DataLoader integration |
| `parser.gleam` | GraphQL query parser (uses `lexer.gleam`) |
| `executor.gleam` | Query execution with null propagation |
| `validation.gleam` | Query validation against schema |
| `dataloader.gleam` | N+1 query prevention with batching |
| `error.gleam` | GraphQL-spec errors with path, location, extensions |
| `response.gleam` | Response construction and JSON serialization |
| `security.gleam` | Query depth/complexity limits, alias limits |
| `persisted_queries.gleam` | APQ with SHA256 hashing |
| `telemetry.gleam` | Execution telemetry hooks |

### Codegen Modules (`src/mochi/codegen/`)

| Module | Output |
|--------|--------|
| `typescript.gleam` | `.d.ts` TypeScript type definitions |
| `sdl.gleam` | `.graphql` Schema Definition Language |
| `gleam.gleam` | Gleam code generation from SDL |

### Transport Layer

| Module | Purpose |
|--------|---------|
| `transport/websocket.gleam` | WebSocket transport (graphql-ws protocol) |
| `transport/multipart.gleam` | Multipart request handling for file uploads |

## Key Concepts

### Code First API

The main API for defining schemas:

```gleam
import mochi/query
import mochi/types

// 1. Define Gleam type
pub type User {
  User(id: String, name: String, age: Int)
}

// 2. Create GraphQL type with extractors
let user_type = types.object("User")
  |> types.id("id", fn(u: User) { u.id })
  |> types.string("name", fn(u: User) { u.name })
  |> types.int("age", fn(u: User) { u.age })
  |> types.build(decode_user)

// 3. Define queries
let users_query = query.query("users", return_type, resolver, encoder)

// 4. Build schema
let schema = query.new()
  |> query.add_query(users_query)
  |> query.add_type(user_type)
  |> query.build
```

### Type Builders (`types.gleam`)

```gleam
types.object("TypeName")
|> types.description("...")
|> types.id("field", extractor)
|> types.string("field", extractor)
|> types.int("field", extractor)
|> types.float("field", extractor)
|> types.bool("field", extractor)
|> types.list_string("field", extractor)
|> types.build(decoder)
```

### Query Builders (`query.gleam`)

```gleam
// No args
query.query(name, return_type, resolver, encoder)

// With args (labeled arguments for clarity)
query.query_with_args(
  name: "user",
  args: [query.arg("id", schema.non_null(schema.id_type()))],
  returns: schema.named_type("User"),
  decode: fn(args) { query.get_id(args, "id") },
  resolve: fn(id, ctx) { get_user_by_id(id) },
  encode: fn(user) { types.to_dynamic(user) },
)

// Mutation (also uses labeled arguments)
query.mutation(
  name: "createUser",
  args: [query.arg("input", schema.non_null(schema.named_type("CreateUserInput")))],
  returns: schema.named_type("User"),
  decode: fn(args) { decode_create_user_input(args) },
  resolve: fn(input, ctx) { create_user(input) },
  encode: fn(user) { types.to_dynamic(user) },
)
```

### Argument Parsing Helpers (`query.gleam`)

```gleam
// Required arguments
query.get_string(args, "name")   // -> Result(String, String)
query.get_id(args, "id")         // -> Result(String, String)
query.get_int(args, "age")       // -> Result(Int, String)
query.get_float(args, "price")   // -> Result(Float, String)
query.get_bool(args, "active")   // -> Result(Bool, String)

// Optional arguments
query.get_optional_string(args, "name")  // -> Option(String)
query.get_optional_int(args, "limit")    // -> Option(Int)

// List arguments
query.get_string_list(args, "tags")  // -> Result(List(String), String)
query.get_int_list(args, "ids")      // -> Result(List(Int), String)
```

### Codegen

```gleam
import mochi/codegen/typescript
import mochi/codegen/sdl

// Generate TypeScript
let ts = typescript.generate(schema)

// Generate SDL
let gql = sdl.generate(schema)
```

## Examples

Working examples in `examples/`:
- `mochi_wisp/` - Full web app with Wisp framework, benchmarks
- `pokemon_api/` - DataLoader example with Pokemon data
- `core_library_examples/` - Pure library usage demos
- Single-file examples: `code_first_example.gleam`, `auth_example.gleam`, etc.

## Testing

- Uses `gleeunit` for unit tests
- Uses `birdie` for snapshot testing (Erlang-only due to platform-specific output formats)
- Snapshot tests use `@target(erlang)` attribute
- Snapshot files are in `birdie_snapshots/`

### Snapshot Testing Pattern
```gleam
@target(erlang)
pub fn my_feature_test() {
  result
  |> string.inspect
  |> birdie.snap(title: "Descriptive test title")
}
```

## Key Patterns

### Type-Safe Field Extractors
Fields use extractor functions that pull data from your Gleam types:
```gleam
types.string("name", fn(u: User) { u.name })  // extractor: User -> String
```

### Dynamic Conversion
- `types.to_dynamic(value)` - Convert any Gleam value to Dynamic for resolvers/encoders
- `types.record([...])` - Build Dynamic records with `types.field("key", value)`
- `types.option(value)` - Convert Option to Dynamic (None becomes null)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qwexvf)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qwexvf)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
