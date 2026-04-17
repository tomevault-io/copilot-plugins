## go-pubbing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**go-pubbing** is a production-grade, single-instance in-memory pub/sub system for Go with hierarchical topics, wildcard subscriptions, and optional statefulness. The motto is: **Excellence. Always.**

### Phase 1 Implementation Status ✅

**COMPLETED** - Phase 1 (Exact Topic Matching) - First dev cycle

All core infrastructure and fundamental features are implemented and tested:

#### ✅ Implemented Features
- **Broker**: Full lifecycle (New, Publish, Subscribe, Shutdown)
- **Topic System**: Hierarchical topics with exact matching (no wildcards yet)
- **Subscriptions**: Context-aware, handler-based, automatic cleanup
- **Messages**: Immutable with defensive copies, sequence numbers, timestamps, headers
- **Options**: Functional options for broker, publish, and subscribe
- **Logging**: Injectable logger interface with Zap adapter
- **Error Handling**: Rich error types with context
- **Testing**: 79 tests, 100% pass, race detection enabled
- **Performance**: 6M+ topic matches/sec, 157ns/op
- **Documentation**: Comprehensive README, 11 runnable examples

#### Test Coverage
- Main package: 67.0%
- Adapters: 100.0%
- Internal: 69.6%
- Topic system: 98.7%

#### ⏳ Planned for Future Phases
- **Phase 2**: Wildcard patterns (`*` and `>`)
- **Phase 3**: Message retention and replay
- **Phase 4**: Acknowledgment and delivery guarantees
- **Phase 5**: Queue groups and load balancing

### Core Philosophy
- **Single-threaded broker loop**: No locks, all mutable state owned by one goroutine ✅
- **Safety first**: No panics (except init), recover from user panics, no goroutine leaks ✅
- **Zero external dependencies**: Pure Go (except testing dependencies) ✅
- **Command pattern**: All state changes via commands sent to broker loop ✅
- **Future-proof**: Architecture supports multi-instance extension ✅

## Critical Style Requirements

**NO LITERAL STRINGS IN CODE** - Always use CONSTANTS for:
- Error messages
- Configuration keys
- Topic patterns
- Field names in maps/structs
- Any string that appears in code logic

Bad:
```go
if headers["content-type"] == "application/json" { ... }
```

Good:
```go
const (
    HeaderContentType = "content-type"
    MimeTypeJSON = "application/json"
)
if headers[HeaderContentType] == MimeTypeJSON { ... }
```

## Architecture Overview

### Threading Model
```
User Code → Broker API (thread-safe) → Command Channels → Central Broker Loop (single goroutine)
                                                           ↓
                                                    Topic Trie + Retention + Subscribers
                                                           ↓
                                              Subscriber Goroutines (one per subscription)
```

**Key principle**: Broker loop is single-threaded and owns ALL mutable state. No locks needed in loop. External API is thread-safe via channels.

### Core Components

1. **Broker** (`broker.go`, `broker_internal.go`)
   - Public API: `Publish()`, `Subscribe()`, `Request()`, `Shutdown()`
   - Internal loop: Processes commands sequentially
   - Uses channels for communication (cmdCh, stopCh, stoppedCh)

2. **Topic Trie** (`topic/trie.go`)
   - Efficient O(segments) wildcard matching
   - Supports `*` (single segment) and `>` (multi-level)
   - Pre-structured at subscribe time, not publish time
   - RWMutex because reads >> writes

3. **Retention Store** (`retention/store.go`, `retention/ringbuffer.go`)
   - Per-topic ring buffers
   - Lazy allocation (only when messages published)
   - Policy inheritance: specific > wildcard > global
   - Bounded memory by design

4. **Subscription** (`subscription.go`)
   - Each has own goroutine for isolation
   - Supports callback OR channel (not both)
   - Handles ack/nack, redelivery, panic recovery

5. **Message** (`message.go`)
   - ID: NanoID (not UUID - shorter)
   - Sequence: uint64 (global monotonic via atomic)
   - Payload: []byte (zero-copy)
   - Headers: map[string]string

6. **Commands** (`internal/command.go`)
   - publishCmd, subscribeCmd, unsubscribeCmd, ackCmd, shutdownCmd, statsCmd
   - Synchronous commands use result channels
   - Async commands (ack) are fire-and-forget

### Topic Rules
- Segments separated by `.` (dot)
- No empty segments: `mission..state` is INVALID
- No leading/trailing dots: `.mission` or `mission.` is INVALID
- `*` matches exactly one segment: `mission.*.change.*`
- `>` matches one or more segments, must be last: `mission.>`
- Only valid characters: `a-zA-Z0-9-_`

## Common Development Commands

### Setup
```bash
# Initialize module (if not done)
go mod init github.com/itsatony/go-pubbing

# Get dependencies (minimal - only test deps)
go mod tidy
```

### Testing
```bash
# Run all tests
go test ./...

# Run with race detector (ALWAYS use during development)
go test -race ./...

# Run with coverage
go test -cover ./...

# Run specific test
go test -run TestBrokerPublishSubscribe ./...

# Run benchmarks
go test -bench=. -benchmem ./...

# Run benchmarks for specific component
go test -bench=BenchmarkTopicMatching ./topic/
```

### Building
```bash
# Vet code
go vet ./...

# Format code
go fmt ./...

# Build (library, no binary)
go build ./...
```

### Profiling
```bash
# CPU profile
go test -cpuprofile=cpu.prof -bench=BenchmarkHighThroughput
go tool pprof cpu.prof

# Memory profile
go test -memprofile=mem.prof -bench=BenchmarkPublish
go tool pprof mem.prof
```

## Implementation Guidelines

### Memory Management
- Use `sync.Pool` for Message objects to reduce GC pressure
- Limit channel buffer sizes to prevent unbounded growth
- Clean up closed subscriptions promptly
- Pre-allocate slices when size is known

### Error Handling
- Define specific error types in `errors.go`
- Use `errors.New()` for simple errors
- Use custom error types for rich context (see `PubSubError` in plan)
- Never panic in user-facing code (only in init/New)
- Wrap errors with operation context

### Thread Safety
- **Broker.Publish()**: Safe for concurrent use
- **Broker.Subscribe()**: Safe for concurrent use
- **Subscription.Unsubscribe()**: Safe for concurrent use
- **Message**: Immutable after creation (except ack/nack methods)
- **Internal state**: Protected by single-threaded broker loop (no locks!)

### Testing Strategy
- Test files parallel the source files: `broker_test.go`, `topic/trie_test.go`
- Use `testutil/` for shared test helpers
- Always test with `-race` flag
- Write benchmarks for hot paths
- Use `examples_test.go` for godoc examples
- Test categories:
  - Basic functionality (pub/sub, unsub)
  - Topic matching (wildcards, patterns)
  - Statefulness (retention, replay, durable)
  - Reliability (ack/nack, redelivery, timeouts)
  - Queue groups (load balancing)
  - Performance (backpressure, slow consumers)
  - Concurrency (race conditions, deadlocks)
  - Shutdown (graceful, timeout)

### Code Organization
```
go-pubbing/
├── broker.go                    # Public API
├── broker_internal.go           # Broker loop implementation
├── options.go                   # Functional options
├── message.go                   # Message type
├── subscription.go              # Subscription type
├── errors.go                    # Error types
├── hooks.go                     # Observability hooks
├── stats.go                     # Statistics
├── topic/
│   ├── trie.go                  # Topic matching
│   └── validation.go            # Topic/pattern validation
├── retention/
│   ├── store.go                 # Retention store
│   ├── ringbuffer.go            # Ring buffer
│   └── policy.go                # Retention policies
├── internal/
│   ├── command.go               # Command types
│   ├── nanoid.go                # ID generation
│   └── pool.go                  # Object pooling
└── testutil/
    └── testutil.go              # Test helpers
```

## Key Algorithms

### Topic Matching (in Topic Trie)
- Recursive descent through trie
- Try exact match → single wildcard (*) → multi-wildcard (>)
- Multi-wildcard matches current and ALL remaining segments
- Pre-compiled patterns at subscribe time for efficiency

### Ring Buffer
- Fixed capacity circular buffer per topic
- Overwrites oldest when full
- Tracks start sequence (oldest) and end sequence (next write)
- O(1) push and range retrieval

### Queue Group Load Balancing
- Group subscribers by queue group name
- Fan-out to all non-queue subscribers
- Round-robin within each queue group (simple: `msgSequence % len(members)`)
- Future: Consider least-loaded or weighted distribution

### Request/Reply
- Generate unique inbox: `_INBOX.<nanoID>`
- Create ephemeral subscription to inbox
- Publish with `ReplyTo` header set to inbox
- Wait for reply with timeout
- Auto-unsubscribe when done

## Patterns and Best Practices

### Functional Options
```go
// Always use functional options for configuration
broker, err := pubbing.New(
    pubbing.WithRetention(10000, 1*time.Hour),
    pubbing.WithBackpressure(pubbing.BackpressureModeDropOldest),
)

err = broker.Publish("topic", payload,
    pubbing.WithHeaders(map[string]string{"trace-id": "123"}),
    pubbing.WithReplyTo("response.topic"),
)
```

### Context Usage
```go
// Always pass context to Subscribe
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

sub, err := broker.Subscribe(ctx, "pattern", handler)
// Subscription automatically unsubscribes when ctx is cancelled
```

### Graceful Shutdown
```go
// In shutdown:
// 1. Stop accepting new publishes
// 2. Wait for pending acks (with timeout)
// 3. Cancel all subscriptions
// 4. Wait for subscriber goroutines to exit
// 5. Close broker loop

broker.Shutdown(5 * time.Second)
```

### Handler Panic Recovery
```go
// Subscription goroutine MUST recover from handler panics
func (s *Subscription) safeExecuteHandler(msg *Message) (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("handler panic: %v", r)
            // Log stack trace
        }
    }()
    return s.handler(msg)
}
```

## Performance Targets

- **Single pub/sub**: 500k+ msgs/sec
- **10x10**: 250k+ msgs/sec
- **100x100**: 100k+ msgs/sec
- **Latency P50/P99**: <100µs / <1ms
- **Per subscription overhead**: <1KB
- **Topics**: 100k+ topics
- **Subscribers**: 10k+ active subscriptions

### Optimization Strategies
1. Minimize allocations (sync.Pool, pre-allocate)
2. Lock-free paths (atomics, single-threaded loop)
3. Efficient data structures (trie, ring buffer, maps)
4. Profile before optimizing (use pprof)

## Common Pitfalls to Avoid

1. **DON'T use locks in broker loop** - It's single-threaded by design
2. **DON'T block in subscriber handlers** - They block delivery
3. **DON'T create unbounded channels** - Set buffer limits
4. **DON'T forget to unsubscribe** - Causes goroutine leaks
5. **DON'T panic in production code** - Only in New() for invalid config
6. **DON'T use literal strings** - ALWAYS use constants
7. **DON'T assume ordering across topics** - Only per-topic ordering guaranteed
8. **DON'T forget race detector** - ALWAYS run tests with `-race`

## Future Extensions (v2+)

The architecture is designed to support multi-instance/distributed operation:
- Storage abstraction (Redis, S3 backends)
- Partition strategy (PartitionID + LocalSequence)
- Broker clustering (Raft/etcd)
- Distributed queue groups
- Sequence number evolution to (PartitionID, Sequence) tuples

Keep this in mind when making design decisions - don't paint into corners.

## Quick Reference

### Message Flow
```
Publish → Broker API → publishCmd → Broker Loop → Topic Trie Match
  → Dispatch to Subscribers → Subscriber Goroutines → Handler/Channel
  → Ack/Nack → Broker Loop → Update State
```

### Subscription Types
- **Ephemeral**: No replay, only new messages
- **Durable**: Named, tracks position, can replay
- **Callback**: Handler function (auto-ack based on return)
- **Channel**: User reads from channel (manual ack)
- **Queue Group**: Load balanced across members

### Default Values
```go
DefaultRetentionMessages = 10000
DefaultRetentionAge      = 1 * time.Hour
DefaultAckTimeout        = 30 * time.Second
DefaultMaxRedelivery     = 3
DefaultBufferSize        = 100
```

## Documentation Standards

- Every exported type/function/constant MUST have godoc comment
- Comments are complete sentences starting with the name
- Explain WHY, not WHAT (code shows what)
- Mark TODOs with reason: `// TODO(username): Fix race condition when... #123`
- Write runnable examples in `examples_test.go`

## License

MIT License - See LICENSE file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itsatony) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
