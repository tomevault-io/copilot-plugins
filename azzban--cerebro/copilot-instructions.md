## hexagonal-architecture

> Hexagonal architecture conventions — domain, port, adapter boundaries


# Hexagonal Architecture

## Layer Rules

| Layer | Package | Rule |
|---|---|---|
| **Domain** | `internal/domain` | Pure Go types and constants. **Zero external imports.** No business logic. |
| **Ports** | `internal/port` | Interfaces only. One file per abstraction group (`broker.go`, `store.go`, …). |
| **Adapters** | `internal/adapter/<system>/` | Implements a port. Imports the port interface, not sibling adapters. |
| **App** | `internal/app` | Composition root only. Wires ports to adapters. No business logic here. |
| **Domain logic** | `internal/risk`, `internal/execution`, `internal/strategy`, `internal/agent` | Depends on domain + ports; never on adapters. |

## Dependency Direction

```
cmd → cli → app → [domain, port, risk, execution, strategy, agent]
                        ↑
                    adapter (implements port)
```

Adapters **must not** import each other. If two adapters need shared behaviour, extract it into a `pkg/` utility.

## Adding a New Port

1. Define the interface in `internal/port/`.
2. Write the adapter in `internal/adapter/<system>/`.
3. Register it in `internal/app/runtime.go` — no other file should do wiring.

```go
// internal/port/notifier.go
type Notifier interface {
    Notify(ctx context.Context, msg domain.Alert) error
}

// internal/adapter/telegram/notifier.go
type Notifier struct { bot *tgbotapi.BotAPI; chatID int64 }
func (n *Notifier) Notify(ctx context.Context, msg domain.Alert) error { … }
```

## Domain Types

- Enums are **typed string constants**, not `iota`:
  ```go
  type Side string
  const (
      SideBuy  Side = "BUY"
      SideSell Side = "SELL"
  )
  ```
- All monetary values use `decimal.Decimal`.
- IDs use `uuid.UUID`.

## What Doesn't Belong Where

- No `pgx` / Redis imports inside `internal/domain`, `internal/port`, or any strategy/risk/execution package.
- No business decisions inside adapters — they translate, not decide.

---
> Source: [AzzBAN/cerebro](https://github.com/AzzBAN/cerebro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
