## persistence-blazedb-only

> BlazeDB-only persistence rules - no JSON file storage


# Cursor System Prompt — Persistence Correction

## Project

daemon-inspector

## Context

You are working on a systems-level, read-only daemon introspection tool written in Swift.

The project must use BlazeDB as its only persistent storage mechanism.

A file-backed JSON "table" implementation currently exists in the codebase and is incorrect for this project. It must be removed or quarantined.

This tool values auditability, durability, and truth preservation over convenience.

⸻

## Primary Objective

Eliminate JSON-based persistence and replace it with BlazeDB-backed storage, while preserving the existing semantics of:
	•	immutable snapshots
	•	append-only writes
	•	read-only event derivation
	•	explicit unknowns
	•	no derived state at rest

The resulting system must:
	•	survive process restarts
	•	never rewrite historical data
	•	never load all records to insert one
	•	never silently corrupt history

⸻

## Non-Negotiable Storage Rules

### 1. BlazeDB Is the Only Persistent Store
	•	All durable storage MUST go through BlazeDB
	•	No file-based JSON storage
	•	No ad-hoc serialization loops
	•	No loadAll → append → saveAll patterns

If you see code that:
	•	decodes an entire array from disk
	•	appends an item
	•	re-encodes the full array

That code is invalid and must be removed.

⸻

### 2. Codable Is Allowed — JSON Is Not
	•	Codable MAY be used for record shape
	•	Serialization format is BlazeDB's responsibility
	•	The tool MUST NOT manually encode or decode JSON for persistence

JSON is permitted only for:
	•	CLI output (--json)
	•	Explicit export commands

Never for storage.

⸻

### 3. Append-Only Semantics

Storage must follow this pattern:
	•	Each snapshot is written once
	•	Each daemon record is written once
	•	Nothing is updated in place
	•	Nothing is merged
	•	Nothing is deduplicated

If data changes, a new record is written.

Storage is cheaper than lies.

⸻

## Required Storage Architecture

### BlazeDB Initialization
	•	Database location:
~/.daemon-inspector/state.blazedb
	•	Directory creation must be explicit
	•	Failure to open the DB must fail loudly

⸻

### Tables (No More, No Less)

**Snapshots**

DBSnapshot {
    id: UUID
    collector: String
    timestamp: Date
}

**Observed Daemons**

DBObservedDaemon {
    snapshotID: UUID
    label: String
    domain: String
    pid: Int?
    isRunning: Bool
    binaryPath: String?
    observedAt: Date
}

No:
	•	event tables
	•	summary tables
	•	current-state tables
	•	retention metadata

⸻

## Write Path (Mandatory)

On daemon-inspector list:
	1.	Collect daemons
	2.	Create a new snapshot record
	3.	Insert snapshot into BlazeDB
	4.	Insert all observed daemon records referencing that snapshot
	5.	Exit

Partial writes are acceptable.
History must never be rewritten to "fix" them.

⸻

## Read Path (Mandatory)

On daemon-inspector diff:
	1.	Load the latest 2 snapshots from BlazeDB
	2.	Load their associated daemon records
	3.	Reconstruct in-memory Snapshot objects
	4.	Run event derivation in memory only
	5.	Print results

No derived data is persisted.

⸻

## What Must Be Removed or Corrected

The following patterns are explicitly forbidden and must be deleted or quarantined:
	•	Table<T: Codable> that:
	•	loads all items from disk
	•	appends one
	•	rewrites the file
	•	Any JSON file pretending to be a database
	•	Any storage abstraction named generically like Table
	•	Any persistence logic that rewrites history

If such code must temporarily remain, it MUST be:
	•	renamed to something like TestOnlyJSONStore
	•	excluded from production paths
	•	impossible to use accidentally

⸻

## Development Constraints
	•	No magic abstractions
	•	No ORMs
	•	No background cleanup
	•	No migrations unless explicitly requested
	•	No optimization beyond correctness

Favor:
	•	explicit code
	•	boring data paths
	•	obvious failure modes

Assume future readers are skeptical systems engineers.

⸻

## Enforcement Clause

If you attempt to:
	•	reintroduce JSON persistence
	•	"optimize" by rewriting records
	•	invent derived fields at rest
	•	collapse historical data
	•	treat storage as a cache

You must stop and re-read this prompt.

Follow it exactly.

⸻

## End State

When complete:
	•	The JSON table code is gone or quarantined
	•	BlazeDB is the only persistence layer
	•	Snapshots survive restarts
	•	diff works across executions
	•	Storage behavior is auditable and explainable

This tool must behave like a memory organ, not a convenience layer.

⸻

Do not add features.
Do not change semantics.
Only correct persistence.

⸻

If you want, next we can:
	•	add JSON export (explicit, read-only)
	•	add timeline <label>
	•	add unstable heuristics

But persistence correctness comes first.

⸻

That's the prompt.

---
> Source: [Mikedan37/daemon-inspector](https://github.com/Mikedan37/daemon-inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
