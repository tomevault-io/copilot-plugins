## runtime-wiring

> Composition root patterns and goroutine lifecycle in internal/app


# Runtime Wiring (`internal/app`)

## Sole Responsibility

`internal/app` is the **composition root only**. It:
- Instantiates adapters (broker, cache, stores, notifiers).
- Wires them to business services via port interfaces.
- Launches goroutines via `errgroup.WithContext`.
- Handles shutdown on context cancellation.

**No business logic lives here.** If you find yourself adding a trading decision to `runtime.go`, it belongs in `internal/strategy`, `internal/risk`, or `internal/execution`.

## Goroutine Pattern

All long-running goroutines follow this template:

```go
g.Go(func() error {
    slog.InfoContext(ctx, "component starting", "name", "market-hub")
    if err := hub.Run(ctx); err != nil && !errors.Is(err, context.Canceled) {
        return fmt.Errorf("market hub: %w", err)
    }
    return nil
})
```

- Always check `context.Canceled` / `context.DeadlineExceeded` and return `nil` for them — they are clean shutdowns, not errors.
- `errgroup` cancels the group context on the first non-nil error; all goroutines must react promptly.

## Paper vs Live Guard

The live broker path **must** remain behind an explicit guard until fully implemented and audited:

```go
if a.cfg.Environment == domain.EnvironmentProduction {
    return fmt.Errorf("live broker not yet implemented — use --paper")
}
```

Never remove this guard without:
1. A complete adapter implementation.
2. Integration tests passing on Binance testnet.
3. Risk gate coverage for the live path.

## In-Memory vs Real Adapters

Currently the paper path uses in-memory stores (`newMemoryCache`, `newMemoryTradeStore`, etc.). When wiring real Postgres/Redis adapters:

- Instantiate the adapter (e.g. `postgres.NewTradeStore(pool)`).
- Assign it to the port interface variable.
- Do **not** change any code outside `internal/app` — ports guarantee the swap is transparent.

## Startup Sequence

1. Load + validate config.
2. Connect external services (DB, Redis, exchange WS) — fail fast here.
3. Build domain services with injected ports.
4. Start goroutines via `errgroup`.
5. Block until `ctx` is cancelled or a fatal error propagates.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
