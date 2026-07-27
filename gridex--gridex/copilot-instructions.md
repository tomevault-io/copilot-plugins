## gridex

> Guide for AI agents (Claude, Codex, Cline, …) working on the macOS Swift codebase. Scope: everything under [macos/](.). Windows and Linux ports have their own conventions.

# AGENTS.md — Gridex macOS

Guide for AI agents (Claude, Codex, Cline, …) working on the macOS Swift codebase. Scope: everything under [macos/](.). Windows and Linux ports have their own conventions.

**Read order on every new task:**
1. This file.
2. [`CLAUDE.md`](../CLAUDE.md) at the repo root (project overview + behavioural guard-rails — `Think before coding`, `Surgical changes`, `Goal-driven execution`).
3. The closest existing adapter/service to what you're touching — copy its shape before inventing your own.

If anything in this file conflicts with `CLAUDE.md`, the root `CLAUDE.md` wins. This file is the **macOS-specific** layer on top.

---

## 0. Architecture invariants — non-negotiable

These are load-bearing. Don't violate them without an explicit user instruction.

- **5-layer Clean Architecture, dependencies point inward only.**
  `Core ← Domain ← Data ← Services ← Presentation`. Core has zero dependencies. Data implements Domain. Presentation never imports Data directly.
- **Single `DependencyContainer.shared`** at [`App/DependencyInjection/DependencyContainer.swift`](App/DependencyInjection/DependencyContainer.swift) is the only composition root. Don't `init` services elsewhere; ask the container.
- **Thread-safe services are `actor`.** `ConnectionManager`, `QueryEngine`, `MCPServer`, `SSHTunnelService`, `SchemaInspectorService`, `ProviderRegistry`, `BackupService` are all actors. Cross-actor types must be `Sendable`.
- **Domain models are plain Swift** (`struct`, `Codable`, `Sendable`). SwiftData `@Model` entities live only under [`Data/Persistence/Models/`](Data/Persistence/Models/) and round-trip via `toConfig()` / equivalent.
- **Errors are `GridexError`.** Never throw raw `NSError` or driver errors out of an adapter. Wrap with the matching case.
- **Credentials live in Keychain.** `KeychainService` (via `keychainService` on the container). Passwords are passed into adapters in-memory only; they are never on `ConnectionConfig`.
- **No global mutable state besides `DependencyContainer.shared` and `AppState`.** Per-window state goes on `AppState`; global on the container.

---

## 1. Reusable surface — use these before writing new code

Before introducing a new helper/protocol/model, scan this section. Hidden coupling notes in `[Coupling]` callouts.

### 1.1 Core protocols — the contracts

| Protocol | File | Conform when… |
|---|---|---|
| `DatabaseAdapter` | [Core/Protocols/Database/DatabaseAdapter.swift](Core/Protocols/Database/DatabaseAdapter.swift) | Adding a new database engine. ~50 methods covering connect/disconnect, execute (incl. parameterized), schema introspection, CRUD, pagination, transactions. |
| `SchemaInspectable` | [Core/Protocols/Database/SchemaInspectable.swift](Core/Protocols/Database/SchemaInspectable.swift) | The new adapter can introspect schemas. Powers AI context + ER diagram. |
| `LLMService` | [Core/Protocols/AI/LLMService.swift](Core/Protocols/AI/LLMService.swift) | Adding a new AI provider. Streaming-first, `AsyncThrowingStream`. |
| `AIContextProvider` | [Core/Protocols/AI/AIContextProvider.swift](Core/Protocols/AI/AIContextProvider.swift) | You almost certainly don't need a new one — reuse `aiContextEngine` from the container. |
| `MCPTool` | `Services/MCP/Tools/` | Adding a new MCP tool. Must declare permission tier + input schema. |

### 1.2 Core models — the value types every layer exchanges

| Model | File | Notes |
|---|---|---|
| `RowValue` | [Core/Models/Database/RowValue.swift](Core/Models/Database/RowValue.swift) | Unified cell type (null/string/int/double/bool/date/uuid/json/data/array). Always use; never leak driver-native types upward. |
| `ConnectionConfig` | [Core/Models/Database/ConnectionConfig.swift](Core/Models/Database/ConnectionConfig.swift) | Canonical connection spec. Includes `mongoOptions: [String:String]?` (URI query params), SSH tunnel config, mTLS cert paths. **Password is NOT a field** — passed separately. |
| `QueryResult` | [Core/Models/Query/QueryResult.swift](Core/Models/Query/QueryResult.swift) | Standard query output. `columns + rows + rowsAffected + executionTime + queryType`. |
| `QueryParameter` | [Core/Models/Query/QueryParameter.swift](Core/Models/Query/QueryParameter.swift) | Wraps `RowValue` for parameterized execution. Adapters bind via dialect-specific placeholders. |
| `FilterExpression` | [Core/Models/Query/FilterExpression.swift](Core/Models/Query/FilterExpression.swift) | Grid filter UI → SQL. `toSQL(dialect:)` builds the WHERE clause. **Use this; never concat WHERE strings by hand.** |
| `QuerySortDescriptor` | [Core/Models/Query/SortDescriptor.swift](Core/Models/Query/SortDescriptor.swift) | Grid sort state → ORDER BY. |
| `SchemaSnapshot`, `TableDescription`, `ColumnInfo`, `IndexInfo`, `ForeignKeyInfo`, `ViewInfo`, `TableInfo` | [Core/Models/Schema/](Core/Models/Schema/) | Schema introspection output. `TableDescription.toDDL(dialect:)` regenerates CREATE TABLE for export. |
| `AIContext`, `LLMMessage`, `ChatMessage` | [Core/Models/AI/AIModels.swift](Core/Models/AI/AIModels.swift) | AI chat data model (token-budgeted context, role messages, persisted history with inlined SQL/results). |
| `ProviderConfig` | [Core/Models/AI/ProviderConfig.swift](Core/Models/AI/ProviderConfig.swift) | Runtime AI provider config. API key is **not** a field — fetched from Keychain via `"ai.apikey.<uuid>"`. |

### 1.3 Errors

All in [`Core/Errors/GridexError.swift`](Core/Errors/GridexError.swift). Pick the closest case; don't add a new one without a real distinct meaning.

| Case | Throw it when |
|---|---|
| `connectionFailed(underlying:)` | Adapter `connect()` fails. Wrap the driver error. |
| `connectionTimeout` | Connection didn't respond within timeout. |
| `authenticationFailed` | Credentials rejected. |
| `sslRequired` | Server demands TLS but config disables it. |
| `queryExecutionFailed(String)` | Query syntax/runtime error. **Wrap driver errors via the adapter's formatter** (see 1.6). |
| `queryCancelled`, `queryTimeout` | Self-explanatory. |
| `sshConnectionFailed`, `sshAuthenticationFailed`, `sshTunnelFailed` | SSH tunnel failure paths. |
| `schemaLoadFailed(String)`, `tableNotFound(String)` | Schema introspection failed. |
| `aiProviderError`, `aiAPIKeyMissing`, `aiTokenLimitExceeded`, `aiStreamingError` | LLM provider failures. |
| `keychainError`, `persistenceError`, `importError`, `exportError` | I/O & persistence. |
| `unsupportedOperation(String)`, `internalError(String)` | Last resort. Prefer a more specific case if one fits. |

### 1.4 Key enums (verified — use exact case names)

| Enum | File | Cases |
|---|---|---|
| `DatabaseType` | [Core/Enums/DatabaseType.swift](Core/Enums/DatabaseType.swift) | `.sqlite, .postgresql, .mysql, .redis, .mongodb, .mssql, .clickhouse` |
| `SQLDialect` | [Core/Enums/SQLDialect.swift](Core/Enums/SQLDialect.swift) | one per `DatabaseType`; carries `quoteIdentifier`, `qualifiedIdentifier`, `parameterPlaceholder(_:)` |
| `SSLMode` | [Core/Enums/SSLMode.swift](Core/Enums/SSLMode.swift) | `.preferred, .disabled, .required, .verifyCA, .verifyIdentity` (libpq semantics) |
| `MCPConnectionMode` | [Core/Enums/MCPConnectionMode.swift](Core/Enums/MCPConnectionMode.swift) | `.locked, .readOnly, .readWrite` |
| `ProviderType` | [Core/Enums/ProviderType.swift](Core/Enums/ProviderType.swift) | `.anthropic, .openai, .gemini, .ollama` |
| `SSHAuthMethod`, `ColorTag`, `MCPPermissionTier`, `QueryType`, `FilterCombinator`, `FilterOperator`, `SortDirection` | [Core/Enums/](Core/Enums/) | Use directly in forms/UI/SQL builders. |

### 1.5 SQL generation helpers

`SQLDialect` is the single source of truth for dialect-specific SQL pieces:

```swift
let d = SQLDialect.postgresql
d.quoteIdentifier("user table")           // "user table"  (PG: ""), MySQL: ``, MSSQL: []
d.qualifiedIdentifier("users", schema: "auth")  // "auth"."users"
d.parameterPlaceholder(1)                 // PG: $1, MSSQL: @p1, MySQL/SQLite: ?
```

**Always** quote identifiers via `quoteIdentifier`. Never hardcode quote characters.

`QueryBuilder` at [`Services/QueryEngine/QueryBuilder.swift`](Services/QueryEngine/QueryBuilder.swift) is the fluent SELECT builder — use it for grid pagination/filtering instead of string concat. No DML builder yet; INSERT/UPDATE/DELETE go through adapter methods (`insertRow`, `updateRow`, `deleteRow`).

### 1.6 Adapter-shared helpers

Each adapter has its own private decoder/error formatter. Don't cross-call.

| Helper | Adapter | Visibility | Notes |
|---|---|---|---|
| `formatPostgresError(_:)` | PostgreSQL | `static`, callable | Extracts `message/detail/hint/position/sqlState` from `PSQLError`. Use it whenever you re-throw a PG error. |
| `wrap(mongoError:)` | MongoDB | `private static` | Extracts `errmsg/codeName` from `MongoServerError`. Extract if you need it elsewhere. |
| `decodeCell(_:dataType:)` | every adapter | private | Driver-specific. Output is `RowValue`. |
| `executeParameterized(sql:params:)` | PostgreSQL | private | Bound `$1, $2, …` execution. Use in any new metadata query that takes user input. |
| `inlineValue(_:)` / `buildInlineSQL(...)` | PostgreSQL | private | Last-resort SQL inlining for paths that can't bind. Prefer parameterised. |

`[Coupling]` These helpers are file-private to the adapter. **Don't** call across adapters; if you need shared error formatting/decoding, extract first.

`disableAutoSubstitutions()` on `NSTextView` ([`Core/Extensions/NSTextView+PlainText.swift`](Core/Extensions/NSTextView+PlainText.swift)) is the **shared** helper for any text input that must remain plain ASCII (SQL, JSON). See incident in §3.

### 1.7 Services (`DependencyContainer` surface)

Access these via `DependencyContainer.shared.<name>`:

| Property | Type | Use for |
|---|---|---|
| `modelContainer` | `ModelContainer` | SwiftData root. Repositories own this; you rarely touch it. |
| `keychainService` | `KeychainServiceProtocol` | All credential I/O. |
| `connectionRepository` | `ConnectionRepository` | Saved-connection CRUD. |
| `queryHistoryRepository` | `QueryHistoryRepository` | Persisted query history (favourites, search). |
| `llmProviderRepository` | `LLMProviderRepository` | AI provider configs. |
| `connectionManager` | `ConnectionManager` (actor) | `connect/disconnect/active connection` lifecycle. SSH tunnel routed internally. |
| `queryEngine` | `QueryEngine` (actor) | Run queries; auto-logs to history. |
| `schemaInspector` | `SchemaInspectorService` (actor) | Cached `SchemaSnapshot`s per connection. |
| `aiContextEngine` | `AIContextEngine` | Token-budgeted prompts from a SchemaSnapshot. |
| `mcpServer` | `MCPServer` (actor) | start/stop, per-connection mode. |
| `providerRegistry` | `ProviderRegistry` (actor) | resolve LLM by name/type. |

Bootstrap once in `WindowRoot`: `await container.bootstrapMCPServer()` + `await container.bootstrapProviderRegistry()`.

### 1.8 Persistence: entity ↔ config round-trip

Pattern: every persisted SwiftData entity exposes a `toConfig()` / `toEntry()` method. Business logic uses Domain models; entities stay inside the Data layer.

| Entity | File | Round-trip |
|---|---|---|
| `SavedConnectionEntity` | [Data/Persistence/Models/SavedConnectionEntity.swift](Data/Persistence/Models/SavedConnectionEntity.swift) | `toConfig() -> ConnectionConfig`. **All new fields must be `Optional` for SwiftData lightweight migration.** |
| `QueryHistoryEntity` | [Data/Persistence/Models/QueryHistoryEntity.swift](Data/Persistence/Models/QueryHistoryEntity.swift) | `toEntry() -> QueryHistoryEntry` |
| `LLMProviderEntity` | [Data/Persistence/Models/LLMProviderEntity.swift](Data/Persistence/Models/LLMProviderEntity.swift) | `toConfig() -> ProviderConfig` |

Repositories live under [`Domain/Repositories/`](Domain/Repositories/) (protocols) and [`Data/Persistence/Repositories/`](Data/Persistence/Repositories/) (impls).

### 1.9 DI + per-window state

- [`DependencyContainer`](App/DependencyInjection/DependencyContainer.swift) — global singleton. Composes everything.
- [`AppState`](App/State/AppState.swift) — per-window `@MainActor ObservableObject`. Holds active connection, tabs, sidebar/AI panel visibility, pending edits, cached grid/editor state. **Hook into existing `@Published` props** (`activeConnectionId`, `tabs`, `sidebarVisible`, …) — don't add a parallel state store.
- `AppState.active` returns the currently-focused window's state for use in command menu actions.

`[Coupling]` Don't add `@Published` properties that nobody observes — they bloat the type and confuse future readers (see §3, the `showSettings` incident).

### 1.10 Reusable SwiftUI / AppKit components

| Component | File | Purpose |
|---|---|---|
| `TPRow<Content>` | [Presentation/Views/ConnectionForm/ConnectionFormView.swift](Presentation/Views/ConnectionForm/ConnectionFormView.swift) (line ~602) | Form row: label on left, content on right, fixed `labelWidth` (default 110). |
| `TPTextField` | same file (line ~635) | `NSViewRepresentable` text field with `focusRingType = .none`. Has `isSecure` for passwords. |
| `TPButton` | same file (line ~620) | Flat, `maxWidth` button. |
| `TPFileButton` | same file (line ~671) | Opens `NSOpenPanel`, binds selected path. SSL keys, SQLite picker, etc. |
| `ClickableModifier` | [Presentation/Views/Shared/ClickableModifier.swift](Presentation/Views/Shared/ClickableModifier.swift) | Hover + click decoration on arbitrary views. |
| `PlainJSONTextEditor`, `SQLTextView` | Presentation/Views/{MongoDB,QueryEditor}/ | `NSTextView` wrappers that call `disableAutoSubstitutions()`. **Use these instead of SwiftUI `TextEditor` for SQL/JSON input** (see §3 smart-quotes incident). |

`[Coupling]` `TP*` components are currently **defined inline** in `ConnectionFormView.swift` — not exported. If you need them in a new form, extract to `Presentation/Views/Shared/` first; don't duplicate.

### 1.11 Notification.Name event bus

Cross-cutting events flow through `NotificationCenter.default`. Constants are extension members on `Notification.Name` defined in [`App/Lifecycle/GridexApp.swift`](App/Lifecycle/GridexApp.swift):

`.executeQuery`, `.executeSelection`, `.explainQuery`, `.formatSQL`, `.deleteSelectedRows`, `.commitChanges`, `.reloadData`, `.toggleFilterBar`.

Adding a new app-wide command? Add it to that extension and post from a `CommandMenu`/`CommandGroup` in `GridexApp.swift`. Receivers should observe in `init` and tear down on deinit.

---

## 2. DO

Concrete, codified patterns. Each one is an explicit "yes" — the inverse of a §3 incident.

1. **Add new `ConnectionConfig` fields as `Optional?` with a sensible fallback at the read site.**
   New SwiftData columns must also be optional. Old rows decoded without the field must keep working. `toConfig()` should `flatMap`/`??` to back-compat.

2. **Use `executeParameterized` for any metadata query that takes user input.**
   PostgreSQL adapter has `private executeParameterized(sql:params:)`. Write similar bound execution paths in new adapters.

3. **Wrap driver errors via the adapter's formatter before throwing `GridexError.queryExecutionFailed`.**
   `formatPostgresError`, `wrap(mongoError:)`. Bare `error.localizedDescription` loses the SQLSTATE / `errmsg` / detail / hint that users actually need.

4. **Persist the full UI state, not its boolean shadow.**
   If the picker has 5 modes, store the 5-mode enum, not `isOn: Bool`. The boolean form ignores user intent (SSL `verifyCA` ≠ "on", see §3).

5. **Reach for the existing actor service.**
   New connection lifecycle? Go through `ConnectionManager`, not a fresh `Adapter()`. New schema fetch? `SchemaInspectorService`. Don't bypass the cache.

6. **Conform to `DatabaseAdapter` exhaustively.**
   The grid, ER diagram, MCP tools, AI context all assume the full surface. A half-implemented adapter breaks features silently three layers up.

7. **Use the parent role's catalog, not `information_schema.*`, for "list everything" queries.**
   `pg_namespace` / `pg_class` / `pg_proc` are readable by all roles. `information_schema` is privilege-filtered (incident #34).

8. **Use `@FocusedObject` for cross-window `AppState` in command menus.**
   See `currentAppState` in `GridexApp.swift`. Falls back to `AppState.active` when no window is focused.

9. **Validate identifiers from external clients (MCP).**
   `MCPIdentifierValidator` ASCII-allowlists table/schema/column names. Use it for any new write tool.

10. **Bump `Info.plist` version in a separate `chore(release): bump version to X.Y.Z` commit on the same branch.**
    Convention is `CFBundleShortVersionString` + `CFBundleVersion` (integer) advanced together. macOS uses its own series independent of Windows/Linux.

11. **Conventional Commits with scope.** `feat(postgres): …`, `fix(mcp): …`, `chore(release): …`, `docs(readme): …`. Keep subjects ≤ 72 chars; explain the *why* in the body.

12. **One concern per PR.** A schema fix doesn't need a refactor. A version bump is its own commit, not folded into the feature commit.

---

## 3. DON'T (with concrete incident references)

Each item maps to a real bug we shipped and then fixed. The commit hash is the receipt — read it before re-introducing the pattern.

1. **Don't query `information_schema.*` for "show me everything" lists.** It's privilege-filtered per SQL standard. Use `pg_namespace` / `pg_class` / `pg_proc`. → `d6e51b3` *(fix #34, only `public` schema visible on Supabase)*.

2. **Don't collapse a multi-state UI enum into a `Bool` when persisting.** SSL mode picker had 5 states; we stored only `sslEnabled: Bool`, so `REQUIRED` / `VERIFY_CA` / `VERIFY_IDENTITY` all degraded to `.prefer(tls)`. → `6a4aad0`.

3. **Don't add a custom `CommandGroup(replacing: .appSettings)` while a SwiftUI `Settings { … }` scene exists.** They both render menu items, you get a duplicate "Settings…". → `813f365` *(fix #26)*.

4. **Don't use SwiftUI `TextEditor` for SQL or JSON input.** macOS auto-substitution turns `"` into `“ ”` and breaks `JSONSerialization` / SQL parsers with an opaque error. Use an `NSViewRepresentable` calling `disableAutoSubstitutions()`. → `0c5644d`.

5. **Don't drop URI query parameters when parsing connection strings.** A pasted `mongodb://...?authSource=admin&replicaSet=rs0` must round-trip through `ConnectionConfig.mongoOptions` to the adapter's `buildURI`. → `9466c55`.

6. **Don't ship `@Published` state that no one observes.** It signals intent that doesn't exist and traps the next reader. The duplicated Settings menu's button wrote to `AppState.showSettings`, observed by nothing. → `813f365`.

7. **Don't break the `Codable` contract on persisted value types.** Adding a non-optional field fails decoding for existing rows. Make it `Optional` and resolve at the read site. → `6a4aad0` *(sslMode added as `SSLMode?`)*.

8. **Don't assume IPv4 reachability.** Supabase free-tier direct URLs (`db.<ref>.supabase.co:5432`) have only AAAA records; an IPv4-only network never connects. On connection failure, surface an actionable hint (try `ipv6OnlyHint(host:)`-style detection) instead of returning a generic timeout. → `6a4aad0`.

9. **Don't surface raw `PSQLError` / `MongoError` to users.** Wrap with the adapter's formatter; otherwise they see `"operation couldn't be completed (… error 1.)"` instead of the actual `errmsg`. → `7042ecb`, `6a4aad0`.

10. **Don't hand-roll WHERE / ORDER BY strings.** Use `FilterExpression.toSQL(dialect:)` and `QuerySortDescriptor.toSQL(dialect:)`. Manual construction loses dialect-specific quoting and value escaping.

11. **Don't introduce driver-native types above the adapter.** `Postgres.Jsonb`, `MongoKitten.Document`, `MySQL.Variant` stop at the adapter boundary. Decode to `RowValue` (or a Core model) before returning.

12. **Don't bypass `DependencyContainer`.** Construct services through the container, not direct `init`. Tests can swap the container; ad-hoc `init`s can't.

13. **Don't `--no-verify` or `--no-gpg-sign` to push past a hook.** Investigate the hook failure. Same for `git push --force` on shared branches.

14. **Don't merge unrelated work into an open PR.** When the user says "fix X", don't fold "while I'm here, also …" into the same commit/branch. Open a second branch.

---

## 4. Workflow

### Branch + commit flow
- Branch off `main`. Naming: `fix/<issue>-<short>`, `feat/<area>-<short>`, `chore/…`, `docs/…`.
- Conventional Commits in subject. Body answers *why*. Reference issue numbers (`#34`).
- Version bump in `Info.plist` is a **separate commit** (`chore(release): bump version to X.Y.Z`) on the same branch as the change that justifies it.
- Open the PR with a clear title (≤ 70 chars), Summary + Test Plan in the body. Caveats and migration notes go in the Test Plan.

### Build + verify
- `swift build` from repo root. Must pass with no new warnings before commit.
- `./scripts/build-app.sh` for the `.app` bundle.
- `./scripts/release.sh` (or `release-all.sh`) for signed/notarized DMG.
- Postgres-specific changes: spin up a local PG via `brew services start postgresql@16` (or initdb to a temp dir on a non-default port) and reproduce the user's failure mode before claiming a fix. The session that produced incident #34 verified by reproducing Supabase's role layout locally.

### When the user reports a connection bug
Order of suspicion (from incident base rate):
1. Network layer (IPv6-only host, firewall, DNS).
2. Auth (wrong user/role, missing `authSource` for Mongo).
3. TLS mode mismatch.
4. Driver-level edge case (URI parsing, parameter binding).

Resolve DNS for the user's host before digging into TLS. If only AAAA exists and the user's network is v4-only, no SSL change will help — direct them to a pooler / IPv4 endpoint.

---

## 5. When you're stuck

In order, before asking the user:

1. **Read the closest sibling.** Adding a Postgres feature? `MySQLAdapter.swift` already solved the same shape. Adding an AI provider? Compare `AnthropicService.swift` ↔ `OpenAIService.swift`.
2. **`git log --oneline -- <file>`** on the file you're editing. The last 5 commits often cover the gotchas.
3. **`git log -G "<symbol>"`** to see when a symbol was introduced or refactored — the message usually explains the constraint.
4. **Reproduce locally.** A Postgres bug? `initdb` + `pg_ctl` + `psql`. A SwiftUI bug? Run the dev build and click through.

If after that you still have multiple plausible interpretations, **stop and ask the user**. Per `CLAUDE.md` rule 1: "If multiple interpretations exist, present them — don't pick silently." Better one extra round-trip than 200 lines on the wrong premise.

---
> Source: [gridex/gridex](https://github.com/gridex/gridex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
