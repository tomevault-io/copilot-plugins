## pulse

> Device telemetry ingest service in Go. Fixed memory, bounded queue, live

# pulse

Device telemetry ingest service in Go. Fixed memory, bounded queue, live
WebSocket fan-out.

## Commands

```bash
make test     # go test -race ./...   <- always run this before committing
make cover    # coverage summary
make bench    # store benchmarks
make vet      # go vet ./...
make run      # server on :8080
make load     # simulated device fleet against a running server
```

## Invariants

These are load-bearing. Changing any of them changes what the service promises.

1. **`ingest.Submit` must never block.** It is a non-blocking channel send. If
   it blocks, backpressure gets expressed as HTTP connection timeouts instead
   of a `429`, and the client's timeout config decides our shedding policy.
2. **`stream.Hub.Broadcast` must never block.** Non-blocking send per
   subscriber; a full buffer drops the frame and increments a counter. If it
   blocks, one slow WebSocket client stalls ingest for the whole fleet.
3. **Store memory is bounded per device, not globally.** Each device has a
   fixed ring buffer. Do not introduce a shared global buffer; one chatty
   device would evict everyone else's history.
4. **Shutdown order is HTTP, then queue drain, then subscribers.** Reversing
   the first two drops work already acknowledged with a `202`.

Two tests exist specifically to guard 1 and 2:
`TestSubmitShedsWhenQueueFull` and `TestSlowConsumerDropsInsteadOfBlocking`.
If either starts failing, the fix is the code, not the test.

## Conventions

- Go 1.22. Routing uses stdlib `http.ServeMux` method+wildcard patterns. Do not
  add a router dependency.
- One third-party dependency (`gorilla/websocket`). Justify any addition.
- Exported identifiers get doc comments. Comments explain why, not what.
- Conventional Commits for messages: `feat:`, `fix:`, `test:`, `docs:`,
  `refactor:`, `perf:`, `chore:`, `ci:`.
- `gofmt` clean, `go vet` clean, tests pass under `-race`. CI enforces all three.

---
> Source: [Ohimoiza1205/pulse](https://github.com/Ohimoiza1205/pulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
