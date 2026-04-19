## groodo

> Context for the storage, API clients, and utilities layer — auto-included when working on lib files


# lib/ — Storage, API Clients & Utilities

## Overview

Contains all data access logic, API communication, and shared utility functions. The core pattern is a **strategy-based storage layer** that switches between localStorage (guest) and API (authenticated) without changing consuming code.

## Architecture

```
Feature Code
    ↓
Abstract Base Client (interface)
    ├── LocalStorage*Client  (guest users)
    └── GroodoApi*Client     (authenticated users)
            ↓
        Low-level API modules (HTTP calls)
```

## Storage Clients

### Base Classes (interfaces)
- `taskStorageClient.js` — Defines: `listTasks`, `createTask`, `updateTask`, `deleteTask`
- `documentStorageClient.js` — Defines: `listDocuments`, `getDocument`, `createDocument`, `updateDocument`, `deleteDocument`
- `projectStorageClient.js` — Defines: `listProjects`, `getProject`, `createProject`, `updateProject`, `deleteProject`, `getProjectTasks`

### Concrete Implementations (`clients/`)
- `LocalStorageClient` / `LocalStorageDocumentsClient` / `LocalStorageProjectsClient` — Persist to browser localStorage
- `GroodoApiClient` / `GroodoDocumentsClient` / `GroodoProjectsClient` — Persist via REST API, include data transformation between API and internal formats

### Low-level API Modules
- `groodoApiTasksClient.js` — HTTP calls for tasks
- `groodoApiDocumentsClient.js` — HTTP calls for documents
- `groodoApiProjectsClient.js` — HTTP calls for projects

Each handles auth token injection and sliding expiration via `touchCookie`.

## Auth

- `authClient.js` — `signIn`, `signUp`, `signOut`, `getMe`, `getToken` functions
- `cookies.js` — Cookie utilities with sliding expiration for auth tokens

## Utilities

- `config.js` — API host configuration, auto-detects dev vs production
- `date.js` — Date utilities for the **Sunday–Thursday work week**: week bounds, navigation, formatting, date keys
- `storage.js` — localStorage persistence with **data versioning and migration** support (used by board tasks). `saveState` and `loadState` are **async functions** (return Promises) so that consuming code follows the same contract whether the underlying storage is localStorage or a remote API. No artificial delay is needed — the highlight animation is now decoupled from persistence timing via `RECONCILE_TASK`. Migration helpers (`_saveStateSync`, `migrateData`, `migrateFromVersion1`) remain synchronous and are used internally within `loadState`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gregory-chris) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
