## luminate

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Luminate is a high-cardinality observability system built in Go. The key architectural decision is a **storage abstraction layer** that supports two backends:
- **ClickHouse**: Horizontally scalable (3-10 pods), stateless, production-ready
- **BadgerDB**: Single instance, embedded, zero dependencies

## Build and Development Commands

### Common Commands
```bash
# Build
make build-local        # Build for current platform
make build             # Build for all platforms (Linux, macOS)

# Test
make test              # Run all tests
make test-coverage     # Generate coverage report
go test -v ./pkg/...   # Run tests for specific package

# Run single test
go test -v -run TestMetricValidation ./pkg/models

# Development
make run               # Build and run with default config
make dev               # Run with hot reload (requires air)

# Docker
make docker-build      # Build Docker image
make docker-run        # Run container locally

# Kubernetes
make k8s-deploy        # Deploy with ClickHouse (scalable)
make k8s-deploy-badger # Deploy with BadgerDB (single instance)
make k8s-logs          # View logs
make k8s-status        # Check deployment status
```

### Testing Specific Components
```bash
# Test storage interface
go test -v ./pkg/storage/...

# Test with race detector
go test -race ./...

# Benchmark
go test -bench=. -benchmem ./pkg/storage/badger
```

## Architecture Overview

### Core Abstraction: MetricsStore Interface

Located in [pkg/storage/interface.go](pkg/storage/interface.go:19-44):

```go
type MetricsStore interface {
    Write(ctx context.Context, metrics []Metric) error

    QueryRange(ctx context.Context, req QueryRequest) (QueryResult[MetricPoint], error)
    Aggregate(ctx context.Context, req AggregateRequest) (QueryResult[AggregateResult], error)
    Rate(ctx context.Context, req RateRequest) (QueryResult[RatePoint], error)

    ListMetrics(ctx context.Context, req DiscoveryRequest) ([]string, error)
    ListDimensionKeys(ctx context.Context, req DiscoveryRequest) ([]string, error)
    ListDimensionValues(ctx context.Context, req DiscoveryRequest) ([]string, error)

    Estimate(ctx context.Context, req StoreStatsRequest) (StoreStats, error)
    DeleteBefore(ctx context.Context, req DeleteRequest) error
    Health(ctx context.Context) error
    Close() error
}
```

**Key Design:** Single interface with two implementations. BadgerDB does bounded streaming/manual aggregations, ClickHouse uses native SQL aggregations. Discovery and delete operations are tenant-scoped at the interface boundary; query methods return execution metadata for result limits and planner feedback.

**Series Identity:** Use the WS2 canonical `models.SeriesHash` helper for cardinality, ClickHouse `series_hash UInt64`, INTEGRAL partitioning, and rollup full-series keys. Do not re-hash raw dimension maps independently.

### Data Model

See [pkg/models/metric.go](pkg/models/metric.go:11-26):

**Validation Rules:**
- Metric names: `^[a-zA-Z_][a-zA-Z0-9_]*$` (1-256 chars)
- Dimension keys: Same pattern (1-128 chars)
- Dimension values: 1-256 chars, UTF-8
- Timestamps: `[now - 7 days, now + 1 hour]`
- Values: Must be finite (no NaN/Inf)
- Max 20 dimensions per metric

### Aggregation Types

Nine types supported ([pkg/storage/interface.go](pkg/storage/interface.go:56-64)):
- Basic: AVG, SUM, COUNT, MIN, MAX
- Percentiles: P50, P95, P99 (for latency tracking)
- Time-weighted: INTEGRAL (for resource consumption like CPU-seconds)

## Implementation Guidelines

### When Implementing Storage Backends

**BadgerDB** ([pkg/storage/badger/](pkg/storage/badger/)):
- Design key schema for efficient time-range scans (e.g., `metric:<name>:<timestamp>:<hash(dimensions)>`)
- Implement aggregations using in-memory iterators
- Use prefix scans for filtering
- Focus on write throughput optimization
- Accept slower query performance vs ClickHouse

**ClickHouse** ([pkg/storage/clickhouse/](pkg/storage/clickhouse/)):
- Use batch inserts with `PrepareBatch()`
- Map aggregation types to SQL functions:
  - P95 → `quantile(0.95)(value)`
  - INTEGRAL → compute deltas with `lagInFrame(...) OVER (PARTITION BY series_hash ORDER BY timestamp)`; never use a global timestamp window across different series
- Use `Map(String, String)` for dimensions column
- Partition by `toYYYYMM(timestamp)`
- Use bloom filter indexes for high-cardinality dimensions

### API Handler Pattern

All handlers should follow this pattern:

```go
func (h *Handler) Query(w http.ResponseWriter, r *http.Request) {
    // 1. Parse JSON request
    var req JSONQuery
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        respondError(w, http.StatusBadRequest, "INVALID_REQUEST", err)
        return
    }

    // 2. Convert to storage request
    storageReq, err := req.ToStorageQuery(r.Context())
    if err != nil {
        respondError(w, http.StatusBadRequest, "INVALID_QUERY", err)
        return
    }

    // 3. Execute query
    result, err := h.store.QueryRange(r.Context(), storageReq)
    if err != nil {
        respondError(w, http.StatusInternalServerError, "QUERY_FAILED", err)
        return
    }

    // 4. Return JSON response
    respondJSON(w, http.StatusOK, result)
}
```

### Middleware Chain Order

In [cmd/luminate/main.go](cmd/luminate/main.go:54-61), middleware is applied in this order:

1. **RecoveryMiddleware** - Catch panics (outermost)
2. **LoggingMiddleware** - Log requests
3. **RateLimitMiddleware** - Rate limiting per tenant
4. **AuthMiddleware** - JWT validation (if enabled)
5. **Handler** - Route to API handler (innermost)

## Horizontal Scaling Architecture

### ClickHouse Deployment

**Key characteristics:**
- **Stateless pods**: No local state, all data in ClickHouse
- **Auto-scaling**: 3-10 replicas via HPA
- **Triggers**: 70% CPU or 80% memory
- **Anti-affinity**: Pods spread across nodes
- **Zero-downtime updates**: `maxUnavailable: 0`

**Files:**
- [deployments/kubernetes/deployment.yaml](deployments/kubernetes/deployment.yaml:1-157) - Main deployment
- [deployments/kubernetes/hpa.yaml](deployments/kubernetes/hpa.yaml:1-58) - Auto-scaling config
- [deployments/kubernetes/service.yaml](deployments/kubernetes/service.yaml:1-22) - Load balancer

### BadgerDB Deployment

**Key characteristics:**
- **Single replica**: StatefulSet with `replicas: 1`
- **Persistent storage**: 100GB SSD volume
- **Not horizontally scalable**: Embedded database

**Files:**
- [deployments/kubernetes/statefulset.yaml](deployments/kubernetes/statefulset.yaml:1-84) - StatefulSet config

## Configuration

Configuration uses YAML with environment variable substitution. See [pkg/config/config.go](pkg/config/config.go:13-51).

**Environment variable format:**
```yaml
auth:
  jwt_secret: "${JWT_SECRET}"  # Reads from env var
```

**Override via env:**
```bash
LUMINATE_SERVER_PORT=9090
LUMINATE_STORAGE_BACKEND=clickhouse
```

## API Design

### Single Unified Query Endpoint

All queries use `POST /api/v1/query` with `queryType` discriminator:

```json
{
  "queryType": "aggregate|range|rate",
  "metricName": "api_latency",
  "timeRange": {"relative": "1h"},
  "aggregation": "p95",
  "groupBy": ["customer_id"]
}
```

**Why single endpoint?** Simpler API surface, easier to version, consistent error handling.

### Time Range Formats

Support both absolute and relative:
```json
{"start": "2024-01-01T00:00:00Z", "end": "2024-01-01T23:59:59Z"}
{"relative": "1h"}  // Also: 24h, 7d, 30d
```

## Multi-Tenancy

**JWT-based isolation:**
- Extract `tenant_id` from JWT token
- Automatically add `_tenant_id` dimension to all metrics
- Filter all queries by tenant
- See [original CLAUDE.md](CLAUDE.md:1349-1428) for JWT structure

## Testing Strategy

### Unit Tests
- Mock `MetricsStore` interface for API handler tests
- Test validation logic in `pkg/models`
- Test config parsing with different env vars

### Integration Tests
- Spin up real BadgerDB instance
- Test full write → query cycle
- Test aggregations with known data

### Load Tests
- Validate 100K+ metrics/sec write throughput
- Verify auto-scaling triggers at 70% CPU
- Test graceful shutdown under load

## Common Patterns

### Error Handling
Always return structured errors:
```go
type ErrorResponse struct {
    Error struct {
        Code    string                 `json:"code"`
        Message string                 `json:"message"`
        Details map[string]interface{} `json:"details,omitempty"`
    } `json:"error"`
}
```

### Context Propagation
Always pass context through call chain:
```go
func (s *Store) Write(ctx context.Context, metrics []Metric) error {
    // Check context cancellation
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }

    // Pass context to underlying storage
    return s.db.Write(ctx, metrics)
}
```

### Timeout Handling
Respect query timeout from request:
```go
timeout := req.Timeout
if timeout == 0 {
    timeout = h.config.QueryLimits.DefaultQueryTimeout
}
if timeout > h.config.QueryLimits.MaxQueryTimeout {
    timeout = h.config.QueryLimits.MaxQueryTimeout
}

ctx, cancel := context.WithTimeout(r.Context(), timeout)
defer cancel()
```

## Performance Targets

### ClickHouse Backend
- Write: 100K+ metrics/sec per pod
- Query (p95): < 100ms
- Scaling: Linear to 10+ pods

### BadgerDB Backend
- Write: 10K+ metrics/sec
- Query (p95): < 500ms
- Scaling: Vertical only

## Debugging

### View logs in Kubernetes
```bash
kubectl logs -f -l app=luminate -n luminate
kubectl logs -f luminate-0 -n luminate  # For StatefulSet
```

### Check metrics
```bash
kubectl port-forward -n luminate svc/luminate-svc 8080:8080
curl http://localhost:8080/metrics
```

### Health check
```bash
curl http://localhost:8080/api/v1/health
```

## Important Constraints

1. **No joins**: Single metric per query (by design)
2. **Simple filters**: Only equality filters (no OR, NOT, ranges)
3. **Timestamp window**: Reject data >7 days old or >1 hour future
4. **Cardinality limits**: Max 20 dimensions, 1M unique series per metric
5. **Query timeout**: Hard cap at 30s

For advanced queries beyond interface, access backend directly.

## Project Structure

```
pkg/
├── storage/
│   ├── interface.go       # Core abstraction (CRITICAL)
│   ├── badger/           # BadgerDB implementation
│   └── clickhouse/       # ClickHouse implementation
├── models/
│   └── metric.go         # Data models + validation
├── config/
│   └── config.go         # YAML config + env vars
├── api/                  # HTTP handlers (to implement)
├── auth/                 # JWT auth (to implement)
└── middleware/           # HTTP middleware (to implement)

cmd/luminate/
└── main.go              # Entry point, server setup

deployments/
├── docker/Dockerfile     # Multi-stage build
└── kubernetes/
    ├── deployment.yaml   # Scalable (ClickHouse)
    ├── statefulset.yaml  # Single (BadgerDB)
    └── hpa.yaml          # Auto-scaling config
```

## References

- Full architecture & design decisions: [ARCHITECTURE.md](ARCHITECTURE.md)
- Deployment guide: [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md)
- Project structure: [PROJECT_STRUCTURE.md](PROJECT_STRUCTURE.md)

---
> Source: [sanketn26/luminate](https://github.com/sanketn26/luminate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
