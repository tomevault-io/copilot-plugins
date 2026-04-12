## brightpath-coder

> Backend development conventions including service structure, dependency injection, middleware patterns, error propagation, and clean architecture.


# Backend Standards

## Service Structure
- Follow clean architecture: handlers → services → repositories
- Business logic in service layer; no framework dependencies
- Repository pattern for data access; interface-driven for testability

## Error Handling
- Wrap errors with context at every layer: fmt.Errorf("doing X: %w", err)
- Never swallow errors; no bare catch{} or _ = err
- Use typed errors for business logic; HTTP status mapping in handlers only
- Log errors at the boundary (handler); propagate below

## Dependency Injection
- Dependencies injected via constructor, not global state
- Use interfaces for external dependencies (DB, HTTP clients, queues)
- Config loaded once at startup; passed explicitly, not read from env mid-request

## Middleware
- Authentication, logging, tracing, rate limiting as middleware
- Middleware order matters: auth before business logic, logging wraps all
- Request context carries request ID, user info, trace span

## Concurrency
- Protect shared state with mutexes or channels
- Use context.Context for cancellation and timeouts
- Set timeouts on all external calls; downstream < upstream < client

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cdalsoniii) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
