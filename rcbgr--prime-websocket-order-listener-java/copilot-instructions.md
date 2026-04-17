## prime-websocket-order-listener-java

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A standalone Spring Boot 3.5 application (Java 21) that connects to the **Coinbase Prime WebSocket feed** (`wss://ws-feed.prime.coinbase.com`) and listens to the `heartbeats` and `orders` channels. Products are partitioned into groups of up to 10, with a dedicated WebSocket connection per group. Received orders are buffered in a shared thread-safe queue and consumed by a background logger.

## Build & Run

```bash
# Build (skipping tests)
mvn package -DskipTests

# Run all tests
mvn test

# Run a single test class
mvn test -Dtest=PrimeMessageProcessorTest

# Run a single test method
mvn test -Dtest=PrimeMessageProcessorTest#checkSequence_triggersReconnectOnGap

# Run the application (requires env vars below)
mvn spring-boot:run
```

### Required environment variables

```bash
export ACCESS_KEY=...
export SIGNING_KEY=...
export PASSPHRASE=...
export PORTFOLIO_ID=...
```

## Architecture

### Technology stack

| Concern | Choice |
|---|---|
| Runtime | Java 21 (virtual threads via `Thread.ofVirtual()`) |
| Framework | Spring Boot 3.5 (no embedded web server — `web-application-type: none`) |
| WebSocket | Jakarta WebSocket API 2.2 backed by Tyrus 2.2 (Grizzly NIO transport) |
| JSON | Jackson (managed by Spring Boot BOM) |
| Logging | SLF4J + Logback, console only (no file appenders) |

### Package layout

```
com.coinbase.prime.samples
├── PrimeWebSocketApplication        # @SpringBootApplication entry point
├── config/
│   ├── CoinbasePrimeProperties      # @ConfigurationProperties("coinbase.prime")
│   └── WebSocketClientConfig        # @Bean ClientManager + ObjectMapper
├── model/
│   ├── PrimeMessage                 # Top-level inbound envelope (channel, sequence_num, events)
│   ├── Order                        # Individual order (all fields as String to preserve precision)
│   └── FeeDetails                   # Nested fee breakdown within an Order
├── service/
│   ├── SignatureService             # HMAC-SHA256 signing for subscribe/unsubscribe messages
│   ├── OrderQueueService            # LinkedBlockingQueue<Order>(5 000) — shared bounded buffer; enqueue() returns boolean for back-pressure
│   └── PrimeMessageProcessor        # Core business logic (sequence tracking, routing, reconnect trigger)
│                                    # NOT a Spring bean — one instance per PrimeConnectionWorker
├── websocket/
│   └── PrimeWebSocketEndpoint      # Jakarta @ClientEndpoint — NOT a Spring bean; created fresh per connection
├── connection/
│   ├── PrimeConnectionManager      # ApplicationRunner + DisposableBean; partitions products, owns workers
│   ├── PrimeConnectionWorker       # Per-connection loop (not a Spring bean)
│   ├── ConnectionRegistry          # @Service; holds ConnectionHealth for all workers, logs report every 60 s
│   ├── ConnectionHealth            # Mutable health record per worker (status + failure counts)
│   ├── ConnectionStatus            # Enum: CONNECTING, CONNECTED, CLOSED
│   └── FailureType                 # Enum: failure categories + FailureType.from(String) classifier
└── consumer/
    └── OrderConsumer                # Drains OrderQueueService on a virtual thread; logs each order
```

### Data flow

```
Coinbase Prime WSS (one connection per ≤10-product partition)
       │ raw JSON frames
       ▼
PrimeWebSocketEndpoint (@ClientEndpoint, per-connection instance)
       │ delegates
       ▼
PrimeMessageProcessor (one instance per PrimeConnectionWorker)
  ├── global sequence check → reconnect if gap
  ├── heartbeats  → debug log
  ├── orders      → OrderQueueService.enqueue()
  └── error frame → reconnect
       │
       ▼
OrderQueueService (LinkedBlockingQueue, 5 000 capacity, shared by all workers)
       │
       ▼
OrderConsumer (virtual thread, classifies and logs each order)
```

### Connection lifecycle

`PrimeConnectionManager.run()` partitions `product-ids` into batches of at most 10 and starts one `PrimeConnectionWorker` per batch. It then blocks on a `CountDownLatch` to keep the main thread alive (Spring Boot with `web-application-type: none` exits as soon as `run()` returns; virtual threads are daemon threads and do not prevent JVM exit).

Each `PrimeConnectionWorker` runs a connection loop on a dedicated virtual thread (`ws-connection-loop-N`):

1. Sets status to `CONNECTING`, creates a new `PrimeWebSocketEndpoint` instance.
2. Calls `ClientManager.connectToServer(endpoint, uri)` — blocks until the WebSocket handshake completes (connect timeout: 10 s).
3. Sets status to `CONNECTED`, resets sequence numbers, sends signed `subscribe` frames for `heartbeats` then `orders`.
4. Schedules a 30-second ping via `ScheduledThreadPoolExecutor`.
5. Awaits `endpoint.getCloseFuture()` — a `CompletableFuture<Void>` that completes in `@OnClose` or `@OnError`.
6. Attributes the failure reason (processor-triggered → sequence gap / server error; transport error → WebSocket error; otherwise → connection closed), records it in `ConnectionHealth`, then loops back with exponential back-off (initial 1 s, doubles to max 10 s). When the close reason is a queue-full, an additional 3-second pause is inserted before back-off resets, to allow the consumer to drain.

On Ctrl-C, Spring calls `PrimeConnectionManager.destroy()` which: releases the `CountDownLatch`, then calls `shutdown()` on each worker. Each worker sends `unsubscribe` frames, closes its session, and joins its loop thread.

### Reconnect triggers

| Trigger | Source | Failure reason |
|---|---|---|
| Session error / network drop | Tyrus `@OnError` | `"WebSocket error: <message>"` |
| Unexpected session close | Tyrus `@OnClose` | `"connection closed"` |
| Server-sent `{"type":"error"}` frame | `PrimeMessageProcessor` | `"server error: <message>"` |
| Sequence gap (received > expected+1) | `PrimeMessageProcessor` | `"sequence gap (dropped=N)"` |
| Order queue full | `PrimeMessageProcessor` | `"queue full (capacity=5000)"` — also triggers a 3 s pause before reconnect |
| `connectToServer()` throws | Tyrus | `"connect failed: <message>"` |

The reconnect callback is a `Consumer<String>` (not a `Runnable`) so the failure reason is passed through and recorded in `ConnectionHealth`.

Tyrus/Grizzly can internally catch `InterruptedException` and re-throw it as `DeploymentException`. The worker detects this via `hasCause(e, InterruptedException.class)` and exits the loop cleanly on shutdown.

### Sequence number tracking

The Coinbase Prime WebSocket uses a **single monotonically increasing counter per connection** shared across all channels. `PrimeMessageProcessor` tracks this with a single `AtomicLong lastSequence` (not a per-channel map).

- First message on the connection: accepted as baseline.
- `received == previous + 1`: normal — advance counter.
- `received < previous + 1`: out-of-order — warn, keep connection.
- `received > previous + 1`: **gap** — log error, trigger reconnect.

Counter is reset via `resetSequenceNumbers()` at the start of each new connection.

### Connection health monitoring

`ConnectionRegistry` (@Service) maintains a `ConnectionHealth` record for every worker. Workers update their record directly throughout their lifecycle.

Every 60 seconds `ConnectionRegistry.logReport()` logs one line per worker:

```
Connection health report (5 connection(s)):
  Worker 0  [BTC-USD,ETH-USD,...]   status=CONNECTED    failures=1  [SEQUENCE_GAP=1]  lastFailure=...  reason='sequence gap (dropped=2)'
  Worker 1  [LINK-USD,...]          status=CONNECTED    failures=0
```

`FailureType.from(String)` is the single classification point — all call sites go through it. Categories: `QUEUE_FULL`, `SERVER_ERROR_SLOW_CONSUME`, `SERVER_ERROR_SLOW_READ`, `SEQUENCE_GAP`, `CONNECTION_RESET`, `TLS_ERROR`, `DISCONNECT_REQUESTED`, `MESSAGE_TOO_BIG`, `UNKNOWN`.

### Order consumer log labels

`OrderConsumer` classifies each dequeued order into one of three labels:

| Label | Condition |
|---|---|
| `FILL` | `cumQty > 0` — the order has been (partially or fully) executed |
| `ORDER status changed` | `cumQty == 0` and status differs from the last recorded status for this order ID |
| `ORDER` | First time this order ID is seen |

Terminal status values (`FILLED`, `CANCELLED`, `EXPIRED`, `REJECTED`, `FAILED`) are removed from the tracking map to prevent unbounded memory growth.

### WebSocket client configuration (WebSocketClientConfig)

| Setting | Value | API |
|---|---|---|
| Max text message buffer | 1 MB | `ClientManager.setDefaultMaxTextMessageBufferSize` |
| Max binary message buffer | 1 MB | `ClientManager.setDefaultMaxBinaryMessageBufferSize` |
| Session idle timeout | 5 min | `ClientManager.setDefaultMaxSessionIdleTimeout` |
| Connect / handshake timeout | 10 s | `ClientProperties.HANDSHAKE_TIMEOUT` |
| Keep-alive | WebSocket ping every 30 s | `session.getAsyncRemote().sendPing()` via `ScheduledThreadPoolExecutor` |
| Container idle timeout | disabled (0) | `ClientProperties.SHARED_CONTAINER_IDLE_TIMEOUT` |

### Signing messages

`SignatureService.sign()` computes:
```
prehash = channel + accessKey + apiKeyId + timestamp + portfolioId + products
signature = Base64( HMAC-SHA256(secretKey, prehash) )
```
where `products` = product IDs concatenated directly with no separator between them (e.g. `BTC-USD` + `ETH-USD` → `"BTC-USDETH-USD"`), and `timestamp` = epoch seconds as a string (e.g. `"1700000000"`).

### Testing without a live connection

All tests run without a live WebSocket connection:

- **`PrimeMessageProcessorTest`** — feeds raw JSON strings directly into `processMessage()`. Covers global sequence validation, order routing, error handling, and reconnect triggering.
- **`PrimeWebSocketEndpointTest`** — calls `@OnMessage`, `@OnClose`, `@OnError` directly on an in-process endpoint instance with a mock `Session`.
- **`SignatureServiceTest`** — pure cryptographic unit tests.
- **`OrderQueueServiceTest`** — exercises `enqueue`, `poll`, and capacity-limit behaviour.
- **`OrderConsumerTest`** — uses Mockito + Awaitility against a mock `OrderQueueService`.
- **`PrimeWebSocketApplicationTest`** — `@SpringBootTest` context load with `@MockBean` replacing `PrimeConnectionManager` and `OrderConsumer`.

### Coinbase Prime WebSocket reference

- Endpoint: `wss://ws-feed.prime.coinbase.com`
- Channels subscribed: `heartbeats`, `orders`
- Docs: https://docs.cdp.coinbase.com/prime/websocket-feed/overview
- Channels: https://docs.cdp.coinbase.com/prime/websocket-feed/channels
- Every subscribe/unsubscribe message must be signed.
- `portfolio_id` is required for the `orders` channel.
- Rate limit: 750 connection requests per 10 s per IP.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rcbgr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
