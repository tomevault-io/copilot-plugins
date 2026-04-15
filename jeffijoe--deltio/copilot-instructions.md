## deltio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Deltio is a high-performance Google Cloud Pub/Sub emulator written in Rust, designed as a lightweight alternative to the official emulator for local development and CI. Everything is in-memory (no persistence). It implements the official Google Pub/Sub gRPC API using proto definitions from googleapis/googleapis.

## Build & Development Commands

Requires Rust (stable) and `protoc` (Protocol Buffers compiler). Proto compilation happens automatically via `build.rs`.

```bash
cargo build                          # Debug build
cargo build --release                # Release build (LTO, single codegen unit)
cargo test                           # All tests (unit + integration)
cargo test --test subscriber_test    # Single integration test file
cargo test test_publish              # Tests matching a name
cargo clippy                         # Lint
cargo fmt --all -- --check           # Check formatting
cargo fmt                            # Auto-format
```

**E2E tests** (require a running Deltio instance):
```bash
./target/release/deltio --bind 0.0.0.0:8085 --log trace &
cd e2e && cargo run
```

**Note:** Integration tests use Unix sockets (via `TestHost` in `tests/test_helpers/mod.rs`), so they don't run on Windows.

## Architecture

### Actor Pattern

The core concurrency model uses actors communicating via tokio mpsc channels. Each actor runs as a spawned tokio task processing request enums, with oneshot channels for request-response patterns.

- **TopicActor** (`src/topics/topic_actor.rs`): Manages topic state — published messages, attached subscriptions. Created by `Topic::new()`.
- **SubscriptionActor** (`src/subscriptions/subscription_actor.rs`): Manages subscription state — pulled messages, ACKs, deadline modifications. Created by `Subscription::new()`.

### Manager Layer

- **TopicManager** (`src/topics/topic_manager.rs`): Coordinates topic creation/access using `Arc<RwLock<State>>`. Brief write locks during creation only.
- **SubscriptionManager** (`src/subscriptions/subscription_manager.rs`): Same pattern for subscriptions. Uses delegate callbacks for actor-to-manager notifications (e.g., deletion).

### gRPC Services

- **PublisherService** (`src/api/publisher.rs`): Implements the `Publisher` gRPC trait (create/get/list/delete topics, publish messages).
- **SubscriberService** (`src/api/subscriber.rs`): Implements the `Subscriber` gRPC trait (subscriptions, pull, streaming pull, ACK, push config).

### Push Subscriptions

`PushLoop` (`src/push/push_loop.rs`) runs a background async loop that polls registered push subscriptions and delivers messages via HTTP. Registry is shared between the subscription manager and push loop.

### Key Design Decisions

- **MiMalloc** global allocator for performance (disabled on 32-bit x86 Linux)
- **Arc\<TopicMessage\>** for zero-copy message sharing across subscriptions
- **Strongly-typed resource names** (`TopicName`, `SubscriptionName`) with format `projects/{project}/topics|subscriptions/{name}`
- **thiserror** for domain-specific error types mapped to gRPC Status codes in the API layer

## CI

CI runs on push/PR across Windows, macOS, Ubuntu. Lint/format checks run on Ubuntu only. Tests skip Windows (Unix socket requirement). E2E runs against a release binary on all platforms.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeffijoe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
