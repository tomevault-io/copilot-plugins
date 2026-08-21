## go-standards

> Go coding standards for Cerebro — error handling, logging, context, concurrency


# Go Coding Standards

## Error Handling

Always wrap errors with context; never silently discard them.

```go
// ❌ BAD
result, _ := doThing()
if err != nil { return err }

// ✅ GOOD
result, err := doThing()
if err != nil {
    return fmt.Errorf("doThing: %w", err)
}
```

Sentinel errors live in the same package as the function that produces them, prefixed `Err`:
```go
var ErrOrderRejected = errors.New("order rejected by risk gate")
```

## Logging

Use `log/slog` (stdlib structured logging). Always pass context and structured key-value pairs:

```go
slog.InfoContext(ctx, "order submitted", "symbol", symbol, "qty", qty, "side", side)
slog.ErrorContext(ctx, "broker unavailable", "err", err, "attempt", attempt)
```

Never use `fmt.Println` or `log.Printf` in production paths.

## Context

- Every function in a hot path must accept `ctx context.Context` as the **first parameter**.
- Respect cancellation: check `ctx.Err()` inside loops and before I/O.
- Never store a context in a struct field.

## Concurrency

- Use `errgroup.WithContext` (from `golang.org/x/sync/errgroup`) for goroutine fan-out.
- Protect shared state with a `sync.Mutex` or channel — document which pattern and why.
- Goroutines must respect context cancellation to allow clean shutdown.

## Money / Decimal

**Never use `float64` for prices, quantities, or PnL.** Always use `github.com/shopspring/decimal`:

```go
// ❌ BAD
price := 42000.5 * 0.01

// ✅ GOOD
price := decimal.NewFromFloat(42000.5)
fee := price.Mul(decimal.NewFromFloat(0.01))
```

## General

- Prefer table-driven tests with `t.Run`.
- Unexported helpers over large exported APIs.
- Build tags for optional features: `//go:build metrics`.
- `golangci-lint` must pass with zero warnings before merging.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
