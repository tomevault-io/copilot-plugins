## go-pinecone

> - **Exported identifiers:** `PascalCase` (types, functions, constants, variables)


# Naming Conventions

- **Exported identifiers:** `PascalCase` (types, functions, constants, variables)
- **Unexported identifiers:** `camelCase`
- **Acronyms:** Keep uppercase in exported identifiers (`URL`, `HTTP`, `API`, `ID`, `gRPC` → `GRPC`)
- **Constants/enums:** `PascalCase` (Go convention; e.g., `Cosine`, `Dotproduct`, `Euclidean`)
- **Test functions:** Suffix unit tests with `Unit` (e.g., `TestNewClientParamsSetUnit`)
- **Request structs:** Suffix with `Request` (e.g., `CreateServerlessIndexRequest`)
- **Parameters:** Short, contextual names in function bodies (`ctx`, `err`, `idx`, `pc`) are idiomatic
- **Client instances:** Use `pc` for `Client` instances in examples and tests
- **Files:** `snake_case` (e.g., `admin_client.go`, `index_connection.go`)

---
> Source: [pinecone-io/go-pinecone](https://github.com/pinecone-io/go-pinecone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
