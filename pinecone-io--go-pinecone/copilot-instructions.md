## go-pinecone

> The SDK uses a **3-plane architecture**: Control Plane (`Client`), Admin Plane (`AdminClient`), and Data Plane (`IndexConnection`). Each plane has its own client struct defined in a dedicated file. Keep the public API surface in `pinecone/`; the `internal/` packages are never imported by users.


# Architecture

The SDK uses a **3-plane architecture**: Control Plane (`Client`), Admin Plane (`AdminClient`), and Data Plane (`IndexConnection`). Each plane has its own client struct defined in a dedicated file. Keep the public API surface in `pinecone/`; the `internal/` packages are never imported by users.

**Key files:**
- `pinecone/client.go` — `Client`: Control Plane + Inference
- `pinecone/admin_client.go` — `AdminClient`: Admin operations
- `pinecone/index_connection.go` — `IndexConnection`: Data Plane (gRPC)
- `pinecone/models.go` — All shared types, constants, enums
- `pinecone/errors.go` — `PineconeError` error type
- `internal/gen/` — Auto-generated clients (do not edit)

**Generated vs. manual code:** All client code in `internal/gen/` is auto-generated from OpenAPI/Protobuf specs. Manual SDK code lives entirely in `pinecone/` and `internal/provider/`, `internal/useragent/`, `internal/utils/`.

**Transport:** REST (`net/http`) for Control Plane, Inference, and Admin; gRPC (`google.golang.org/grpc`) for Data Plane.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pinecone-io)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pinecone-io)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
