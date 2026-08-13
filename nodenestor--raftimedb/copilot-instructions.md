## raftimedb

> RaftTimeDB is a **transparent replication proxy** for SpacetimeDB. It wraps SpacetimeDB in a Raft consensus layer so that every write (reducer call) is replicated across multiple nodes. If any node dies, the cluster keeps running. Clients connect to RaftTimeDB exactly like they would connect to SpacetimeDB — same WebSocket protocol, same messages, no code changes.

# RaftTimeDB

## What This Is

RaftTimeDB is a **transparent replication proxy** for SpacetimeDB. It wraps SpacetimeDB in a Raft consensus layer so that every write (reducer call) is replicated across multiple nodes. If any node dies, the cluster keeps running. Clients connect to RaftTimeDB exactly like they would connect to SpacetimeDB — same WebSocket protocol, same messages, no code changes.

**Think of it like this:** SpacetimeDB is your database. RaftTimeDB makes it indestructible by running 3+ copies that stay perfectly in sync.

```
Your app (unchanged)  →  RaftTimeDB (any node)  →  SpacetimeDB
                              ↕ Raft consensus ↕
                         RaftTimeDB node 2  →  SpacetimeDB
                              ↕ Raft consensus ↕
                         RaftTimeDB node 3  →  SpacetimeDB
```

## For Users: How To Set Up and Use

### What You Need

- **3 machines** (or containers, or VMs) — Raft needs a majority, so 3 nodes tolerates 1 failure
- **SpacetimeDB** running on each machine (the same version)
- **Your module** published to ALL SpacetimeDB instances (see "Publishing Modules" below)
- **RaftTimeDB binary** on each machine (build with `cargo build --release`)

### Step 1: Start SpacetimeDB on Each Node

Each node needs its own SpacetimeDB instance:

```bash
# On each machine
spacetimedb-standalone start --listen-addr 0.0.0.0:3000
```

### Step 2: Publish Your Module to ALL Nodes

This is important: **publish the same module to every SpacetimeDB instance.**

```bash
spacetime publish --server http://node1:3000 my_module
spacetime publish --server http://node2:3000 my_module
spacetime publish --server http://node3:3000 my_module
```

RaftTimeDB replicates reducer calls (WebSocket writes), not module deployments (HTTP). Each SpacetimeDB instance needs the module installed independently. Once they all have the same module, Raft ensures they execute the same writes in the same order = identical state.

### Step 3: Start RaftTimeDB on Each Node

```bash
# Node 1
rafttimedb --node-id 1 --stdb-url ws://localhost:3000 \
  --peers "2=node2:4001,3=node3:4001"

# Node 2
rafttimedb --node-id 2 --stdb-url ws://localhost:3000 \
  --peers "1=node1:4001,3=node3:4001"

# Node 3
rafttimedb --node-id 3 --stdb-url ws://localhost:3000 \
  --peers "1=node1:4001,2=node2:4001"
```

Each node runs two servers:
- **:3001** — Client WebSocket (what your app connects to)
- **:4001** — Raft management API (inter-node + CLI)

### Step 4: Bootstrap the Cluster

Run this **once** to initialize Raft:

```bash
rtdb init --nodes 1=node1:4001 2=node2:4001 3=node3:4001
```

This triggers leader election. Within a few seconds, one node becomes the leader.

### Step 5: Verify

```bash
rtdb status --addr http://node1:4001
rtdb status --addr http://node2:4001
rtdb status --addr http://node3:4001
```

You should see one Leader and two Followers, all on the same term.

### Step 6: Connect Your App

Point your SpacetimeDB client at **any** RaftTimeDB node:

```
ws://node1:3001    # or node2:3001, or node3:3001 — any works
```

That's it. Your app works exactly like before, but now it's replicated.

### Docker Compose (Local Dev)

For local development, everything is preconfigured:

```bash
cd deploy/docker
docker compose up -d --build

# Wait for startup, then bootstrap:
curl -X POST http://localhost:4001/cluster/init \
  -H 'Content-Type: application/json' \
  -d '{"1":{"addr":"node-1:4001"},"2":{"addr":"node-2:4001"},"3":{"addr":"node-3:4001"}}'

# Check status:
curl http://localhost:4001/cluster/status
curl http://localhost:4002/cluster/status
curl http://localhost:4003/cluster/status

# Connect your client to ws://localhost:3001 (or :3002, :3003)
```

## Scaling the Cluster

RaftTimeDB supports **dynamic scaling** — add or remove nodes while the cluster is live. No downtime required.

### Adding Nodes

1. Start a new SpacetimeDB instance + RaftTimeDB proxy (with a unique `RTDB_NODE_ID`)
2. Publish the same module to the new SpacetimeDB instance
3. Add it to the Raft cluster:
```bash
rtdb add-node --node-id 4 --addr new-host:4001 --cluster http://existing-node:4001
```

The new node automatically syncs: it receives the Raft log from the leader, catches up, and becomes a voting member.

### Removing Nodes

**Always remove from Raft before stopping the container.** Never remove the current leader — remove followers first.

```bash
rtdb remove-node --node-id 4 --cluster http://existing-node:4001
# Then stop the node
```

### Quorum Rules

Always run an **odd number** of nodes for proper majority quorum:

| Nodes | Quorum needed | Tolerates failures | Use case |
|-------|--------------|-------------------|----------|
| 3     | 2            | 1                 | Dev / small prod |
| 5     | 3            | 2                 | Production |
| 7     | 4            | 3                 | High availability |

### Docker Compose Scaling

For local dev, each new node needs two docker-compose services added:
- `stdb-N` (SpacetimeDB on port `5000+N`)
- `node-N` (RaftTimeDB proxy, WS on `3000+N`, Raft on `4000+N`)

Start only the new containers with `docker compose up -d stdb-N node-N` — existing nodes stay running.

See `/scale` Claude skill for step-by-step automation.

## Can I Do Everything I Can With Normal SpacetimeDB?

**Short answer: yes, for runtime operations.**

| Operation | Works Through RaftTimeDB? | Notes |
|-----------|--------------------------|-------|
| Subscribe to tables | Yes, directly | Reads go straight to local SpacetimeDB |
| One-off queries | Yes, directly | Same — no consensus overhead |
| Call reducers | Yes, through Raft | ~1-5ms added for consensus |
| Call procedures | Yes, through Raft | Same as reducers |
| Publish modules | **No** — do it on each SpacetimeDB directly | HTTP operation, not WebSocket |
| Manage databases | **No** — do it on each SpacetimeDB directly | HTTP operation, not WebSocket |
| SpacetimeDB CLI (`spacetime` commands) | Talk to SpacetimeDB directly (:3000) | These use HTTP, not the proxy |

### How Writes Replicate

1. Client sends `CallReducer` to any node (say node-2, a follower)
2. RaftTimeDB reads tag byte `0x03` → classifies as Write
3. Extracts module name from WebSocket path → routes to correct shard
4. Proposes write through that shard's Raft consensus
5. If this node is a follower for that shard, forwards to the shard's leader internally
6. Leader commits the write, replicates to all nodes
7. **Originating node**: handler forwards the write to its local SpacetimeDB (client gets the response)
8. **Other nodes**: per-shard forwarder task sends the committed write to their local SpacetimeDB
9. All 3 SpacetimeDB instances now have identical state

### How Reads Work

Reads (Subscribe, Unsubscribe, OneOffQuery) go **directly** to the local SpacetimeDB with zero overhead. This is safe because all replicas have identical state thanks to Raft-ordered writes.

### What "Decentralized" Means Here

RaftTimeDB is **distributed and fault-tolerant**, not "decentralized" in the blockchain sense:
- There's always one **leader** at a time (elected automatically by Raft)
- Writes go through the leader for ordering
- If the leader dies, a new one is elected in <5 seconds
- This is the same model as etcd, CockroachDB, and TiKV

## Configuration Reference

| Flag | Env Var | Default | Description |
|------|---------|---------|-------------|
| `--node-id` | `RTDB_NODE_ID` | required | Unique node ID (1, 2, 3, ...) |
| `--listen-addr` | `RTDB_LISTEN_ADDR` | `0.0.0.0:3001` | Client WebSocket address |
| `--raft-addr` | `RTDB_RAFT_ADDR` | `0.0.0.0:4001` | Raft + management API address |
| `--stdb-url` | `RTDB_STDB_URL` | `ws://127.0.0.1:3000` | Local SpacetimeDB WebSocket URL |
| `--peers` | `RTDB_PEERS` | required | Comma-separated: `"2=host:4001,3=host:4001"` |
| `--data-dir` | `RTDB_DATA_DIR` | `./data` | Persistent data directory |
| `--tls-cert` | `RTDB_TLS_CERT` | none | Path to TLS certificate (PEM) |
| `--tls-key` | `RTDB_TLS_KEY` | none | Path to TLS private key (PEM) |
| `--tls-ca-cert` | `RTDB_TLS_CA_CERT` | none | Path to CA cert for peer verification (PEM) |

### CLI Commands

```bash
rtdb init --nodes 1=host:4001 2=host:4001 3=host:4001   # Bootstrap cluster (once)
rtdb status --addr http://host:4001                       # Show node status
rtdb add-node --node-id 4 --addr host:4001 --cluster http://leader:4001  # Add node
rtdb remove-node --node-id 2 --cluster http://leader:4001               # Remove node
rtdb create-shard --shard-id 1 --cluster http://leader:4001             # Create shard
rtdb add-route --module game --shard-id 1 --cluster http://leader:4001  # Route module
rtdb remove-route --module game --cluster http://leader:4001            # Unroute module
rtdb list-shards --addr http://node:4001                                # List shards
rtdb shard-status --shard-id 1 --addr http://node:4001                  # Shard status
```

### Management API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/cluster/status` | GET | Node status (state, leader, term, active shards) |
| `/cluster/init` | POST | Bootstrap cluster with member map (shard 0 only) |
| `/cluster/add-node` | POST | Add node (learner → voter, all shards) |
| `/cluster/remove-node` | POST | Remove node from cluster (all shards) |
| `/cluster/write` | POST | Internal: leader receives forwarded writes (shard 0) |
| `/cluster/{shard_id}/write` | POST | Internal: shard-specific write forwarding |
| `/cluster/leader` | GET | Current leader ID and address |
| `/cluster/health` | GET | Health check (200 if leader known, 503 otherwise) |
| `/cluster/shards` | GET | List all shards and routes |
| `/cluster/shards/create` | POST | Create a new shard (via Raft consensus) |
| `/cluster/shards/route` | POST | Add module → shard route (via Raft) |
| `/cluster/shards/route` | DELETE | Remove module route (via Raft) |
| `/cluster/shards/{id}/status` | GET | Shard-specific Raft status |
| `/metrics` | GET | Prometheus metrics (text format) |
| `/raft/{shard_id}/append` | POST | Internal: shard-specific Raft AppendEntries |
| `/raft/{shard_id}/vote` | POST | Internal: shard-specific Raft RequestVote |
| `/raft/{shard_id}/snapshot` | POST | Internal: shard-specific Raft InstallSnapshot |
| `/raft/append` | POST | Legacy alias: Raft AppendEntries (shard 0) |
| `/raft/vote` | POST | Legacy alias: Raft RequestVote (shard 0) |
| `/raft/snapshot` | POST | Legacy alias: Raft InstallSnapshot (shard 0) |

## Project Structure

```
crates/
  proxy/src/
    main.rs              ← Entry point: starts HTTP API + WebSocket proxy + TLS
    api.rs               ← axum HTTP server (Raft RPCs + cluster + shard management + metrics)
    config.rs            ← NodeConfig struct (including TLS config)
    metrics.rs           ← Prometheus metric definitions and encoder
    raft/
      mod.rs             ← RaftPool: multi-shard Raft pool, propose_write, leader forwarding
      types.rs           ← ReducerCallRequest/Response, TypeConfig
      network.rs         ← HTTP transport (reqwest → POST /raft/{shard_id}/*)
      state_machine.rs   ← Apply committed entries, snapshots, shard config (0xFF), forwarder
      log_store.rs       ← In-memory Raft log (kept for reference/testing)
      persistent_log_store.rs ← Per-shard persistent Raft log using redb (production)
    websocket/
      mod.rs             ← Proxy: accepts client connections, holds Arc<RaftPool>
      handler.rs         ← Message classification, module extraction, shard-aware routing
      upstream.rs        ← Placeholder for connection management
    router/
      mod.rs             ← ShardRouter, ShardConfigCommand, extract_module_name, encode/decode
  cli/src/
    main.rs              ← rtdb CLI: status, init, add/remove-node, shard management
deploy/
  docker/
    Dockerfile           ← Multi-stage Rust build
    docker-compose.yml   ← 3-node local cluster
  helm/                  ← Kubernetes Helm chart (placeholder)
  k3s/                   ← K3s manifests (placeholder)
tests/
  e2e/
    docker_cluster_test.sh  ← Full cluster E2E test
```

## Key Architecture Decisions

- **HTTP transport** for Raft RPCs (reqwest + axum) — simple and debuggable
- **Explicit bootstrap** via `rtdb init` — deterministic, like etcd
- **Server-side leader forwarding** — followers forward writes to leader internally; clients never see topology
- **One-byte classification** — reads tag byte 0 of each WebSocket message to route reads vs writes
- **Opaque blob replication** — entire WebSocket message replicated, no BSATN parsing needed
- **origin_node_id dedup** — each Raft entry records which node proposed it; the forwarder skips entries from the local node (handler already forwarded them)
- **Metadata-only snapshots** — SpacetimeDB is the source of truth; snapshots store last_applied + membership + shard config
- **Persistent log store** — redb (pure Rust) stores Raft log + vote on disk; survives restarts. Per-shard: `{data_dir}/shard-{id}/raft-log.redb`
- **Multi-Raft (multi-shard)** — one Raft group per SpacetimeDB module. Different modules get different leaders, enabling parallel write throughput. Shard 0 is the default; all unrouted modules go there
- **Shard config via Raft** — shard mappings (module → shard) are consensus-replicated through shard 0's Raft log (0xFF prefix tag). No external config store needed
- **All nodes participate in all shards** — simpler model. Per-shard membership subsets is a future optimization
- **Path-based Raft RPC routing** — `/raft/{shard_id}/append` with legacy `/raft/append` aliasing to shard 0

## Development

```bash
cargo build                    # Build both crates
cargo test                     # Run all 65 tests
cargo build --release          # Release build
cargo install --path crates/cli   # Install rtdb CLI
RUST_LOG=rafttimedb=debug cargo run --bin rafttimedb -- --node-id 1 ...  # Debug logging
```

### Code Conventions

- Rust 2024 edition
- openraft 0.9 with `serde` + `storage-v2` features
- tokio async runtime throughout
- `anyhow::Result` for application errors, `thiserror` for library errors
- tracing for structured logging (info/debug/warn/error)
- Tests: unit tests in `#[cfg(test)] mod tests` inside each file, integration tests in `tests/`

### Running the E2E Test

```bash
# Requires Docker
./tests/e2e/docker_cluster_test.sh
```

Tests: containers running, API reachable, cluster bootstrap, leader election, write forwarding, fault tolerance (kill node, re-election), node rejoin.

## Production Deployment

For deploying across multiple servers, use the `/production-deploy` Claude skill. Key points:

### What each server needs
1. SpacetimeDB instance (port 3000, localhost only)
2. RaftTimeDB proxy binary (WS: 3001, Raft API: 4001)
3. The same module published to every SpacetimeDB instance

### Load Balancer (EXTERNAL — not part of RaftTimeDB)

Put a load balancer in front of the WebSocket ports (3001) so clients connect to one URL. RaftTimeDB handles routing internally — any node can accept reads and writes.

**nginx example:**
```nginx
upstream rafttimedb {
    server server-a:3001;
    server server-b:3001;
    server server-c:3001;
}
server {
    listen 443 ssl;
    location / {
        proxy_pass http://rafttimedb;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

**Cloud:** Use a Network Load Balancer (AWS NLB, GCP TCP LB, Azure LB) pointed at port 3001. Health check: `GET http://{node}:4001/cluster/status` returns 200.

### Persistence

Raft log is stored on disk at `{RTDB_DATA_DIR}/shard-{id}/raft-log.redb` using redb (pure Rust, ACID). Each shard gets its own database file. Nodes survive restarts and rejoin the cluster automatically. Legacy `raft-log.redb` at the data dir root is auto-migrated to `shard-0/` on first startup.

### TLS Encryption

Enable TLS for inter-node and client communication:

```bash
rafttimedb --node-id 1 \
  --tls-cert /path/to/cert.pem \
  --tls-key /path/to/key.pem \
  --tls-ca-cert /path/to/ca.pem \  # optional, for verifying peer nodes
  ...
```

When TLS is enabled:
- Raft inter-node RPCs use HTTPS
- Management API serves over HTTPS
- Leader forwarding uses HTTPS

### Monitoring

- `rtdb status --addr http://{node}:4001` — cluster health
- `GET /metrics` — Prometheus metrics (text format)
- `GET /cluster/health` — returns 200 if leader known, 503 otherwise
- `GET /cluster/leader` — current leader ID and address
- `journalctl -u rafttimedb -f` — live logs (with systemd)
- Watch for: frequent leader changes, high term numbers, nodes stuck as Learner

Prometheus metrics exposed:
- `rafttimedb_raft_term` — current Raft term
- `rafttimedb_raft_is_leader` — 1 if this node is leader, 0 otherwise
- `rafttimedb_connections_active` — active WebSocket client connections
- `rafttimedb_writes_total` — total writes proposed through Raft
- `rafttimedb_reads_total` — total reads forwarded directly
- `rafttimedb_write_latency_seconds` — write latency histogram
- `rafttimedb_entries_applied_total` — Raft entries applied to state machine

### Client Reconnection

When a node shuts down gracefully, it sends a WebSocket close frame with a leader hint:
```
Close code: 1001 (Going Away)
Reason: "leader=<id>:<addr>"
```

Clients can parse this to reconnect directly to the current leader. The `/cluster/leader` endpoint also provides the leader address for HTTP-based discovery.

### Graceful Shutdown

The proxy handles Ctrl+C (all platforms) and SIGTERM (Unix). On shutdown:
1. All tasks receive a shutdown signal via watch channel
2. Active WebSocket connections receive close frames with leader hints
3. HTTP server stops accepting new connections
4. 5-second drain period allows in-flight requests to complete

## Multi-Shard (Multi-Raft)

RaftTimeDB supports multiple Raft groups — one per SpacetimeDB module. Different modules get different leaders, so writes for different modules process in parallel.

**Backwards compatible**: existing single-shard clusters keep working with zero config changes. Shard 0 is the default — all unrouted modules go there.

### Creating and Using Shards

```bash
# Create a new shard
rtdb create-shard --shard-id 1 --cluster http://leader:4001

# Route a module to the new shard
rtdb add-route --module game --shard-id 1 --cluster http://leader:4001

# Now all "game" module writes go through shard 1's Raft group
# Other modules still use shard 0 (default)

# Check shard status
rtdb list-shards --addr http://node:4001
rtdb shard-status --shard-id 1 --addr http://node:4001

# Remove a route (module goes back to shard 0)
rtdb remove-route --module game --cluster http://leader:4001
```

### How It Works

1. Shard config commands are replicated via shard 0's Raft log (consensus-safe)
2. All nodes see the same config changes in the same order
3. New shards inherit membership from shard 0 (all nodes participate in all shards)
4. Each shard has its own Raft leader, log store, forwarder, and SpacetimeDB connection
5. The WebSocket handler extracts the module name from the path and routes to the correct shard
6. Shard config is preserved in snapshots — new nodes receive the full shard topology

### Shard Config Encoding

Shard config commands use tag byte `0xFF` (SpacetimeDB tags are 0-4, so no conflict). Format: `[0xFF][JSON payload]`. This is replicated as a normal Raft entry through shard 0.

## Known Limitations / Next Steps

- **No module deployment replication** — you must `spacetime publish` to each instance separately. A future version could proxy the SpacetimeDB HTTP API too.
- **All nodes in all shards** — every node participates in every Raft group. A future optimization could allow per-shard membership subsets for very large clusters.

---
> Source: [NodeNestor/RaftimeDB](https://github.com/NodeNestor/RaftimeDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
