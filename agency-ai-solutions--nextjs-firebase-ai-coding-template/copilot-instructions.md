## adr

> Architectural Decision Records


# Architecture Decision Log

<!--
ADR_AGENT_PROTOCOL v1.0

You (the agent) manage this file as the single source of truth for all ADRs.

INVARIANTS
- Keep this exact file structure and headings.
- All ADR entries use H2 headings: "## ADR-XXXX — <Title>" (4-digit zero-padded ID).
- Allowed Status values: Proposed | Accepted | Superseded
- Date format: YYYY-MM-DD
- New entries must be appended to the END of the file.
- The Index table between the INDEX markers must always reflect the latest state and be sorted by ID desc (newest on top).
- Each ADR MUST contain: Date, Status, Owner, Context, Decision, Consequences.
- Each ADR must include an explicit anchor `<a id="adr-XXXX"></a>` so links remain stable.

HOW TO ADD A NEW ADR
1) Read the whole file.
2) Compute next ID:
   - Scan for headings matching: ^## ADR-(\d{4}) — .+$
   - next_id = (max captured number) + 1, left-pad to 4 digits.
3) Create a new ADR section using the “New ADR Entry Template” below.
   - Place it AFTER the last ADR section in the file.
   - Add an `<a id="adr-XXXX"></a>` line immediately below the heading.
4) Update the Index (between the INDEX markers):
   - Insert/replace the row for this ADR keeping the table sorted by ID descending.
   - Title in the Index MUST link to the anchor: [<Title>](#adr-XXXX)
   - If this ADR supersedes another: set “Supersedes” in this row, and update that older ADR:
       a) Change its Status to “Superseded”
       b) Add “Superseded by: ADR-XXXX” in its Consequences block
       c) Update the older ADR’s Index row “Superseded by” column to ADR-XXXX
5) Validate before saving:
   - Exactly one heading exists for ADR-XXXX
   - All required fields are present and non-empty
   - Index contains a row for ADR-XXXX and remains properly sorted
6) Concurrency resolution:
   - If a merge conflict or duplicate ID is detected after reading: recompute next_id from the current file state, rename your heading, anchor, and Index row accordingly, and retry once.

COMMIT MESSAGE SUGGESTION
- "ADR-XXXX: <Short Title> — <Status>"

END ADR_AGENT_PROTOCOL
-->

## Index

<!-- BEGIN:ADR_INDEX -->

| ID   | Title                                                        | Date       | Status   | Supersedes | Superseded by |
| ---- | ------------------------------------------------------------ | ---------- | -------- | ---------- | ------------- |
| 0004 | [Event-Driven Broker Architecture with Firestore](#adr-0004) | 2025-01-27 | Accepted | —          | —             |
| 0002 | [Monorepo Structure with Frontend and Backend](#adr-0002)    | 2025-01-27 | Accepted | —          | —             |
| 0001 | [Full-Stack Firebase Template Architecture](#adr-0001)       | 2025-01-27 | Accepted | —          | —             |

<!-- END:ADR_INDEX -->

---

## New ADR Entry Template (copy for each new decision)

> Replace placeholders, keep section headers. Keep prose concise.

```

## ADR-XXXX — \<Short, specific title>

<a id="adr-XXXX"></a>
**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Superseded
**Owner**: <Name>

### Context

<1–3 sentences: what changed or what forces drive this decision now>

### Alternatives

<Quick bullet list of alternatives considered, and why they were rejected.>

### Decision

\<Single clear decision in active voice; make it testable/verifiable>

### Consequences

* **Pros**: \<benefit 1>, \<benefit 2>
* **Cons / risks**: \<cost 1>, \<risk 1>
* **Supersedes**: ADR-NNNN (if any)
* **Superseded by**: ADR-MMMM (filled later if replaced)

### (Optional) Compliance / Verification

\<How we’ll check this is honored: tests, checks, fitness functions, runbooks>

```

---

## ADR-0001 — Full-Stack Firebase Template Architecture

<a id="adr-0001"></a>
**Date**: 2025-01-27 `
**Status**: Accepted  
**Owner**: AI Agent

### Context

Modern web applications require both frontend and backend components with real-time data synchronization. Firebase provides a comprehensive platform for building full-stack applications, but setting up the architecture with best practices requires significant boilerplate code.

### Alternatives

- **Separate repositories**: Frontend and backend in different repos, harder to coordinate changes
- **Backend-only template**: Covers only server-side concerns, developers must handle frontend integration
- **Frontend-only template**: Limited to client-side Firebase SDK, missing serverless functions capabilities
- **Full-stack monorepo template**: Single template with both Next.js frontend and Python Firebase Functions

### Decision

Create a comprehensive full-stack Firebase template with:

- Next.js 14 frontend with TypeScript and Material-UI
- Python Firebase Functions backend with broker architecture pattern
- Shared Firebase project configuration
- Integrated testing strategy for both frontend and backend
- Single repository with clear separation of concerns

### Consequences

- **Pros**: Complete starting point for Firebase projects, coordinated development, shared configuration
- **Cons / risks**: More complex initial setup, larger template size, requires knowledge of both frontend and backend
- **Supersedes**: —
- **Superseded by**: —

### Compliance / Verification

Template includes working examples for both frontend and backend. Firebase configuration shared between components. Documentation covers full development workflow.

---

## ADR-0002 — Monorepo Structure with Frontend and Backend

<a id="adr-0002"></a>
**Date**: 2025-01-27  
**Status**: Accepted  
**Owner**: AI Agent

### Context

Full-stack Firebase applications need coordinated development between frontend (Next.js) and backend (Firebase Functions). The question was how to organize the code structure to support both independent development and shared configuration.

### Alternatives

- **Separate repositories**: Independent versioning but coordination overhead and duplicate configuration
- **Nested backend in frontend**: Simple structure but mixes concerns and complicates deployment
- **Side-by-side monorepo**: Clear separation with shared root configuration

### Decision

Implement side-by-side monorepo structure:

- `/front/` - Next.js frontend application with its own package.json and dependencies
- `/back/` - Python Firebase Functions backend with its own requirements.txt and structure
- Root-level Firebase configuration (`firebase.json`, `firestore.rules`, `storage.rules`)
- Shared documentation in root README.md with component-specific READMEs

### Consequences

- **Pros**: Clear separation of concerns, independent tooling, shared Firebase configuration, coordinated documentation
- **Cons / risks**: Slightly more complex initial setup, need to manage two different development environments
- **Supersedes**: —
- **Superseded by**: —

### Compliance / Verification

Directory structure clearly separates frontend and backend. Firebase CLI recognizes backend functions source in configuration. Both components can be developed independently.

---

## ADR-0004 — Event-Driven Broker Architecture with Firestore

<a id="adr-0004"></a>
**Date**: 2025-01-27  
**Status**: Accepted  
**Owner**: AI Agent

### Context

Modern full-stack applications require consistent data state between frontend and backend components. Traditional request-response patterns create tight coupling and require complex state synchronization logic. Real-time applications need immediate UI updates when data changes, regardless of the source of the change.

### Alternatives

- **Direct API communication**: Backend returns data directly to frontend, requires manual state management
- **Event streaming with external broker**: Use services like Redis Pub/Sub or RabbitMQ, adds infrastructure complexity
- **WebSocket connections**: Real-time but requires connection management and doesn't persist data
- **Firestore as event-driven broker**: Leverages built-in real-time capabilities and acts as single source of truth

### Decision

Implement event-driven broker architecture where Firestore serves as both the data store and event broker:

- All data mutations flow through Firestore exclusively
- Backend functions save data to Firestore without returning responses to frontend
- Frontend subscribes to Firestore collections/documents using hooks for real-time updates
- Firestore acts as the single source of truth for both backend and frontend
- UI updates automatically through Firestore real-time listeners

### Consequences

- **Pros**: Eliminates data synchronization issues, automatic real-time updates, reduced coupling between frontend and backend, simplified state management, leverages Firebase's built-in capabilities
- **Cons / risks**: Increased Firestore read operations, requires proper security rules design, potential data consistency challenges with complex operations, network dependency for all data access
- **Supersedes**: —
- **Superseded by**: —

### Compliance / Verification

Backend functions must only perform Firestore writes without returning data responses. Frontend components must use Firestore hooks (useFirestore, real-time listeners) for all data access. No direct API data responses to frontend. All data mutations trigger UI updates through Firestore change events.

---

---
> Source: [agency-ai-solutions/nextjs-firebase-ai-coding-template](https://github.com/agency-ai-solutions/nextjs-firebase-ai-coding-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
