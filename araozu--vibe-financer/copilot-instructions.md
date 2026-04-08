## vibe-financer

> This project follows a simplified layered architecture to maintain separation of concerns without the overhead of full Clean Architecture ceremony (no mandatory interfaces or dependency injection containers).

# Project Architecture: Layered Domain-Driven Design (Lite)

This project follows a simplified layered architecture to maintain separation of concerns without the overhead of full Clean Architecture ceremony (no mandatory interfaces or dependency injection containers).

## Platform

- Use bun
- Database: PostgreSQL (using `node-postgres` / `pg` driver and `drizzle-orm/node-postgres`)
- Timestamps: Always use `timestampz` (timestamps with timezones) in the database.

## Language

- Always use the Null coalescing operator `??`

## Layers

### 1. Domain Layer (`src/lib/domain/`)

- **Responsibility**: Core business logic, pure functions, and domain-specific types.
- **Rules**:
  - Must be "pure" (side-effect free).
  - **No imports** from Infrastructure or Application layers.
  - Contains logic like budget calculations, transaction validation rules, and financial formatting.

### 2. Infrastructure Layer (`src/lib/infra/`)

- **Responsibility**: Technical implementation details and external systems.
- **Contents**:
  - `db/`: Drizzle schema and client initialization.
  - `repos/`: Database-specific queries (Repositories).
- **Rules**:
  - This is the only place where Drizzle `db` or specific SQL queries should live.

### 3. Application Layer (`src/lib/application/`)

- **Responsibility**: Orchestration of use cases.
- **Rules**:
  - The "glue" between Domain and Infrastructure.
  - Coordinates workflows (e.g., "Create Transaction" -> validate via Domain -> save via Infra Repo).
  - Does not contain complex business logic itself; it delegates to the Domain.

### 4. Presentation Layer (`src/routes/` & `src/lib/components/`)

- **Responsibility**: UI (Svelte components) and SvelteKit entry points (+page.server.ts).
- **Rules**:
  - `+page.server.ts` loaders and actions should call Application services/functions.
  - UI components should be kept lean, focusing on display and user interaction.

---

## Directory Mapping

```text
src/lib/
  ├── domain/        # Business logic & Domain types
  ├── application/   # Use cases & Orchestration
  ├── infra/         # Persistence (Drizzle, Repos)
  └── components/    # UI (Presentation)
```

## UI

- Always prefer shadcn components.
- If a shadcn component is not available, install it: `bun x shadcn-svelte@latest add card -y`
- When installing, use the `-y` flag for autoinstall
- Always prefer shadcn styles, variants, etc; rather than manual tailwind classes

---

## Event Sourcing Pattern

This project uses **Event Sourcing** as the core persistence pattern. All state changes are captured as immutable events, and current state is derived by replaying events.

### Core Principles

1. **Events are Facts**: Once an event is written, it is **never deleted or modified**. Events represent historical facts that have occurred.
2. **State is Derived**: Current state is computed by replaying the event stream (left fold over events).
3. **Audit Trail by Default**: Every event stores `userId`, `occurredAt`, and optional `metadata` for complete traceability.
4. **Optimistic Concurrency**: Version numbers prevent conflicting concurrent updates.

### Architecture Components

#### 1. Domain Events (`src/lib/domain/events.ts`)

- **Responsibility**: Define all domain events as immutable types.
- **Structure**: All events extend `BaseEvent<T, P>` with:
  - `streamId`: The aggregate ID (e.g., account ID)
  - `streamType`: Type of aggregate ('account' | 'transaction')
  - `eventType`: Specific event name ('AccountCreated', 'TransactionUpdated', etc.)
  - `payload`: Event-specific data
  - `version`: Version within the stream for concurrency control
  - `occurredAt`: Timestamp when event occurred
  - `userId`: Who performed the action
  - `metadata`: Optional context (IP, user agent, etc.)
- **Rules**:
  - Use factory functions (e.g., `createAccountCreatedEvent`) to construct events
  - Events are pure data structures (no logic)
  - All event types must be in the `EventType` union

#### 2. Aggregates (`src/lib/domain/account-aggregate.ts`)

- **Responsibility**: Pure functions that project state from events.
- **Key Functions**:
  - `projectAccountState(events)`: Rebuild current state from full event stream
  - `projectAccountStateFromSnapshot(snapshot, events)`: Optimize by replaying from snapshot
  - `projectBalanceHistory(events)`: Get all balance changes over time
  - `getBalanceAtTime(events, asOf)`: Time-travel queries
  - `applyEvent(state, event)`: State machine that applies a single event
- **Rules**:
  - Must be **pure** (no side effects, no I/O)
  - No database queries (state is derived only from events in memory)
  - All business logic for state transitions lives here

#### 3. Event Store Repository (`src/lib/infra/repos/event-store.repo.ts`)

- **Responsibility**: Persist and query events from the database.
- **Key Operations**:
  - `append(event, options)`: Add single event to a stream
  - `appendMany(events)`: Add multiple events atomically
  - `getStream(streamId)`: Get all events for an aggregate
  - `getStreamVersion(streamId)`: Get current version for concurrency checks
  - `getStreamAsOf(streamId, date)`: Time-travel queries
  - `getAuditTrail(streamId)`: Full history with metadata
- **Concurrency Control**:
  - Pass `expectedVersion` to `append()` to prevent conflicts
  - Throws `ConcurrencyError` if version mismatch detected
- **Rules**:
  - Always use transactions for event writes
  - Events are append-only (no updates or deletes)
  - Version numbers must be sequential within a stream

#### 4. Projections / Read Models

- **Responsibility**: Denormalized views for fast queries (account table, transaction table).
- **Pattern**: CQRS (Command Query Responsibility Segregation)
  - **Write Side**: Events in event store (source of truth)
  - **Read Side**: Projections in tables (optimized for queries)
- **Synchronization**:
  - After appending events, update projections immediately
  - Use `eventStoreRepo.createAccountProjection()` / `updateAccountProjection()`
- **Rebuilding**: Projections can be deleted and rebuilt from events
  - See `src/lib/application/account/rebuild-projections.ts`
  - Useful for fixing inconsistencies or adding new projections

#### 5. Snapshots (Performance Optimization)

- **Purpose**: Avoid replaying thousands of events by caching state at intervals
- **Operations**:
  - `saveSnapshot(streamId, state, version)`: Store state checkpoint
  - `getLatestSnapshot(streamId)`: Retrieve most recent snapshot
  - `getStreamAfterVersion(streamId, version)`: Get events since snapshot
- **Strategy**: Create snapshots every N events (e.g., every 100 events)
- **Rules**: Snapshots are optional performance hints; events remain source of truth

### Application Layer Pattern

When creating/updating entities in Application layer:

```typescript
// 1. Load current state from events
const account = await getAccountState(accountId);

// 2. Get current version for concurrency check
const currentVersion = await getAccountVersion(accountId);

// 3. Validate business rules (use domain functions)
if (!canAcceptTransaction(account)) {
	throw error(404, 'Account not found or deleted');
}

// 4. Create domain event(s) with factory functions
const event = createTransactionCreatedEvent(
	accountId,
	userId,
	payload,
	currentVersion + 1 // Increment version
);

// 5. Append to event store with optimistic concurrency
await eventStoreRepo.append(event, {
	expectedVersion: currentVersion
});

// 6. Update read model (projection) for fast queries
await eventStoreRepo.updateAccountProjection(accountId, {
	currentBalance: newBalance
});

// 7. Return domain object (not event)
```

### Best Practices

1. **Always Increment Version**: New events must have `version = currentVersion + 1`
2. **Handle ConcurrencyError**: Catch and retry or return 409 Conflict
3. **Atomic Operations**: Events + projections should be consistent (use transactions if possible)
4. **Multi-Stream Transactions**: Be aware of limitations (see TODO in `create-transaction.ts`)
5. **Time Travel**: Use `getStreamAsOf()` for point-in-time queries (e.g., account balance on specific date)
6. **Audit Compliance**: Always pass `userId` to event factories for audit trails
7. **Projection Rebuild**: If projections become inconsistent, rebuild from events
8. **Times**: Use date-fnz libraries. Build for times with timezones.

### Known Limitations

- **Multi-Stream Atomicity**: Transfers affect two accounts but aren't fully atomic (see `create-transaction.ts` TODO)
- **SQLite Nested Transactions**: Limited support for nested transactions
- **Mitigation**: Optimistic concurrency + projection rebuild capability

### Anti-Patterns to Avoid

❌ **Don't modify events** after they're written  
❌ **Don't query database** in aggregate functions  
❌ **Don't skip version checks** when appending events  
❌ **Don't use projections** as source of truth (events are truth)  
❌ **Don't forget to update projections** after appending events  
❌ **Don't create events** without using factory functions

## UI

When working on UI:

- Use colors/variables/values from the shadcn theme at ~/src/routes/layout.css
- Add new variables only if needed
- Avoid inlining css, use the shadcn components & values

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Araozu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Araozu)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
