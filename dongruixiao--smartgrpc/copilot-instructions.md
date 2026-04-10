## smartgrpc

> `smartgrpc` is a small Python framework that lets developers declare gRPC APIs with:

# smartgrpc Agent Guide

## Project Goal

`smartgrpc` is a small Python framework that lets developers declare gRPC APIs with:

- `pydantic.BaseModel` for messages
- `enum.IntEnum` for enums
- Python classes and methods for services

The framework then:

1. registers decorated elements in `SmartGrpcApplication`
2. renders `.proto` files from Python declarations
3. compiles protobuf stubs with `grpcio-tools`
4. adapts protobuf requests/responses to Pydantic models on server and client sides

The current repo now has a productionized synchronous v1 core: schema generation, runtime adapters, compatibility checks, and regression tests are in place. Remaining gaps are mainly async/platform concerns and deeper socket-level integration coverage.

## Repository Layout

- `src/smartgrpc/application.py`
  Main entry point. Holds the registry, decorator helpers, proto generation, compilation, and server startup.
- `src/smartgrpc/decorators.py`
  Decorators for `@message`, `@enum`, `@service`, and `@interface`.
- `src/smartgrpc/registry.py`
  Collects declared elements and groups them by source file for rendering.
- `src/smartgrpc/render.py`
  Turns registered Python types into proto snippets.
- `src/smartgrpc/templates.py`
  Jinja2 templates used by the renderer.
- `src/smartgrpc/protos.py`
  Writes `.proto` files and invokes protobuf compilation.
- `src/smartgrpc/server.py`
  Builds runtime gRPC servicer adapters around user-defined service classes.
- `src/smartgrpc/client.py`
  Builds runtime client wrappers around generated gRPC stubs.
- `src/smartgrpc/convert.py`
  Converts between protobuf messages and Pydantic models, including oneof and well-known types.
- `src/smartgrpc/types.py`
  Scalar type aliases and runtime type tagging helpers.
- `examples/bookstore`
  Official example covering `include()`, cross-file references, and all 4 synchronous RPC shapes.
- `tests`
  Regression suite covering render, convert, runtime adapters, schema generation, compatibility, streaming, and well-known types.

## Intended Usage Flow

Typical usage in this repo is:

1. create `sg = SmartGrpcApplication()`
2. decorate models with `@sg.message`
3. decorate enums with `@sg.enum`
4. decorate service classes with `@sg.service`
5. decorate RPC methods with `@sg.interface`
6. call `sg.generate()`
7. call `sg.compile()`
8. run `sg.run()`
9. create a client with `create_client(service_class=..., app=sg, target=...)`

## Current Gaps And Risks

- async APIs, interceptors, TLS/auth helpers, and DI are not implemented.
- Arbitrary `Union[T1, T2, ...]` mapping is still intentionally unsupported beyond explicit `oneof_field(...)`.
- Socket-level real-network integration coverage is still thinner than adapter-level coverage.
- Example runs generate protobuf artifacts in-place; regenerate them from source rather than editing generated files manually.

## Working Rules For Future Changes

- Prefer changing library code under `src/smartgrpc` before editing generated `_pb2.py` files.
- If protobuf outputs need to change, regenerate them from source declarations rather than patching generated files manually.
- When changing message conversion behavior, verify both directions:
  - Pydantic -> protobuf
  - protobuf -> Pydantic
- When changing rendering behavior, validate with at least one example app and inspect the generated `.proto`.
- Preserve Python 3.12 syntax already used by the project.
- Prefer `rg` for code search.

## Local Dev Commands

- Install deps: `uv sync`
- Lint/format/type-check: `make format`
- Run example server: `uv run python examples/bookstore/server.py`
- Run example client: `uv run python examples/bookstore/client.py`

## Recommended Near-Term Milestones

1. Add socket-level integration tests around the bookstore example.
2. Design router-style composition on top of `include()`.
3. Add async/interceptor support.
4. Add service factory / DI support.
5. Expand production docs around deployment and compatibility workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dongruixiao)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dongruixiao)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
