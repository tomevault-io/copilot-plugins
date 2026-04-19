## toska-mesh-go

> This directory contains the Go runtime SDK for services joining the ToskaMesh service mesh.

# CLAUDE.md — toska-mesh-go (Go Runtime SDK)

This directory contains the Go runtime SDK for services joining the ToskaMesh service mesh.

## What This Is

`toska-mesh-go` is a thin client library that Go services use to join the mesh. It handles:
- Auto-registration with Discovery (gRPC)
- Health endpoint (`/health`)
- Heartbeat/TTL renewal
- Graceful deregistration on shutdown
- Metadata propagation (routing strategy, weight, scheme, health endpoint)

The control plane itself lives in `../toska-mesh/`. This is just the Go SDK for service authors.

## Build, Test, and Run

```bash
make test       # go test -race ./...
make build      # build example binaries → bin/
make generate   # regenerate Go protobuf from ../toska-mesh-proto/
make lint       # golangci-lint (skipped if not installed)
make clean      # remove bin/

# Run the example service
go run ./examples/hello-mesh-service
```

## Module

```
module github.com/toska-mesh/toska-mesh-go
```

## Directory Layout

```
toska-mesh-go/
├── pkg/
│   ├── runtime/          # MeshService builder (the public API)
│   │   ├── mesh.go       # MeshService struct and lifecycle
│   │   └── options.go    # ServiceOptions and functional options
│   └── meshpb/           # generated protobuf Go code (do not edit)
├── examples/
│   └── hello-mesh-service/main.go
├── Makefile
└── go.mod
```

## Usage

```go
svc, err := runtime.New(
    runtime.WithServiceName("my-service"),
    runtime.WithPort(9090),
    runtime.WithDiscoveryAddress("discovery:8080"),
    runtime.WithRoutingStrategy(runtime.RoundRobin),
    runtime.WithMetadata("version", "1.0.0"),
)
if err != nil { log.Fatal(err) }

svc.HandleFunc("GET /hello", helloHandler)

if err := svc.Run(ctx); err != nil { log.Fatal(err) }
```

## Coding Conventions

Same as `../toska-mesh/CLAUDE.md`: standard Go style, errors as values, context as first parameter, table-driven tests.

## Protobuf

Generated from `../toska-mesh-proto/discovery.proto`. Run `make generate` after modifying the proto.

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `MESH_SERVICE_NAME` | `mesh-service` | Service name for discovery |
| `MESH_SERVICE_PORT` | `8080` | HTTP bind port |
| `MESH_DISCOVERY_ADDRESS` | `localhost:8080` | Discovery gRPC address |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abstractivemachines) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
