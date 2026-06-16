## inpx-library-server

> This repository is a self-hosted ebook library server.

# INPX Library Server — Agent Instructions

## Project Overview

This repository is a self-hosted ebook library server.

Repository type:

* single-package Node.js application
* NOT a monorepo
* NOT a workspace-based repository

Main stack:

* Node.js
* Express
* SQLite (better-sqlite3)
* ESM modules
* Vanilla frontend JavaScript
* Docker

Primary goals:

1. Stability
2. Backward compatibility
3. Low resource usage
4. Performance
5. Maintainability

---

## Working Directory Rules

Repository root is the primary working directory.

All commands must be executed from repository root unless explicitly specified otherwise.

Never:

* run npm commands from nested directories
* assume monorepo structure
* invent alternative workflows

Before executing commands:

1. verify current working directory
2. verify package.json exists
3. verify command exists in package.json scripts

---

## Main Commands

Install dependencies:

```
npm install
```

Development:

```
npm run dev
```

Production:

```
npm start
```

Tests:

```
npm test
```

Do not bypass existing npm scripts unless necessary.

Prefer existing workflows and scripts.

---

## Architecture Principles

The codebase prioritizes:

* simplicity
* predictability
* maintainability
* low memory usage

Prefer:

* minimal diffs
* incremental changes
* explicit code
* existing project patterns
* reusable helpers

Avoid:

* mass refactoring
* unnecessary abstractions
* large rewrites
* formatting-only changes

Never introduce:

* React
* Vue
* Svelte
* TypeScript
* ORM
* enterprise architecture patterns
* dependency injection
* CQRS/event sourcing

---

## Compatibility Requirements

Backward compatibility is critical.

Never silently break:

* API responses
* database compatibility
* Docker compatibility
* OPDS compatibility
* configuration formats
* existing environment variables

Prefer additive changes over breaking changes.

---

## Performance Constraints

Target environments include:

* NAS devices
* Raspberry Pi
* Docker containers
* low-memory systems

Libraries may contain:

* hundreds of thousands of books
* huge archives
* slow disks
* limited RAM

Performance and memory efficiency are important.

Avoid:

* loading huge datasets into memory
* full archive extraction
* memory-heavy scans
* unnecessary buffering

Prefer:

* streaming
* pagination
* batching
* indexed SQL queries
* incremental processing

---

## Database Rules

Database:

* SQLite
* better-sqlite3

Use:

* prepared statements
* existing DB helpers
* indexed queries

Avoid:

* ORM
* query builders
* unnecessary schema rewrites
* loading large result sets fully into memory

Schema changes must:

* be additive
* preserve old databases
* include migration logic

---

## Filesystem Safety

Never trust filesystem paths.

Always:

* normalize paths
* validate paths
* prevent path traversal
* validate archive extraction paths

Never:

* overwrite files silently
* delete user files automatically
* assume filesystem case sensitivity

---

## Security Rules

Always:

* validate request input
* sanitize filenames
* verify permissions
* preserve auth middleware

Never:

* log passwords
* log tokens
* expose SMTP credentials
* trust request parameters

---

## OPDS Compatibility

Compatibility with these clients is critical:

* KOReader
* FBReader

Do not break:

* feed structure
* authentication behavior
* MIME types
* pagination semantics
* existing feed URLs

All OPDS responses must produce valid XML.

---

## Development Workflow

Before modifying code:

1. analyze surrounding files
2. reuse existing patterns
3. reuse existing helpers
4. implement smallest safe change

When fixing bugs:

* identify root cause first
* avoid unrelated refactoring
* preserve existing behavior

When adding features:

* preserve compatibility
* estimate performance impact
* estimate memory impact
* preserve Docker compatibility

---

## Dependency Rules

Before adding dependencies:

* check existing project utilities
* check Node.js built-ins
* estimate maintenance risk
* estimate package size

Never add dependencies for:

* trivial utilities
* simple formatting
* one-time wrappers

Prefer:

* zero-dependency solutions
* mature stable libraries
* already-used packages

---

## Testing Expectations

Before finalizing changes:

* verify affected routes
* verify npm scripts still work
* verify Docker startup
* verify DB compatibility
* verify memory impact

For performance-sensitive changes:

* consider huge library scenarios
* consider low RAM environments

---
> Source: [Habsaec/inpx-library-server](https://github.com/Habsaec/inpx-library-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
