## monk-os-kernel

> **Detect which repo you're in from the directory name:**

# Monk OS - Agent Instructions & Technical Architecture

## Clone Context

**Detect which repo you're in from the directory name:**

| Directory | Role | Rules |
|-----------|------|-------|
| `os/` | **Main** | Stable. No speculative changes. Run tests before committing. |
| `os-dev*/` | **Development** | Active work. Ask user for current goal if unclear. |

**In main**: Be conservative. Only well-tested, complete changes.

**In dev clones**: Move fast, experiment freely. The user will direct you.

---

## Prompt Library (`prompts/`)

**Select and apply prompts based on what you're working on:**

| File | Use When | Summary |
|------|----------|---------|
| `planning-mode.md` | Designing new features | Collaborative planning via `docs/`. Plans persist across sessions, iterate with user. |
| `kernel-dev.md` | Modifying `src/` (kernel, HAL, VFS, EMS) | Linux kernel + TypeScript staff engineer code review. Focus on race conditions, invariants, concurrency, testability. |
| `userspace-dev.md` | Modifying `rom/` (bin commands, lib utilities) | GNU coreutils + TypeScript code review. Focus on POSIX compatibility, streams, error handling, argument parsing. |
| `parallel-agents.md` | Bulk changes across many files | How to split work across 2-5 parallel agents. ~4x faster than sequential. |
| `tester.md` | Fixing `spec/` or `perf/` after refactors | Parallel agent strategy for fixing typecheck and test failures. |
| `performance.md` | Writing `perf/` tests | Throughput, stress, and backend comparison tests. Run via `bun run perf`. |

**Before major work**: Read the relevant prompt and apply its standards.

**For large changes (5+ files)**: Use `parallel-agents.md` to split work. Parallel agents have been very successful for bulk refactors and conversions.

---

> **Quick Context**: Monk OS reframes API architecture as an operating system where **Bun is the hardware**. The single-executable deployment (`bun build --compile`) isn't packaging an app—it's burning firmware.

## 1. Core Philosophy & Architecture

### System Design Principles
- **Everything is a file**: Uniform namespace following Plan 9's paradigm
- **Files are queryable**: BeOS-style database-centric filesystem (files have UUIDs, indexed)
- **Process isolation**: Each process is a Bun Worker with isolated memory
- **Message-driven**: All internal communication uses structured `Message` and `Response` objects
- **Streams-first**: Default API is `AsyncIterable<Response>`, not arrays

### Layered Architecture
```
┌─────────────────────────────────────────────────────────────┐
│  External Applications (os-sdk via Unix socket)             │
├─────────────────────────────────────────────────────────────┤
│  Gateway (src/gateway/) - MessagePack wire protocol         │
├─────────────────────────────────────────────────────────────┤
│  OS Public API (src/os/)                                    │
│  ├── OS class (boot, exec, shutdown, syscall wrappers)      │
│  ├── Domain wrappers (ems, vfs, process)                    │
│  ├── Convenience helpers (read, text, spawn, mount, copy)   │
│  └── Service management (start, stop, restart, list)        │
├─────────────────────────────────────────────────────────────┤
│  Dispatch Layer (src/dispatch/)                             │
│  ├── Dispatcher (syscall routing + sigcall routing)         │
│  ├── StreamController (backpressure, ping/cancel protocol)  │
│  ├── Sigcall registry (userspace handler registration)      │
│  ├── Domain handlers (vfs, ems, hal, process, handle, pool) │
│  └── Worker message handling (onWorkerMessage callback)     │
├─────────────────────────────────────────────────────────────┤
│  Kernel Layer (src/kernel/)                                 │
│  ├── Process Manager (Worker lifecycle, signals)            │
│  ├── Handle System (file, socket, pipe, port, channel)      │
│  ├── Worker Pools (PoolManager, auto-scaling)               │
│  ├── Service Activation (boot, tcp, udp, pubsub, watch)     │
│  └── Extracted functions (kernel/kernel/ subdirectory)      │
├─────────────────────────────────────────────────────────────┤
│  VFS - Virtual File System (src/vfs/)                       │
│  ├── Path resolution, mount table, model coordination       │
│  └── Models: File, Folder, Device, Proc, Link               │
├─────────────────────────────────────────────────────────────┤
│  EMS - Entity Model System (src/ems/)                       │
│  ├── Database abstraction (SQLite, PostgreSQL)              │
│  ├── Observer pipeline (8 rings for mutation processing)    │
│  ├── EntityOps, ModelCache, EntityCache                     │
│  └── Streaming queries with backpressure                    │
├─────────────────────────────────────────────────────────────┤
│  HAL - Hardware Abstraction (src/hal/)                      │
│  ├── BlockDevice, StorageEngine, NetworkDevice, FileDevice  │
│  ├── Timer, Clock, Entropy, Crypto, Console                 │
│  ├── DNS, Host, IPC, Channel, Compression                   │
│  └── BunHAL implementation                                  │
├─────────────────────────────────────────────────────────────┤
│  Bun Runtime (Host OS, Workers, primitives)                 │
└─────────────────────────────────────────────────────────────┘
```

### Naming Conventions

Three delimiters with distinct semantic roles:

| Delimiter | Domain | Example | Semantics |
|-----------|--------|---------|-----------|
| `:` | Syscalls | `auth:login`, `vfs:read` | Verb - "do this action in this subsystem" |
| `.` | Models | `auth.user`, `llm.provider` | Noun - "this entity type owned by this subsystem" |
| `_` | SQL tables | `auth_user`, `llm_provider` | Physical - derived from model name (`s/\./_/`) |

**Core/foundational types use bare names**: `file`, `folder`, `device`, `models`, `fields` - these are primitives, not "owned" by a subsystem.

**Subsystem-specific types use dot notation**: `auth.user`, `auth.session`, `llm.provider`, `llm.model` - clear ownership, enables wildcard queries (`auth.*`).

**EMS ops use model names** (dot notation):
```typescript
await ems.ops.selectOne('auth.user', { username });
await ems.ops.createOne('auth.session', { user_id, expires });
```

**SQL tables use underscores** (mechanical transform):
```sql
CREATE TABLE auth_user (...);
CREATE TABLE auth_session (...);
```

### Message Flow (Syscall Dispatch)
```
Worker ──postMessage──▶ kernel.onWorkerMessage ──▶ dispatcher.onWorkerMessage()
                                                          │
                                                          ▼
                                                   dispatcher.execute()
                                                          │
                                                          ▼
                                                   dispatcher.dispatch()
                                                          │
                                                          ▼
                                                   syscall handlers
                                                   (vfs, ems, hal, etc.)
```

---

## 2. Key Subsystems

### A. Kernel (`src/kernel/`)

**Process Model**:
- Each process is a Bun Worker with UUID identity (not PID)
- Process states: starting → running → stopped → zombie
- File descriptors: integers (0-255) mapped to handle UUIDs per-process
- Standard fds: 0=recv (messages in), 1=send (messages out), 2=warn (diagnostics)
- Signals: SIGTERM (graceful), SIGKILL (immediate)

**Key Directory Structure**:
- `kernel.ts` - Boot sequence, resource lifecycle, exposes `onWorkerMessage` callback
- `process-table.ts` - Process registry
- `types.ts` - Process, SpawnOpts, ExitStatus, ProcessPortMessage, etc.
- `errors.ts` - Re-exports HAL errors + ProcessExited
- `kernel/` - Extracted kernel functions (modular organization)
- `handle/` - Unified handle architecture:
  - `types.ts` - Handle interface and HandleType
  - `file.ts` - FileHandleAdapter (VFS files)
  - `socket.ts` - SocketHandleAdapter (TCP connections)
  - `port.ts` - PortHandleAdapter (listeners, watchers, pubsub)
  - `channel.ts` - ChannelHandleAdapter (protocol-aware I/O)
  - `process-io.ts` - ProcessIOHandle (service I/O routing)
  - `console.ts` - ConsoleHandleAdapter (message ↔ byte boundary)
- `resource/` - Port and pipe implementations:
  - `types.ts` - Port, PortMessage interfaces
  - `listener-port.ts` - TCP listener
  - `watch-port.ts` - VFS file watcher
  - `udp-port.ts` - UDP socket
  - `pubsub-port.ts` - Topic-based messaging
  - `message-pipe.ts` - MessagePipe (message-based inter-process pipe)
- `loader/` - VFS module loader:
  - `vfs-loader.ts` - Transpilation, bundling
  - `imports.ts` - Import resolution
  - `cache.ts` - Module caching
- `services.ts` - Service definitions (activation types, I/O config)
- `pool.ts` - Worker pool management (PoolManager, WorkerPool)
- `pool-worker.ts` - Individual worker management
- `boot.ts` - ROM → VFS copy at boot
- `mounts.ts` - Mount configuration loader
- `poll.ts` - Event polling
- `validate.ts` - Input validation utilities

**Note**: Syscall handlers have been moved to `src/dispatch/`. The kernel now focuses solely on process management, handle management, and resource lifecycle. See "Dispatch Layer" section below.

**Execution Model**:
- Syscalls return `AsyncIterable<Response>` (generators)
- Each `Response` is: `{ op: 'ok'|'error'|'item'|'data'|'event'|'progress'|'done'|'redirect', data?: object }`
- Terminal ops (`ok`, `error`, `done`) signal stream completion
- Non-terminal ops (`item`, `event`, `progress`, `data`) may yield multiple times
- Backpressure via `stream_ping` every 100ms (managed by StreamController)

### A.1 Dispatch Layer (`src/dispatch/`)

**Purpose**: Separates syscall/sigcall routing and implementation from the kernel. The kernel focuses on process/handle management while the dispatcher orchestrates operations across kernel, VFS, EMS, and HAL.

**Architecture**:
- **Dispatcher sits outside kernel**: `Dispatcher` is created by OS layer, receives `(kernel, vfs, ems, hal)` as constructor dependencies
- **Kernel provides callback**: The kernel exposes `onWorkerMessage` callback set by OS after creating the dispatcher
- **Direct dependencies**: Each syscall function receives exactly what it needs as parameters—no context objects
- **Sigcall routing**: Unknown syscall names are checked against the sigcall registry and routed to userspace handlers

**Directory Structure**:
```
src/dispatch/
├── index.ts           # Exports Dispatcher
├── dispatcher.ts      # Switch-based routing, sigcall routing
├── types.ts           # Shared types
├── syscall/           # Kernel-side syscall handlers
│   ├── vfs.ts         # file:* syscalls → VFS + Kernel
│   ├── ems.ts         # ems:* syscalls → EMS
│   ├── hal.ts         # net:*, port:*, channel:* syscalls → HAL + Kernel
│   ├── process.ts     # proc:*, activation:* syscalls → Kernel
│   ├── handle.ts      # handle:*, ipc:* syscalls → Kernel
│   ├── pool.ts        # pool:*, worker:* syscalls → Kernel
│   └── sigcall.ts     # sigcall:register/unregister/list syscalls
├── sigcall/           # Sigcall infrastructure
│   ├── index.ts       # Exports registry
│   └── registry.ts    # Handler registration tracking
└── stream/            # Backpressure protocol
    ├── index.ts
    ├── controller.ts  # StreamController (ping/cancel, high/low water)
    ├── syscall-controller.ts  # Kernel→userspace streams
    ├── sigcall-controller.ts  # Userspace→kernel streams
    ├── constants.ts   # STREAM_HIGH_WATER, STREAM_STALL_TIMEOUT, etc.
    └── types.ts
```

**Syscall Function Pattern**:
```typescript
export async function* fileOpen(
    proc: Process,      // Always first (except pool:stats)
    kernel: Kernel,     // For handle/process operations
    vfs: VFS,           // For filesystem operations
    path: unknown,      // Syscall-specific args (validated internally)
    flags?: unknown,
): AsyncIterable<Response> {
    // Validate arguments
    if (typeof path !== 'string') {
        yield respond.error('EINVAL', 'path must be a string');
        return;
    }
    // Execute operation
    const handle = await vfs.open(path, proc.user, flags);
    const fd = kernel.assignHandle(proc, handle);
    yield respond.ok(fd);
}
```

**Key Design Decisions**:
1. **Yield errors, don't throw**: All syscalls return `AsyncIterable<Response>`, validation errors are yielded
2. **Switch-based routing**: Dispatcher uses switch statement for O(1) syscall lookup
3. **StreamController wrapping**: Each syscall execution is wrapped for backpressure management
4. **Dependency argument order**: `proc`, `kernel`, `vfs`, `ems`, `hal`, then syscall-specific args

### B. Hardware Abstraction Layer - HAL (`src/hal/`)

**14 Device Interfaces**:

| Device | File | Purpose |
|--------|------|---------|
| **BlockDevice** | `block.ts` | Raw byte storage (files, S3, databases) |
| **StorageEngine** | `storage.ts` | Key-value with transactions (SQLite, PostgreSQL, memory) |
| **NetworkDevice** | `network.ts` | TCP/UDP/HTTP (listen, connect, socket operations) |
| **TimerDevice** | `timer.ts` | Scheduling (setTimeout, setInterval) |
| **ClockDevice** | `clock.ts` | Time sources (Date.now, Bun.nanoseconds) |
| **EntropyDevice** | `entropy.ts` | Random bytes, UUIDs |
| **CryptoDevice** | `crypto.ts` | Hash, encryption, key derivation |
| **ConsoleDevice** | `console.ts` | stdin/stdout/stderr |
| **DNSDevice** | `dns.ts` | Hostname resolution |
| **HostDevice** | `host.ts` | Escape to host OS (Bun.spawn) |
| **IPCDevice** | `ipc.ts` | Shared memory, mutex, semaphore, condvar |
| **ChannelDevice** | `channel.ts` | Protocol-aware messaging (HTTP, WebSocket, PostgreSQL, SQLite) |
| **CompressionDevice** | `compression.ts` | Gzip/deflate compression |
| **FileDevice** | `file.ts` | Host filesystem access (kernel-only, for ROM bootstrap) |

**Key Files**:
- `index.ts` - HAL interface, BunHAL class, error exports
- `errors.ts` - POSIX-style error classes (ENOENT, EBADF, EPERM, etc.)
- Each device in its own file: `block.ts`, `storage.ts`, `network.ts`, etc.

**Key Principles**:
- All operations are `async`/`await`
- Naming: POSIX for I/O (`read`, `write`, `stat`, `sync`), domain-specific elsewhere
- `BunHAL` is the main implementation, `init()` and `shutdown()` for lifecycle

### C. Virtual File System - VFS (`src/vfs/`)

**Philosophy**: Everything is a file, everything has a UUID, everything can be queried as a database row.

**Core Models** (5 total):
- **FileModel** (`models/file.ts`) - Regular files backed by StorageEngine
- **FolderModel** (`models/folder.ts`) - Directories (children computed via query)
- **DeviceModel** (`models/device.ts`) - Kernel devices (/dev/console, /dev/random, /dev/null, /dev/zero, /dev/clock) and streaming compression (/dev/gzip, /dev/gunzip, /dev/deflate, /dev/inflate)
- **ProcModel** (`models/proc.ts`) - Process info (/proc/{uuid}/stat, /proc/{uuid}/env, etc.)
- **LinkModel** (`models/link.ts`) - Symbolic links (currently disabled, throws EPERM)

**Key Files**:
- `index.ts` - Exports VFS class and types
- `vfs.ts` - Path resolution, mount table, model coordination, host mounts
- `model.ts` - PosixModel base class, ModelStat, ModelContext, FieldDef
- `handle.ts` - FileHandle interface, OpenFlags, SeekWhence, OpenOptions
- `models/*.ts` - Model implementations

**Design Features**:
- Identity: UUID (generated via hal.entropy.uuid())
- Path: computed from (parent_id, name) - renames are atomic
- Permissions: Grant-based ACL system (not UNIX rwx)
- Host mounts: `mountHost(osPath, hostPath, opts)` for host filesystem access
- Transactions: atomic create/update/delete

**Model Interface** (extends PosixModel):
```typescript
abstract class PosixModel {
    abstract readonly name: string;  // 'file', 'folder', 'device', 'proc', 'link'
    abstract fields(): FieldDef[];
    abstract open(ctx, id, flags, opts?): Promise<FileHandle>;
    abstract stat(ctx, id): Promise<ModelStat>;
    abstract setstat(ctx, id, fields): Promise<void>;
    abstract create(ctx, parent, name, fields?): Promise<string>;
    abstract unlink(ctx, id): Promise<void>;
    abstract list(ctx, id): AsyncIterable<string>;
    watch?(ctx, id, pattern?): AsyncIterable<WatchEvent>;  // optional
}
```

### D. Entity Model System - EMS (`src/ems/`)

**Purpose**: Database abstraction layer with observer pipeline for mutations. Provides streaming queries and entity lifecycle management.

**Architecture**:
```
┌─────────────────────────────────────────────────────────────┐
│  EntityOps (streaming entity operations)                     │
├─────────────────────────────────────────────────────────────┤
│  Observer Pipeline (8 rings for mutation processing)        │
├─────────────────────────────────────────────────────────────┤
│  DatabaseOps (generic SQL streaming)                        │
├─────────────────────────────────────────────────────────────┤
│  DatabaseConnection (HAL channel wrapper)                   │
├─────────────────────────────────────────────────────────────┤
│  HAL ChannelDevice (SQLite, PostgreSQL)                     │
└─────────────────────────────────────────────────────────────┘
```

**Observer Pipeline** (8 rings, lower = earlier):

| Ring | Observers | Purpose |
|------|-----------|---------|
| 0 | UpdateMerger | Deduplication of updates |
| 1 | Frozen, Immutable, Constraints | Validation |
| 4 | TransformProcessor | Field transformation |
| 5 | SqlCreate, SqlUpdate, SqlDelete, PathnameSync | Database persistence |
| 6 | DdlCreateModel, DdlCreateField | Schema operations |
| 7 | Tracked | Change tracking |
| 8 | Cache, EntityCacheSync | Cache invalidation |

**Key Components**:
- **EntityOps** - Streaming entity CRUD with observer pipeline integration
- **ModelCache** - Metadata caching for entity types
- **EntityCache** - Entity instance caching by type
- **DatabaseConnection** - Wraps HAL channel for SQLite/PostgreSQL
- **Observer** - Base class for mutation observers

**Supported Entities**:
- `entity` - Generic catch-all model
- `file` - Documents with binary data (EMS-backed)
- `folder` - Container entities (EMS-backed)
- `device`, `proc`, `link` - Virtual entities (HAL-backed, by design)

**Storage Keys**:
```
entity:{uuid}     → JSON metadata
access:{uuid}     → ACL data
child:{parent}:{name} → child UUID (O(1) child lookup)
data:{uuid}       → file content blobs
```

**Status**: Complete. File/Folder use EMS (SQL-backed). Device/Proc/Link use HAL storage (virtual entities by design).

---

### E. OS Public API (`src/os/`)

**Purpose**: Main entry point for external applications consuming Monk OS.

**Key Files**:
- `index.ts` - Exports OS class, types
- `os.ts` - OS class with lifecycle, syscall wrappers, convenience helpers
- `types.ts` - OSConfig, BootOpts, ExecOpts, OSEvents

**State Machine**:
```
[created] ──boot()──▶ [booting] ──success──▶ [booted]
    │                     │                     │
    │                     │ failure             │ shutdown()
    │                     ▼                     ▼
    │                 [failed]              [shutdown]
    │                                           │
    └───────────────────────────────────────────┘
               (can boot again after shutdown)
```

**OS Class Interface**:
```typescript
class OS {
    constructor(config?: OSConfig);

    // Lifecycle
    async boot(opts?: BootOpts): Promise<void>;
    async exec(opts?: ExecOpts): Promise<number>;  // Standalone mode
    async shutdown(): Promise<void>;
    isBooted(): boolean;

    // Path aliases
    alias(name: string, path: string): this;
    resolvePath(path: string): string;

    // Syscall API (executes as init process)
    async syscall<T>(name: string, ...args: unknown[]): Promise<T>;
    syscallStream(name: string, ...args: unknown[]): AsyncIterable<Response>;

    // Domain wrappers
    async ems<T>(method: string, ...args: unknown[]): Promise<T>;
    async vfs<T>(method: string, ...args: unknown[]): Promise<T>;
    async process<T>(method: string, ...args: unknown[]): Promise<T>;
    file<T>(method: string, ...args: unknown[]): Promise<T>;  // Alias for vfs
    entity<T>(method: string, ...args: unknown[]): Promise<T>;  // Alias for ems

    // Convenience helpers
    async spawn(cmd: string, opts?): Promise<number>;
    async kill(pid: number, signal?: number): Promise<void>;
    async mount(type: string, source: string, target: string, opts?): Promise<void>;
    async unmount(target: string): Promise<void>;
    async copy(hostSource: string, vfsTarget: string): Promise<void>;
    async read(filePath: string): Promise<Uint8Array>;
    async text(filePath: string, encoding?: string): Promise<string>;

    // Service management
    async service(action: string, nameOrPid?: string | number): Promise<unknown>;

    // Internal accessor (for EntityAPI only)
    getEntityOps(): EntityOps;
}

// For tests, use TestOS which provides internal* accessors:
// TestOS.internalHal, internalVfs, internalKernel, internalEms, internalAuth
// See spec/README.md for testing patterns
```

**Boot Sequence**:
1. HAL (hardware abstraction)
2. EMS (entity management)
3. VFS (virtual filesystem)
4. Standard directories (/app, /bin, /etc, /home, /svc, /tmp, /usr, /var, /vol)
5. ROM copy (userspace code from host)
6. Kernel init (creates kernel process as PID 1, mounts /proc, loads services)
7. Dispatcher setup
8. Kernel boot (activates services, starts tick)

**Configuration**:
```typescript
interface OSConfig {
    storage?: {
        type: 'memory' | 'sqlite' | 'postgres';
        path?: string;  // For sqlite
        url?: string;   // For postgres
    };
    env?: Record<string, string>;      // Environment for all processes
    aliases?: Record<string, string>;  // '@app' -> '/vol/app'
    debug?: boolean;                   // Kernel debug logging
    romPath?: string;                  // Host path to ROM (default: './rom')
}
```

**Usage Examples**:
```typescript
// Library mode
const os = new OS({ storage: { type: 'memory' } });
await os.boot();
const users = await os.ems('select', 'User', { where: { active: true } });
await os.shutdown();

// Standalone mode
const os = new OS({ storage: { type: 'sqlite', path: '.data/monk.db' } });
const exitCode = await os.exec();  // Blocks until SIGINT/SIGTERM
process.exit(exitCode);
```

### F. Network Layer (Kernel, not VFS)

**Two Primitive Abstractions**:

1. **Handle** - Unified I/O interface
   - Methods: `exec(msg)` → `AsyncIterable<Response>`, `close()`
   - Types: file, socket, pipe, port, channel
   - Used for: All I/O operations

2. **Port** - Message endpoints (subset of Handle)
   - Methods: `recv()`, `send(to, data)`, `close()`
   - Returns: structured messages with from/meta
   - Used for: UDP datagrams, TCP listeners, file watch, pub/sub topics

**Port Types**:
- `tcp:listen` - Accept TCP connections (yields socket handle)
- `udp` - UDP socket (send/receive datagrams)
- `watch` - File system event stream (matches patterns)
- `pubsub` - Topic-based messaging (subscribe/publish)

**Handle Types**:
- `file` - VFS files, folders, devices (FileHandleAdapter)
- `socket` - TCP connections (SocketHandleAdapter)
- `pipe` - Message-based pipes between processes (MessagePipe implements Handle directly)
- `port` - Listeners, watchers, pubsub (PortHandleAdapter)
- `channel` - Protocol-aware connections (ChannelHandleAdapter)

### G. Channels - Protocol-Aware I/O (`src/hal/channel.ts`)

**Purpose**: Enable message-based communication with external systems without exposing wire protocols to userland.

**Supported Protocols**:
- HTTP/HTTPS (via `fetch()`)
- WebSocket (via `new WebSocket()`)
- PostgreSQL (via `Bun.sql()`)
- Server-Sent Events / event streams

**Syscalls**:
- `channel:open(proto, url, opts)` → channelHandle
- `handle:send(ch, msg)` → Response stream
- `handle:close(ch)` → void

### H. Process Library (`rom/lib/process/`)

**Directory Structure**:
- `index.ts` - Re-exports all modules
- `types.ts` - OpenFlags, Stat, SpawnOpts, Message, Response, respond helper
- `syscall.ts` - Core syscall transport (postMessage, stream handling)
- `sigcall.ts` - Sigcall handler API (onSigcall, offSigcall)
- `respond.ts` - Response builders (ok, error, item, done, etc.)
- `error.ts` - SyscallError class
- `file.ts` - File operations (open, read, write, stat, etc.)
- `dir.ts` - Directory operations (mkdir, readdir, unlink, etc.)
- `net.ts` - Network operations (connect, listen, portRecv, portSend, etc.)
- `proc.ts` - Process operations (spawn, exit, wait, getpid, etc.)
- `env.ts` - Environment (getcwd, chdir, getenv, setenv)
- `io.ts` - Convenience I/O (readFile, writeFile, print, println)
- `pipe.ts` - Pipe and message I/O (pipe, recv, send)
- `channel.ts` - Channel operations (httpRequest, sqlQuery)
- `worker.ts` - Worker pool operations
- `access.ts` - Access control
- `head.ts`, `tail.ts` - Stream utilities

**Transport**:
- Messages format: `{ type: 'syscall:request'|'syscall:response'|'signal'|'sigcall:request', id, data }`
- Syscalls via `postMessage()` (IPC)
- Automatic stream ping every 100ms for backpressure
- Signal handling via `onSignal(handler)`
- Sigcall handling via `onSigcall(name, handler)`

**Convenience Wrappers**:
- `call<T>()` - Single request/response (awaits `ok` response)
- `collect<T>()` - Collect all items into array
- `iterate<T>()` - AsyncIterable (hides Response wrapper)

### I. Worker Pools (`src/kernel/pool.ts`)

**Purpose**: Kernel-managed worker pools for efficient process execution.

**Configuration** (`/etc/pools.json`):
```json
{
    "freelance": { "min": 2, "max": 32, "idleTimeout": 15000 },
    "compute": { "min": 4, "max": 64, "idleTimeout": 30000 }
}
```

**Features**:
- Named pools with isolation between workloads
- Auto-scaling: grows under pressure, shrinks when idle
- Backpressure: waiters queue when pool is exhausted
- Default 'freelance' pool as fallback

**Syscalls**:
- `pool:lease(pool?)` → workerId
- `worker:load(workerId, path)` → void
- `worker:send(workerId, msg)` → void
- `worker:recv(workerId)` → msg
- `worker:release(workerId)` → void
- `pool:stats()` → stats array

---

## 3. Message-Driven Architecture

### Core Principle
All internal kernel communication uses structured `Message` and `Response` objects. Byte serialization only at true I/O boundaries (disk, network).

### I/O Terminology
| Data Type | Input | Output | Example |
|-----------|-------|--------|---------|
| `Response` (messages) | `recv()` | `send()` | MessagePipe, ports |
| `Uint8Array` (bytes) | `read()` | `write()` | Files, sockets |

Process standard file descriptors use message terminology:
- fd 0: `recv` (receive messages)
- fd 1: `send` (send messages)
- fd 2: `warn` (diagnostic output)

### Message Format

```typescript
interface Message {
    op: string;              // Operation name: 'read', 'write', 'query', etc.
    data?: unknown;          // Operation-specific data (structured, not bytes)
}

interface Response {
    op: 'ok'|'error'|'item'|'data'|'event'|'progress'|'done'|'redirect';
    data?: object;           // Structured response data
}
```

### Response Semantics

| Op | Meaning | Terminal? | Usage |
|----|---------|-----------|-------|
| `ok` | Success, optional value | Y | Single-value syscalls (getpid, getcwd) |
| `error` | Failure with code/message | Y | Any syscall error (ENOENT, EBADF, etc.) |
| `item` | One item in sequence | N | Streaming: readdir, pubsub, database rows |
| `data` | Binary data (Uint8Array in `bytes` field) | N | File reads, network reads (at I/O boundary) |
| `event` | Async notification | N | File watch events, signals, server pushes |
| `progress` | Progress indicator | N | Long operations, download status |
| `done` | Sequence complete | Y | Marks end of streaming (readdir done, etc.) |
| `redirect` | Go elsewhere | Y | Path traversal (symlinks, mounts) |

### Handle Interface (Unified for all I/O)

```typescript
interface Handle {
    readonly id: string;
    readonly type: HandleType;  // 'file' | 'socket' | 'pipe' | 'port' | 'channel'
    readonly description: string;
    readonly closed: boolean;
    exec(msg: Message): AsyncIterable<Response>;
    close(): Promise<void>;
}
```

---

## 4. Streams & Backpressure

### Streams-First Philosophy
Streams of `Response` objects are the fundamental data flow unit. Arrays are a convenience wrapper.

### Transport Protocol

1. **Kernel sends multiple messages per syscall**: Each `Response` is a separate message
2. **Message format**: `{ type: 'response', id: string, result: Response }`
3. **Terminal ops signal completion**: Consumer knows stream ended at `ok`, `error`, `done`, or `redirect`
4. **Consumer acknowledges with ping**: `stream_ping` every 100ms for backpressure
5. **Consumer cancels when needed**: `stream_cancel` to abort stream

### Backpressure Mechanism

**Constants** (in `src/kernel/types.ts`):
- STREAM_HIGH_WATER = 1000 items queued
- STREAM_LOW_WATER = 100 items remaining
- STREAM_PING_INTERVAL = 100ms
- STREAM_STALL_TIMEOUT = 5000ms (abort if no ping)

**Behavior**:
- Kernel tracks items sent vs. acknowledged by consumer
- Pauses yielding at high-water mark
- Resumes yielding at low-water mark
- Aborts stream if no ping for 5s (consumer dead)

---

## 5. Technology Stack

### Core Technologies
| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Runtime** | Bun 1.0+ | JavaScript/TypeScript, Workers, primitives |
| **Language** | TypeScript 5.5+ | Strict type safety (target: ES2022) |
| **Storage** | SQLite 3 / PostgreSQL 14+ | Key-value, transactions, persistence |
| **HTTP** | Bun.serve, fetch | Server and client networking |
| **Networking** | Bun.listen, Bun.connect | TCP/UDP, raw sockets, Unix domain |
| **IPC** | SharedArrayBuffer, MessagePort | Inter-process synchronization |
| **Crypto** | Bun's WebCrypto | Hash, encryption, key derivation |
| **Serialization** | JSON only at I/O boundaries | Message objects internally |

### Directory Structure
```
.
├── src/                          # Kernel & core
│   ├── dispatch/                 # Dispatch layer (outside kernel)
│   │   ├── index.ts              # Exports Dispatcher
│   │   ├── dispatcher.ts         # Switch routing, sigcall routing
│   │   ├── types.ts              # Shared types
│   │   ├── syscall/              # Kernel-side syscall handlers
│   │   │   ├── vfs.ts            # file:*, fs:* syscalls
│   │   │   ├── ems.ts            # ems:* syscalls
│   │   │   ├── hal.ts            # net:*, port:*, channel:* syscalls
│   │   │   ├── process.ts        # proc:*, activation:* syscalls
│   │   │   ├── handle.ts         # handle:*, ipc:* syscalls
│   │   │   ├── pool.ts           # pool:*, worker:* syscalls
│   │   │   └── sigcall.ts        # sigcall:register/unregister/list
│   │   ├── sigcall/              # Sigcall infrastructure
│   │   │   ├── index.ts          # Exports registry
│   │   │   └── registry.ts       # Handler registration
│   │   └── stream/               # Backpressure protocol
│   │       ├── index.ts
│   │       ├── controller.ts
│   │       ├── syscall-controller.ts
│   │       ├── sigcall-controller.ts
│   │       ├── constants.ts
│   │       └── types.ts
│   ├── kernel/                   # Process & handle management
│   │   ├── kernel.ts             # Boot, lifecycle, onWorkerMessage
│   │   ├── kernel/               # Extracted kernel functions
│   │   ├── process-table.ts      # Process registry
│   │   ├── types.ts              # Process, SpawnOpts, etc.
│   │   ├── errors.ts             # Error re-exports
│   │   ├── services.ts           # Service definitions
│   │   ├── pool.ts               # Worker pool management
│   │   ├── pool-worker.ts        # Individual worker management
│   │   ├── boot.ts               # ROM → VFS copy
│   │   ├── mounts.ts             # Mount configuration
│   │   ├── poll.ts               # Event polling
│   │   ├── validate.ts           # Input validation
│   │   ├── handle/               # Unified handle system
│   │   │   ├── types.ts
│   │   │   ├── file.ts
│   │   │   ├── socket.ts
│   │   │   ├── port.ts
│   │   │   ├── channel.ts
│   │   │   ├── process-io.ts
│   │   │   └── console.ts
│   │   ├── resource/             # Port and pipe implementations
│   │   │   ├── types.ts
│   │   │   ├── listener-port.ts
│   │   │   ├── watch-port.ts
│   │   │   ├── udp-port.ts
│   │   │   ├── pubsub-port.ts
│   │   │   └── message-pipe.ts
│   │   └── loader/               # VFS module loader
│   │       ├── vfs-loader.ts
│   │       ├── imports.ts
│   │       ├── cache.ts
│   │       └── types.ts
│   ├── vfs/                      # Virtual filesystem
│   │   ├── index.ts              # Exports
│   │   ├── vfs.ts               # Path resolution, mounts
│   │   ├── model.ts              # Model interface
│   │   ├── handle.ts             # FileHandle interface
│   │   └── models/               # Model implementations
│   │       ├── file.ts
│   │       ├── folder.ts
│   │       ├── device.ts
│   │       ├── proc.ts
│   │       └── link.ts
│   ├── hal/                      # Hardware abstraction
│   │   ├── index.ts              # HAL interface, BunHAL
│   │   ├── errors.ts             # POSIX error classes
│   │   ├── block.ts              # BlockDevice
│   │   ├── storage.ts            # StorageEngine
│   │   ├── network.ts            # NetworkDevice
│   │   ├── timer.ts              # TimerDevice
│   │   ├── clock.ts              # ClockDevice
│   │   ├── entropy.ts            # EntropyDevice
│   │   ├── crypto.ts             # CryptoDevice
│   │   ├── console.ts            # ConsoleDevice
│   │   ├── dns.ts                # DNSDevice
│   │   ├── host.ts               # HostDevice
│   │   ├── ipc.ts                # IPCDevice
│   │   ├── channel.ts            # ChannelDevice
│   │   └── compression.ts        # CompressionDevice
│   ├── ems/                      # Entity Model System
│   │   ├── index.ts              # Exports
│   │   ├── entity-ops.ts         # Streaming entity operations
│   │   ├── database-ops.ts       # Generic SQL streaming
│   │   ├── database-connection.ts # HAL channel wrapper
│   │   ├── model-cache.ts        # Metadata caching
│   │   ├── entity-cache.ts       # Entity instance caching
│   │   └── observers/            # Observer pipeline
│   │       ├── index.ts
│   │       ├── observer.ts       # Base observer class
│   │       ├── sql-create.ts     # Ring 5: SQL INSERT
│   │       ├── sql-update.ts     # Ring 5: SQL UPDATE
│   │       ├── sql-delete.ts     # Ring 5: SQL DELETE
│   │       ├── cache.ts          # Ring 8: Cache invalidation
│   │       └── ...
│   ├── os/                       # Public API
│   │   ├── index.ts              # Exports OS class, types, APIs
│   │   ├── os.ts                 # OS class (boot, exec, shutdown, syscall wrappers)
│   │   ├── types.ts              # OSConfig, BootOpts, ExecOpts, OSEvents
│   │   └── README.md             # API documentation
│   ├── gateway/                  # External syscall interface
│   │   ├── index.ts              # Exports Gateway class
│   │   ├── gateway.ts            # MessagePack Unix socket server
│   │   └── README.md             # Wire protocol docs
│   └── message.ts                # Message, Response, respond helpers
├── rom/                          # Read-only bundled code (userspace)
│   ├── lib/                      # Libraries for processes
│   │   ├── errors.ts             # Error definitions
│   │   └── process/              # Syscall wrappers
│   │       ├── index.ts
│   │       ├── types.ts
│   │       ├── syscall.ts
│   │       └── ...
│   ├── app/                      # Applications (registered as services)
│   │   ├── ai/
│   │   │   ├── main.ts           # AI assistant daemon
│   │   │   └── service.json      # Service definition
│   │   ├── displayd/
│   │   │   ├── main.ts           # Display server
│   │   │   └── service.json
│   │   └── ...
│   ├── bin/                      # Executable programs
│   │   ├── true.ts               # Exit 0
│   │   └── false.ts              # Exit 1
│   └── etc/                      # Configuration
│       └── mounts.json           # Mount configuration
├── spec/                         # Tests
│   ├── kernel.test.ts
│   ├── vfs.test.ts
│   └── ...
├── AGENTS.md                    # This file
├── package.json
├── tsconfig.json
└── bunfig.toml
```

### App Services (`rom/app/`)

Applications are registered as services via `service.json`:

```json
{
    "handler": "main.ts",
    "activate": { "type": "manual" },
    "description": "AI assistant daemon"
}
```

**Activation types:**
| Type | Behavior |
|------|----------|
| `boot` | Start automatically on kernel.boot() |
| `manual` | Only via `os.service('start', 'name')` |
| `tcp:listen` | Socket activation - spawn per connection |
| `pubsub:subscribe` | Topic activation - spawn per message |
| `fs:watch` | File watch - spawn per event |

**Service discovery order:**
1. `/etc/services/*.json` - system services
2. `/usr/*/etc/services/*.json` - package services
3. `/app/*/service.json` - app services

Note: The kernel process (id='kernel') is PID 1, created during kernel.init(). Apps are separate services started manually or via activation triggers.

### Gateway (`src/gateway/`)

The Gateway provides external applications (os-sdk) access to syscalls over TCP (port 7778) and WebSocket (port 7779). It runs in kernel context (not as a Worker), executing syscalls directly via `dispatcher.execute()`.

**Wire Protocol**: Length-prefixed MessagePack over TCP.

```
[4-byte big-endian length][msgpack payload]
```

**Request**:
```javascript
{ id: "abc", call: "file:open", args: ["/etc/hosts", { read: true }] }
```

**Response**:
```javascript
{ id: "abc", op: "ok", data: { fd: 3 } }
{ id: "abc", op: "data", bytes: Uint8Array([...]) }  // Binary data native
{ id: "abc", op: "error", code: "ENOENT", message: "..." }
```

Each client connection gets an isolated **virtual process** with its own file descriptor table, cwd, and environment. No Worker thread overhead per connection.

### External SDK (os-sdk)

External applications connect via the Gateway using os-sdk (`@monk-api/os-sdk`). The SDK provides a TypeScript client that handles MessagePack encoding and the wire protocol.

```typescript
import { OSClient } from '@monk-api/os-sdk';

const client = new OSClient();
await client.connect({ socketPath: '/tmp/monk.sock' });

const fd = await client.open('/etc/hosts', { read: true });
const data = await client.read(fd);
await client.close(fd);
```

---

## 6. Key Patterns & Best Practices

### Pattern: HAL Abstraction
- Define interface in device file (e.g., `block.ts`)
- BunHAL aggregates all devices
- Inject via `new BunHAL(config)`
- `init()` and `shutdown()` for lifecycle

### Pattern: Model-Based VFS
- Each filesystem category is a Model (file, folder, device, proc, link)
- Model implements `open()`, `stat()`, `list()`, `unlink()`, `watch()`
- VFS routes paths to correct model via mount table
- FileHandles provide I/O, Models provide metadata

### Pattern: Message-Passing Syscalls
- Syscalls return `AsyncIterable<Response>` (generators)
- Yield `item` for each element, `done` to finish
- Yield `error` (throw) for failures
- Yield `ok` for single values
- Kernel handles streaming, backpressure, and transport

### Pattern: Unified Handle Architecture
- All I/O primitives are Handles with `exec(msg)` → `AsyncIterable<Response>`
- Handle types: file, socket, pipe, port, channel
- Kernel maintains global handle table with reference counting
- Process maintains local fd → handle ID mapping

### Pattern: Streaming Over Arrays
- Default: `async function* iter(): AsyncIterable<T>`
- Convenience: `collect()` wrapper for small datasets
- Benefits: O(1) memory, natural for databases/files, natural for backpressure

### Pattern: Process Identity
- Each process has UUID (via hal.entropy.uuid())
- File descriptor table maps small integers (0-255) to handle UUIDs
- Ports have IDs (tcp:listen, pubsub, etc.)
- Everything is globally addressable via UUID

---

## 7. Implementation Constraints & Rules

### Non-Negotiable Rules
1. **No JSON serialization in kernel** - Only at true I/O boundaries (disk, network)
2. **Message objects everywhere internally** - Not byte strings
3. **All async operations yield responses** - Not Promise<T>, but `AsyncIterable<Response>`
4. **Streams are primary, arrays are convenience** - Use `collect()` for arrays when needed
5. **File descriptors are per-process integers** - They map to global handle UUIDs
6. **Permissions checked at open-time** - Once authorized, Handle is capability
7. **Workers are process boundaries** - Communication via syscalls/postMessage only
8. **UUID identity for everything** - Files, processes, handles, ports
9. **No Bun primitives outside HAL** - Use HAL abstractions; if missing, add to HAL first

### Error Handling
- **NEVER use `new Error()`** — Always use typed errors from `src/hal/errors.ts`
- Available types: `ENOENT`, `EACCES`, `EBADF`, `EINVAL`, `EEXIST`, `ENOTDIR`, `EISDIR`, `ENOSPC`, `ETIMEDOUT`, `EIO`
- Kernel catches typed errors, converts to `{ op: 'error', code, message }` response
- Process library unwraps via SyscallError class
- Tests should verify error codes, not just catch/ignore
- See `prompts/kernel-dev.md` for full error handling patterns

### Testing Strategy
- **Unit tests**: Mock HAL, test kernel in isolation
- **Integration tests**: Real processes, real syscalls
- **Stream tests**: Verify ping, cancel, timeout behavior
- **Error tests**: Verify error codes and messages

---

## 8. Service Activation & Configuration

**Service Configuration** (`/etc/services/*.json`):
```json
{
    "handler": "/bin/logd",
    "activate": {
        "type": "boot"
    },
    "io": {
        "stdin": { "type": "pubsub", "subscribe": ["log.*"] },
        "stdout": { "type": "file", "path": "/var/log/system.log" },
        "stderr": { "type": "console" }
    }
}
```

**Activation Types**:
- `boot` - Service starts on kernel boot
- `tcp:listen` - Socket activation on TCP listen (port, host optional)
- `udp` - UDP socket activation
- `pubsub` - Topic-based activation
- `watch` - File system event activation

**I/O Source Types** (for recv/fd 0):
- `console` - Read from console
- `file` - Read from file path
- `null` - Always EOF
- `pubsub` - Subscribe to topics
- `watch` - Watch file patterns
- `udp` - Receive UDP datagrams

**I/O Target Types** (for send/warn, fd 1/2):
- `console` - Write to console
- `file` - Write to file path (with create/append options)
- `null` - Discard writes

> **Note**: Config files use "stdin/stdout/stderr" keys for compatibility. Internally, fd 0/1/2 are called recv/send/warn to reflect message-based I/O.

---

## 9. Quick Reference: Common Syscalls

```typescript
// File I/O
fd = await open('/path', { read: true });     // Open file
for await (const chunk of read(fd)) { ... }   // Read bytes (streaming)
n = await write(fd, data);                     // Write bytes
await close(fd);                               // Close
stat = await stat('/path');                    // Get metadata
for await (const entry of readdir('/path')) { ... }  // List directory
await mkdir('/path');                          // Create directory
await unlink('/path');                         // Delete file

// Process Management
pid = await spawn('/bin/prog', opts);          // Create process
status = await wait(pid);                      // Wait for exit
await kill(pid, signal);                       // Send signal
await exit(code);                              // Exit process
pid = await getpid();                          // Get own PID
ppid = await getppid();                        // Get parent PID

// Networking
fd = await connect('host', port);              // TCP connect
portId = await listen({ port: 8080 });         // TCP listen
msg = await portRecv(portId);                  // Receive from port
await portSend(portId, to, data);              // Send to port
await pclose(portId);                          // Close port

// Message-based I/O (fd 0/1/2)
for await (const msg of recv(0)) { ... }       // Receive messages from fd
await send(1, respond.item({ text }));         // Send message to fd
await println('hello');                        // Convenience: send text + newline

// Pipes (message-based)
[recvFd, sendFd] = await pipe();               // Create message pipe

// Environment
cwd = await getcwd();                          // Get working directory
await chdir(path);                             // Change directory
val = await getenv(name);                      // Get env var
await setenv(name, val);                       // Set env var

// Channels (protocol-aware I/O)
ch = await channel('postgres', url);           // Open connection
result = await httpRequest(ch, req);           // HTTP request
rows = await sqlQuery(ch, sql, params);        // SQL query
await close(ch);                               // Close connection

// Worker Pools
workerId = await pool.lease('compute');        // Lease worker
await worker.load(workerId, '/script.ts');     // Load script
await worker.send(workerId, msg);              // Send message
result = await worker.recv(workerId);          // Receive message
await worker.release(workerId);                // Release to pool
```

---

## 10. Completeness Status

**Overall: ~95%** — Production-ready for single-node (SQLite) or distributed (PostgreSQL) use.

| Component | Status | Notes |
|-----------|--------|-------|
| Dispatch Layer | 100% | Complete with sigcall routing (src/dispatch/) |
| Kernel/Core | 95% | Full process lifecycle, worker pools, handle management |
| Process Mgmt | 95% | UUID/PID, signals, parent-child, worker isolation |
| VFS | 95% | Plan 9 "everything is a file", hybrid EMS/HAL storage |
| EMS | 95% | 8-ring observer pipeline, streaming queries |
| IPC | 90% | Pipes, ports, channels, shared memory (mutex/semaphore/condvar) |
| Networking | 85% | TCP/UDP, HTTP/WS, PostgreSQL/SQLite channels |
| HAL Devices | 95% | 14 devices: SQLite + PostgreSQL storage backends |
| Boot | 95% | ROM bootstrap, service activation, lifecycle events |
| Public API | 95% | OS class rewritten with syscall wrappers, convenience helpers |
| Gateway | 100% | MessagePack Unix socket interface for os-sdk |

**Storage Backends**:
- **SQLite** (`BunStorageEngine`) — Embedded, single-node, WAL mode
- **PostgreSQL** (`PostgresStorageEngine`) — Distributed, multi-node, full MVCC
- **Memory** (`MemoryStorageEngine`) — Testing, ephemeral

**VFS Hybrid Architecture**:
- File/Folder entities use EMS (SQL tables, EntityCache, observer pipeline)
- Device/Proc/Link entities use HAL KV storage (virtual, no persistence needed)
- Path resolution transparently handles both via child indexes

**Key TODOs**:
- Implement `EntityModel.watch()`
- Full UDP exposure to userland
- Cross-process watch via PostgreSQL LISTEN/NOTIFY

---

**Last Updated**: December 2024 (v0.5.0: Gateway moved to kernel, MessagePack wire protocol, os-sdk integration)
**Next Review**: As needed
**Maintainer**: @monk-api/os team

---
> Source: [ianzepp/monk-os-kernel](https://github.com/ianzepp/monk-os-kernel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
