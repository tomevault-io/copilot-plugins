## spiceport

> Guidance for working in this repository. See `README.md` for the project overview,

# CLAUDE.md

Guidance for working in this repository. See `README.md` for the project overview,
`docs/architecture-analysis.md` for the design rationale, `docs/graph-sharded-datastore.md`
for the storage design as built, `docs/future-work.md` for candidate directions that are
analyzed but not committed, and `docs/scalability-program.md` for the measurement-gated
performance program (nothing there is scheduled; triggers decide).

## What this is

A SpiceDB-compatible **rearchitecture** of [SpiceDB](https://github.com/authzed/spicedb)
(Google Zanzibar) on .NET 10 + Microsoft Orleans — the graph-evaluation engine and schema
compiler are ported faithfully; the system around them (dispatch, caching, storage,
distribution) is re-founded on virtual actors rather than translated mechanism-for-mechanism.
The recursive permission-check dispatch runs on Orleans virtual actors; the gRPC surface is
`authzed.api.v1`-compatible (the real `zed` CLI works against it).

## Build & test

```bash
dotnet build                                   # whole solution
dotnet test                                    # all tests
dotnet test tests/Spiceport.Conformance.Tests  # the SpiceDB conformance corpus (fast)
```

- Target framework is `net10.0`; nullable + implicit usings are ON (see `Directory.Build.props`).
- **Use the dotnet CLI for package/reference/project changes** (`dotnet add package`,
  `dotnet add reference`, `dotnet sln add`). Do not hand-edit `<PackageReference>`/
  `<ProjectReference>` items. Editing build items like `<Protobuf>` is fine.
- The grain-storage **durability tests** (`tests/Spiceport.Grains.Tests/Durability`) use
  Testcontainers and require Docker running; they spin up and dispose their own `postgres`
  container (and skip when Docker is unavailable).
- The solution file is `Spiceport.slnx` (the .NET 10 XML solution format).

## Architecture (the load-bearing ideas)

- **Storage is not *dispatch*-grain state.** Evaluation is a pure function of
  `(schema@revision, tuples@revision, request)`; dispatch grains never hold relationship data.
  The MVCC mechanics (visibility at a revision, the per-revision diff) live in `Spiceport.Datastore`
  and are reused everywhere — the `ReferenceDatastore` reference model and the per-key graph-shard
  folds (`ShardFold`, a key-restriction of the same fold) share one set of semantics.
- **Storage is an event-sourced grain (the log is the storage/compute seam).** All
  relationship/schema/counter state lives behind a single cluster-singleton `DatastoreGrain`, a
  **journaled grain whose append-only `LogEvent` log is the source of truth**; the materialized state
  is the fold. A commit is a version-checked **append** (the CAS serialization point), not a
  whole-state rewrite; the single non-reentrant activation makes the minted revision the cluster-wide
  global order. Persistence is the grain's own via `ICustomStorageInterface` over an Orleans
  grain-storage provider — **no application SQL** (per-version log entries + periodic snapshots +
  compaction). Engine reads go through **per-key `GraphShardGrain`s** (forward shards keyed by
  object, reverse shards keyed by subject — resolved via `ShardedGraphReader` behind the
  `IGraphReaderSource` seam): each shard's activation state is the per-key restriction of the fold
  (`ShardFold`), hydrated once from `ReadShard` and advanced by tailing the log, so activation *is*
  the hot-set cache — cold keys never activate, silo memory is O(hot shards), not O(graph). The
  per-shard `AppliedRevision` watermark is the closed-timestamp gate: exact/at-least-as-fresh reads
  block until the shard's watermark covers the pinned revision. Writes are **declarative
  `DatastoreGrain.Commit`** — preconditions/updates/deleteByFilter/schema (with `ExpectedSchemaHash`)
  /counters evaluated and executed on the sequencer's single non-reentrant activation, rejections
  returned as typed `CommitReply` failures with nothing applied; `ReadWriteTx` survives only as the
  `ExpectedHead` compatibility path over the same wire contract. Broad/admin scans (loose-filter
  ReadRelationships, bulk export, counters) and schema-at-revision bypass the shard mesh and fetch
  the sequencer snapshot (`ISnapshotScanner` / `ISchemaSource`). The sequencer grain still
  materializes the whole fold; slimming it (per-shard durable snapshots + tail trim) is the recorded
  next step (`docs/graph-sharded-datastore.md` §7). The same log feed
  drives **Watch** (one per-silo `LogWatchHub` notifier, no per-stream polling) and an on-by-default
  (opt-out via `MembershipWalkOptions`) **Leopard membership-walk grain mesh** (`IMembershipWalkGrain`,
  sharded as addressable per-subject walk grains — sibling recursion across grain boundaries, cold sets
  deactivate, revision-exact by construction because each hop reads a pinned MVCC snapshot) for
  `LookupResources` (**candidates, never verdicts**: Check confirmation guarantees soundness — no false
  member can survive; completeness is pinned by the walk-on==walk-off equivalence gates, because a
  silently *missing* candidate is the failure Check confirmation cannot see). See
  `docs/architecture-analysis.md` §3.5.
- **The dispatcher seam is the core mechanism.** `Spiceport.Engine`'s `CheckEngine` never
  recurses into itself directly — every sub-problem flows through `IDispatcher.DispatchCheck`.
  Implementations are `OrleansDispatcher` (resolves a grain call via the Orleans directory) and
  `LocalDispatcher` (one expansion step). The grain identity *is* the canonical sub-problem
  `(resourceType, resourceId, relation, subject, quantizedRevision, schemaHash)`.
- **Dispatch via grain calls.** Every sub-problem is a grain call; the Orleans grain directory
  owns location. `CheckGrain` activation state is the only dispatch cache, memoizing the
  pre-context `Branch` with idle-collection eviction. An **exact visited set** carries the
  cycle guard across grain boundaries.
- **Caching subtleties (do not regress):** the `CheckGrain` activation memo stores the
  *pre-context* `Branch` (membership + caveat expression), never the collapsed verdict — caveat
  context is applied per-request at the caller. The grain key is exactly the canonical
  sub-problem; the exact visited set and `DepthRemaining` ride ambient in `RequestContext` via
  `DispatchContext`, not in the grain key. Cycle-cut results are served but not retained.
  Revisions are quantized so grain keys are shared within a window; `schemaHash` is in the key
  so a schema change yields a fresh keyspace.
- **Consistency.** Reads honor a `ConsistencyRequirement`; consistency is enforced entirely at
  `RevisionResolver` time — which revision string gets pinned, plus the per-shard
  closed-timestamp watermark gate — so fresh/at-exact/fully-consistent reads never serve stale data. The
  pinned revision string is the grain key's whole identity: no separate cache-mode segment exists
  downstream of resolution.

## Conventions

- Idiomatic modern C#: records for immutable data, file-scoped namespaces, `IAsyncEnumerable`
  for streaming, `[GenerateSerializer]` for any type crossing a grain boundary.
- Keep engine logic out of the API layer. gRPC service classes are pure translation over the
  grains and the Server-layer read helpers (`ReverseOps` over the `IGraphReaderSource` shard mesh,
  `RelationshipReads` over the `ISnapshotScanner` scan seam) that run in-process on the serving
  silo — the same pattern `AuthzedWatchV1Service` uses for Watch;
  engine/graph logic lives in `Spiceport.Engine`/`Spiceport.Grains`.
- Map errors to gRPC status codes deliberately (e.g. CREATE-conflict -> `AlreadyExists`,
  precondition/schema-validation failure -> `FailedPrecondition`, bad consistency token ->
  `InvalidArgument`). A wrong code makes `zed` retry or crash.
- Cross-grain exceptions must be `[GenerateSerializer]` to round-trip the Orleans boundary.

## Testing discipline

- **The SpiceDB conformance corpus is the compatibility anchor — a finite regression suite, not an
  oracle.** `tests/.../TestData/*.yaml` (schema + relationships + Check/Lookup assertions) must stay
  green; the same corpus runs through the engine over the `ReferenceDatastore` reference model and
  the Orleans grain mesh, and both must agree. Never weaken/skip a corpus case to make something
  pass. Know its limits: it covers only the shapes its cases exercise, both sides of the "two-way"
  run share the same engine (agreement proves the distribution layer, not engine semantics), and it
  tests one static snapshot (no MVCC/revision/write-race behavior). Correctness beyond it rests on
  the property-based, metamorphic, and differential gates. `tests/Spiceport.Differential.Tests` is
  the genuinely external one — it runs seeded random worlds through both a real `authzed/spicedb`
  container and Spiceport's in-process grain mesh and asserts Check/Lookup verdicts agree; it needs
  Docker and skips (not fails) without it.
- **Verify grains via the Orleans `TestingHost`** (in-process `TestCluster`), not by booting a
  host. For server/client-streaming gRPC, drive the service in-process with a fake
  `IServerStreamWriter`/`IAsyncStreamReader` + a fake `ServerCallContext`. Do **not** start a
  Kestrel host (`dotnet run`) inside tests/CI — a backgrounded host can orphan and run forever.
- A real `zed`/`grpcurl` smoke test against a booted host is valuable but is an attended,
  manual step with explicit host teardown — keep it out of the automated suite. The
  real-network measurement rig (`tools/rig/rig.sh`, `tools/Spiceport.RigSilo`) has the same
  standing: attended and manual, never invoked from tests/.
- Cluster-using tests share a non-parallel xUnit collection (the test cluster passes schema via
  a process-wide static).

## Protos

`authzed.api.v1` is vendored under `src/Spiceport.Protos/Protos/authzed/` via
`buf export buf.build/authzed/api`. Grpc.Tools generates `Authzed.Api.V1` server bases; the
`<Protobuf>` items set `ProtoRoot`/`AdditionalImportDirs` so the `buf/validate` + `google/api`
import deps resolve (compiled message-only). To add an RPC, override the generated base in the
relevant `Authzed*V1Service` and map it onto an existing grain.

## House style (from the maintainer)

- No emojis. Prefer semantic HTML in any web UI. Classic (sociable) TDD over mockist.
- Don't write status updates (test counts, dates, "currently…") into committed docs — keep
  `README.md`/`CLAUDE.md`/`docs/` evergreen.

---
> Source: [dave-hillier-co/spiceport](https://github.com/dave-hillier-co/spiceport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
