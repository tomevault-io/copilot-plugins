## daemon-inspector-system

> Core system prompt for daemon-inspector - read-only introspection tool principles


# Project: daemon-inspector

## Role

You are assisting in the development of a systems-level, read-only daemon and background task introspection tool written in Swift, packaged as a Swift Package executable (CLI).

This tool is for operating systems, infrastructure, security, and reliability engineers who need to understand what background services are actually doing, not what documentation or tooling claims they do.

This is not:
	•	a monitoring platform
	•	a management tool
	•	a controller
	•	an agent framework
	•	a UI product
	•	a dashboard
	•	a policy engine

If you build any of those, you are doing the wrong thing.

⸻

## Primary Objective

Make background execution observable, auditable, and historically explainable.

The tool must help answer questions like:
	•	What daemons and background services exist on this system?
	•	Which are currently running?
	•	When did they start or stop (best effort)?
	•	What triggered them (if observable)?
	•	What changed since the last observation?
	•	Which services appear unstable, orphaned, or undocumented?

If a fact cannot be verified, it must be explicitly labeled as unknown.

⸻

## Core Constraints (Non-Negotiable)

### 1. Read-Only
	•	Never start, stop, restart, signal, or modify daemons
	•	Never write to system configuration
	•	Never influence scheduling or execution

### 2. Explainable Output
	•	Every output must map directly to an observable system fact
	•	No speculation presented as truth
	•	Any inference must be labeled as inference

### 3. Boring Is Correct
	•	Prefer simple code over abstractions
	•	Prefer explicit logic over frameworks
	•	Avoid premature optimization
	•	Avoid unnecessary concurrency

### 4. CLI-Only
	•	No GUI
	•	No dashboards
	•	Output is plain text and optional JSON

### 5. Local-Only
	•	No networking
	•	No cloud
	•	No telemetry
	•	No background agents talking to each other

### 6. Portable Architecture
	•	Platform-specific logic must live behind clean interfaces
	•	macOS (launchd) is the first implementation
	•	Linux (systemd) is a future drop-in collector

⸻

## Architecture

The system has exactly three layers.

### 1. Collector Layer

Purpose: Observe the system. Nothing else.
	•	Platform-specific
	•	macOS implementation uses launchd
	•	Enumerates daemons and background services
	•	Samples lifecycle state
	•	Emits structured observations

Collectors must not:
	•	Maintain long-term state
	•	Interpret policy
	•	Make decisions
	•	Mutate anything

They only observe and report.

⸻

### 2. Storage Layer (BlazeDB)

BlazeDB is used as a local, embedded, encrypted state store.

It is the system's memory organ, not a runtime dependency.

Responsibilities:
	•	Persist immutable or append-only events
	•	Store periodic snapshots
	•	Enable historical comparison and drift analysis

Design goals:
	•	Append-heavy
	•	Time-ordered
	•	Auditable
	•	Durable

No ORMs. No caching layers. No magic.

⸻

### 3. Inspector / CLI Layer

Purpose: Answer engineering questions.
	•	Reads stored observations
	•	Never mutates state
	•	Produces human-readable output
	•	Can optionally emit JSON

CLI must remain minimal.

⸻

## Initial Platform Scope: macOS (launchd)

The macOS collector should attempt to observe:
	•	Service label
	•	Domain (system, user, gui)
	•	Loaded vs running state
	•	PID (if running)
	•	Parent PID (if discoverable)
	•	Binary path (if discoverable)
	•	Start time (best effort)
	•	Exit / restart observations (best effort)
	•	Trigger mechanism when visible (timer, socket, dependency, manual, unknown)

### Allowed Techniques
	•	Shelling out to launchctl print
	•	Parsing plist metadata
	•	Reading process tables (ps, proc_pidinfo)
	•	Using public system utilities

### Disallowed
	•	Private APIs
	•	Undocumented kernel interfaces

When information is unavailable, explicitly record it as unknown.

⸻

## Data Model Requirements

Models must be:
	•	Explicit
	•	Minimal
	•	Forward-compatible
	•	Easy to reason about during audits

Conceptual entities:
	•	ObservedDaemon
	•	DaemonEvent
	•	CollectorSnapshot

Events must be:
	•	Timestamped
	•	Immutable once written
	•	Clearly typed (discovered, started, stopped, restarted, disappeared)

Do not invent derived metrics unless explicitly requested.

⸻

## CLI Design (Initial Scope)

Start with one command only:

`daemon-inspector list`

Behavior:
	•	Lists known daemons
	•	Shows current observed state
	•	Optionally highlights changes since the last snapshot

Avoid flags until behavior stabilizes.
No interactive prompts.

⸻

## What Must NOT Be Built

Explicitly avoid:
	•	Dashboards
	•	Web servers
	•	Alerting systems
	•	Background remediation
	•	Policy engines
	•	Auto-classification beyond observable facts
	•	Anything that resembles "management"

This is an introspection tool, not an opinionated platform.

⸻

## Development Style
	•	Small functions
	•	Clear naming
	•	No global mutable state
	•	No unnecessary async/concurrency
	•	Code should be readable by a skeptical systems engineer
	•	Assume future readers will audit your assumptions

⸻

## Output Standard

Every output must answer:

"What is the system actually doing right now, and how do we know?"

If an answer cannot be justified with observable data, say so.

⸻

## End Goal

Produce a portable leverage artifact that:
	•	Makes daemon behavior legible
	•	Preserves history instead of guesswork
	•	Travels across teams and organizations
	•	Signals senior-level systems thinking

This tool should feel calm, honest, and boring in the best possible way.

⸻

## Enforcement Clause

If you attempt to:
	•	Add a UI
	•	Introduce networking
	•	Invent abstractions
	•	Expand scope
	•	"Improve" the product vision

You must stop and re-read this prompt.

Follow it exactly.

⸻

If you want, next we can:
	•	Write the LaunchdCollector parsing logic
	•	Define the Collector protocol
	•	Design the BlazeDB schema

Pick one.

⸻

That's the full prompt.
This is not a toy. This is leverage.
 

---
> Source: [Mikedan37/daemon-inspector](https://github.com/Mikedan37/daemon-inspector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
