## testing-standards

> Testing standards and patterns for Cerebro


# Testing Standards

## Test Types

| Tag | Purpose | Command |
|---|---|---|
| _(none)_ | Unit tests — no external deps, pure in-memory | `make test` |
| `integration` | Requires Binance testnet, Postgres, Redis | `make test-int` |

Always use build tags to separate them:

```go
//go:build integration
```

## Table-Driven Tests

All unit tests use table-driven style with `t.Run`:

```go
func TestRiskGate_Allow(t *testing.T) {
    tests := []struct {
        name    string
        order   domain.OrderIntent
        wantErr bool
    }{
        {"within limits", validOrder, false},
        {"exceeds max notional", bigOrder, true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            gate := risk.NewGate(testConfig)
            err := gate.Allow(context.Background(), tt.order)
            if (err != nil) != tt.wantErr {
                t.Errorf("Allow() err = %v, wantErr %v", err, tt.wantErr)
            }
        })
    }
}
```

## Mocking Ports

Never mock concrete adapters. Test against the port interface using hand-written stubs or `testify/mock`:

```go
type stubBroker struct{ submitted []domain.OrderIntent }
func (s *stubBroker) Submit(ctx context.Context, o domain.OrderIntent) error {
    s.submitted = append(s.submitted, o)
    return nil
}
```

The stub lives in `internal/<package>/testhelpers_test.go` (unexported, test-only).

## Assertions

Use stdlib `testing` + comparison with `decimal.Equal` for money. Avoid assertion libraries unless they're already in `go.mod`.

## What Must Have Tests

- Every function in `internal/risk/` — risk gate logic is safety-critical.
- Every `port` implementation (adapter unit tests with a fake/in-memory upstream).
- Every strategy signal generator.
- CLI flag parsing and config validation paths.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
