## raftkeeper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build

```bash
# One-command build (ZooKeeper-compatible mode, default)
bash build.sh

# ClickHouse-compatible mode
bash build.sh clickhouse

# Manual CMake build
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=RelWithDebInfo
ninja -C build -j10

# Build with sanitizer
cmake -B build -DSANITIZE=address -DCMAKE_BUILD_TYPE=RelWithDebInfo
ninja -C build
```

**Requirements:** Clang 17+, CMake 3.20+. Submodules must be initialized (`git submodule update --init --recursive`).

**Key CMake options:**
- `COMPATIBLE_MODE_ZOOKEEPER` (ON/OFF) — ZooKeeper or ClickHouse client wire compatibility (default ON)
- `SANITIZE` — `address`, `memory`, `thread`, or `undefined`
- `ENABLE_TESTS` — include C++ unit tests (default ON)

Build outputs go to `build/`. The server binary is `build/programs/raftkeeper`.

## Tests

### C++ Unit Tests (gtest)

Built automatically when `ENABLE_TESTS=ON`. Test sources follow the pattern `src/**/tests/gtest*.cpp`.

```bash
# Run all unit tests
./build/src/rk_unit_tests

# Run a specific test
./build/src/rk_unit_tests --gtest_filter='TestSuiteName.TestName'
```

### Integration Tests (Python/pytest)

Run in Docker. Require `RAFTKEEPER_TESTS_SERVER_BIN_PATH` and `RAFTKEEPER_TESTS_BASE_CONFIG_DIR` env vars or passed as arguments:

```bash
cd tests/integration
./runner --binary /path/to/raftkeeper --base-configs-dir programs/server 'test_multinode_simple'
./runner --binary /path/to/raftkeeper --base-configs-dir programs/server 'test.py::test_name'
```

### Style Checks

```bash
cd tests/ci
python3 code_style_check.py
```

## Architecture

RaftKeeper is a ZooKeeper-compatible distributed consensus service using the Raft protocol. It is derived from ClickHouse and uses eBay's [NuRaft](https://github.com/eBay/NuRaft) library for Raft.

### Layer stack (top-down)

```
ConnectionHandler        — TCP/ZooKeeper wire protocol, per-connection reactor handler
KeeperDispatcher         — Central request router: routes to accumulators/forwarders/processors
RequestAccumulator       — Batches write requests before Raft append (throughput optimization)
RequestForwarder         — Forwards non-leader write requests to the current leader
RequestProcessor         — Applies committed Raft log entries to the state machine
KeeperServer             — Wraps NuRaft launcher + raft_server; cluster membership
NuRaftStateMachine       — Raft state machine: commit/rollback/snapshot
KeeperStore              — In-memory data tree (znodes), session manager, watch manager
```

### Key source directories

| Directory | Purpose |
|-----------|---------|
| `src/Service/` | Core server logic: Raft integration, request pipeline, state machine, data store |
| `src/ZooKeeper/` | ZooKeeper protocol implementation: serialization, types, client library |
| `src/Common/` | Shared utilities ported from ClickHouse: memory tracking, thread pools, IO buffers |
| `src/Network/` | Reactor/epoll-based async network I/O framework |
| `programs/server/` | Server entry point and configuration |
| `programs/converter/` | Tool to translate ZooKeeper snapshot data to RaftKeeper format |
| `contrib/` | Third-party dependencies (NuRaft, boost, jemalloc, boringssl, etc.) |

### Request flow

1. **ConnectionHandler** reads ZooKeeper protocol messages from TCP sockets
2. **KeeperDispatcher** receives requests, routes them by type:
   - Read requests → **RequestProcessor** (executed directly on the local state machine)
   - Write requests → **RequestAccumulator** (batched) → **KeeperServer** (Raft append) → Raft log replication → **NuRaftStateMachine::commit()** → **RequestProcessor::commit()** → applied to **KeeperStore**
   - Writes arriving at a follower → **RequestForwarder** sends them to the leader
3. Responses are returned via session-registered callbacks back through the ConnectionHandler

### Data model

- **KeeperStore** holds an in-memory tree of **KeeperNode** objects (znodes), plus **SessionManager** and **WatchManager**
- Nodes are stored in a two-level `KeeperNodeMap` (sharded unordered_map) to reduce hash table scaling latency
- Persistence is via Raft logs (write-ahead) + periodic snapshots (`NuRaftLogSnapshot`)
- Snapshots and logs are stored on disk in directories configured via `snapshot_dir` and `log_dir`

### Cluster membership

- Configured in `programs/server/config.xml` (and per-node configs in integration tests)
- `KeeperServer` handles `add_srv`/`remove_srv` for dynamic membership changes
- Observer nodes participate in log replication but not leader election or quorum

### Compatibility modes

Controlled by `COMPATIBLE_MODE_ZOOKEEPER` compile definition:
- **ZooKeeper mode** (default): Standard ZK wire protocol
- **ClickHouse mode**: Adapted for ClickHouse's slightly-incompatible ZK client (e.g., multi-read response handling)

## Code style

- C++17, formatted with `.clang-format` at repo root
- `.clang-tidy` configured; run via CMake with `-DUSE_CLANG_TIDY=ON`
- Namespace: `RK`
- Logger pattern: `Poco::Logger::get("ClassName")`

---
> Source: [JDRaftKeeper/RaftKeeper](https://github.com/JDRaftKeeper/RaftKeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
