## link

> This rule summarizes important components and concepts identified in the project's knowledge graph.

# Key Project Entities & Concepts (Knowledge Graph Summary)

This rule summarizes important components and concepts identified in the project's knowledge graph.

## Core Components

*   **Project:** `Link Service` - Shortens URLs, generates previews, and creates QR codes. Core requirements in [PRD.md](mdc:PRD.md).
*   **API Definition:** [proto/link/v1/link.proto](mdc:proto/link/v1/link.proto) - Defines the `UrlService` using Protocol Buffers and ConnectRPC. Governed by `buf` ([buf.yaml](mdc:buf.yaml), [buf.gen.yaml](mdc:buf.gen.yaml)).
*   **Server Entry Point:** [server/cmd/main.go](mdc:server/cmd/main.go) - Initializes and starts the ConnectRPC server, including handlers for the `UrlService` and gRPC reflection.
*   **Service Implementation:** [server/internal/service/url_service.go](mdc:server/internal/service/url_service.go) - Contains the Go implementation of the `UrlService` RPC methods (currently stubs). Test file: [server/internal/service/url_service_test.go](mdc:server/internal/service/url_service_test.go).
*   **Build/Run:** [Makefile](mdc:Makefile) - Defines common tasks like `make proto`, `make run`, `make test`, `make tools`, `make docker-up`.
*   **Dependencies:** Dependencies like TiDB and Dragonfly are managed via [docker-compose.yaml](mdc:docker-compose.yaml).

## Key Technologies & Concepts

*   **RPC Framework:** `ConnectRPC` - Used for building the HTTP API based on the Protobuf definition. ([Go Getting Started](mdc:https:/connectrpc.com/docs/go/getting-started))
*   **Protobuf Tooling:** `buf` - Used for linting, generation, and managing Protobuf files.
*   **Server Introspection:** `gRPC Reflection` - Enabled in `server/cmd/main.go` allowing tools to discover the API structure.
*   **Testing Tools:**
    *   `grpcui`: A web UI for interacting with the server via reflection. Run with `grpcui -plaintext localhost:8080`. See [README.md](mdc:README.md).
    *   `grpcurl`: A command-line tool for interacting with the server via reflection. Useful for scripting and quick checks. See [README.md](mdc:README.md) for examples.
*   **Go Testing:** Use `make test` to run unit/integration tests (currently includes Ginkgo/Gomega setup). See [incremental-testing](mdc:.cursor/rules/incremental-testing.mdc).

## Documentation

*   **Project Overview/Usage:** [README.md](mdc:README.md)
*   **Tech Stack:** [TECH-STACK.md](mdc:TECH-STACK.md)
*   **Layout/Best Practices:** [LAYOUT-AND-BEST-PRACTICES.md](mdc:LAYOUT-AND-BEST-PRACTICES.md)
*   **Development Environment:** [development-environment](mdc:.cursor/rules/development-environment.mdc)

## Entities

These are the entities we can search the Knowledge Graph for:

Name,Type
Link Service,Project
proto/link/v1/link.proto,File (API Definition)
server/cmd/main.go,File (Server Entry Point)
server/internal/service/url_service.go,File (Service Implementation)
README.md,File (Documentation)
Makefile,File (Build/Run)
grpcui,Tool (Testing)
grpcurl,Tool (Testing)
buf,Tool (Protobuf)
ConnectRPC,Technology (RPC Framework)
gRPC Reflection,Technology (Server Capability)

This list will evolve over time and should be updated based on the latest knowledge in [knowledge_graph_summary.mdc](mdc:.cursor/rules/knowledge_graph_summary.mdc)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wcygan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wcygan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
