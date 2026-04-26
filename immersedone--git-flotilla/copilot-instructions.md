## git-flotilla

> > This file is the primary instruction set for Claude Code when working on this project.

# CLAUDE.md — Git Flotilla

> This file is the primary instruction set for Claude Code when working on this project.
> Read this entire file before making any changes. Re-read relevant sections before each task.

---

## Project Identity

**Git Flotilla** is a cross-platform desktop GUI (Windows, macOS, Linux) for managing multiple GitHub and GitLab repositories at scale. It is built for DevOps engineers, agencies, and teams who manage many repos and need to perform batch operations, security patching, dependency auditing, and repo health monitoring from a single interface.

**Stack:** Rust · Tauri v2 · TypeScript · Vue 3 (Composition API) · Pinia · Tailwind CSS v4 · Vite

**Repo:** `git-flotilla`
**Crate name:** `git-flotilla`
**Binary name:** `git-flotilla`

---

## Tech Stack Rules

### Rust / Tauri Backend
- All filesystem access, GitHub/GitLab API calls, Git operations, and CVE scraping happen in **Rust only** — never in the frontend
- Use `tauri::command` to expose backend functions to the frontend
- Use `tokio` for async — all I/O must be non-blocking
- Use `reqwest` for HTTP (GitHub API, GitLab API, CVE feeds)
- Use `serde` / `serde_json` for all serialisation
- Use `sqlx` with **SQLite** (via Tauri's app data directory) for local persistence of scan results, repo configs, CVE cache, and audit logs
- Use `git2` (libgit2 bindings) for local Git operations where needed
- Error handling: always use `Result<T, E>` — no `unwrap()` in production paths; propagate errors to frontend via Tauri's error serialisation
- Secrets (tokens, credentials) must be stored using `keyring` crate (OS keychain), never in plaintext config files

### Vue 3 / TypeScript Frontend
- Use **Composition API** exclusively (`<script setup lang="ts">`) — no Options API
- Use **Pinia** for all global state — one store per domain (repos, scans, CVEs, settings, auth, operations)
- Use **Vue Router** for navigation between views
- All Tauri backend calls go through a typed wrapper layer in `src/services/` — never call `invoke` directly in components
- Use **Tailwind CSS v4** utility classes — no custom CSS except for design tokens in `tailwind.config.ts`
- Component library: build internal components in `src/components/ui/` before reaching for external libraries
- Forms: use `vee-validate` + `zod` for all form validation
- Use `@tanstack/vue-query` for async state management of backend calls (loading, error, refetch)

### General
- TypeScript strict mode is **always on** — no `any` types, no `@ts-ignore`
- All new features require a corresponding entry in `PLANNING.md` marked as `[implemented]`
- Run `cargo clippy` and `cargo fmt` before committing Rust code
- Run `pnpm lint` and `pnpm typecheck` before committing frontend code

---

## Project Structure

```
git-flotilla/
├── src-tauri/                  # Rust / Tauri backend
│   ├── src/
│   │   ├── main.rs             # Tauri app entry point
│   │   ├── lib.rs              # Library root
│   │   ├── commands/           # Tauri command handlers (one file per domain)
│   │   │   ├── auth.rs         # GitHub/GitLab auth commands
│   │   │   ├── repos.rs        # Repo discovery, listing, repo list management
│   │   │   ├── scan.rs         # Repo scanning (versions, package managers, files)
│   │   │   ├── packages.rs     # Dependency matrix, overlap analysis, changelog
│   │   │   ├── cve.rs          # CVE scraping, alerts, watchlist, incident timeline
│   │   │   ├── operations.rs   # Batch file updates, commits, PRs, pin/bump
│   │   │   ├── merge_queue.rs  # PR merge queue, batch merge
│   │   │   ├── scripts.rs      # Custom script runner
│   │   │   ├── compliance.rs   # Secret scanning, licence compliance, branch protection
│   │   │   └── settings.rs     # App settings
│   │   ├── models/             # Serde structs matching frontend types
│   │   ├── db/                 # SQLx queries and migrations
│   │   │   └── migrations/     # SQL migration files
│   │   ├── services/           # Business logic (not Tauri commands)
│   │   │   ├── github.rs       # GitHub API client
│   │   │   ├── gitlab.rs       # GitLab API client
│   │   │   ├── cve_scraper.rs  # CVE feed polling service
│   │   │   ├── scanner.rs      # Repo scanning logic
│   │   │   ├── patcher.rs      # Package pin/bump logic, monorepo-aware patching
│   │   │   ├── changelog.rs    # Changelog fetching from GitHub Releases / CHANGELOG.md
│   │   │   ├── rate_limiter.rs # API rate limit tracking and auto-pause
│   │   │   ├── script_runner.rs # Custom script execution across repos
│   │   │   ├── secret_scanner.rs # Secret exposure detection (regex patterns)
│   │   │   ├── licence_checker.rs # Transitive dependency licence analysis
│   │   │   └── scheduler.rs    # Background job scheduler (hourly CVE, scheduled scans)
│   │   └── error.rs            # Unified error type
│   ├── Cargo.toml
│   └── tauri.conf.json
│
├── src/                        # Vue 3 / TypeScript frontend
│   ├── main.ts
│   ├── App.vue
│   ├── router/
│   │   └── index.ts
│   ├── stores/                 # Pinia stores
│   │   ├── auth.ts
│   │   ├── repos.ts
│   │   ├── repoLists.ts
│   │   ├── scans.ts
│   │   ├── packages.ts
│   │   ├── cve.ts
│   │   ├── operations.ts
│   │   ├── mergeQueue.ts
│   │   ├── scripts.ts
│   │   ├── compliance.ts
│   │   └── settings.ts
│   ├── services/               # Typed Tauri invoke wrappers
│   │   ├── auth.ts
│   │   ├── repos.ts
│   │   ├── scan.ts
│   │   ├── packages.ts
│   │   ├── cve.ts
│   │   ├── operations.ts
│   │   ├── mergeQueue.ts
│   │   ├── scripts.ts
│   │   └── compliance.ts
│   ├── components/
│   │   ├── ui/                 # Base design system components
│   │   │   ├── Button.vue
│   │   │   ├── Input.vue
│   │   │   ├── Badge.vue
│   │   │   ├── Card.vue
│   │   │   ├── Modal.vue
│   │   │   ├── Table.vue
│   │   │   ├── Tooltip.vue
│   │   │   └── CommandPalette.vue
│   │   ├── repos/              # Repo list / card components
│   │   ├── scan/               # Scan result components
│   │   ├── packages/           # Dependency matrix components
│   │   ├── cve/                # CVE alert components
│   │   ├── operations/         # Batch operation components
│   │   ├── merge-queue/        # PR merge queue components
│   │   ├── scripts/            # Custom script runner components
│   │   ├── compliance/         # Secret scanning, licence, branch protection components
│   │   ├── drift/              # Drift dashboard components
│   │   └── layout/             # Sidebar, header, shell
│   ├── views/                  # Top-level route views
│   │   ├── Dashboard.vue
│   │   ├── RepoLists.vue
│   │   ├── Scanner.vue
│   │   ├── Packages.vue
│   │   ├── CVEAlerts.vue
│   │   ├── CVEIncident.vue     # Incident timeline per CVE
│   │   ├── Operations.vue
│   │   ├── MergeQueue.vue      # PR merge queue
│   │   ├── ScriptRunner.vue    # Custom script runner
│   │   ├── DriftDashboard.vue  # Cross-repo drift analysis
│   │   ├── Compliance.vue      # Secret scanning, licence audit, branch protection
│   │   ├── Settings.vue
│   │   └── Auth.vue
│   └── types/                  # Shared TypeScript types (mirroring Rust models)
│       ├── repo.ts
│       ├── scan.ts
│       ├── package.ts
│       ├── cve.ts
│       ├── operation.ts
│       ├── mergeQueue.ts
│       ├── script.ts
│       └── compliance.ts
│
├── .flotilla/                  # Local config (gitignored tokens, not gitignored structure)
│   ├── config.yaml             # App config (scan intervals, defaults)
│   └── repo-lists/             # Saved repo list definitions (YAML, committable)
│
├── CLAUDE.md                   # This file
├── PLANNING.md                 # Full feature plan and implementation status
├── README.md                   # Public-facing documentation
├── package.json
├── pnpm-lock.yaml
├── vite.config.ts
├── tailwind.config.ts
└── tsconfig.json
```

---

## Domain Models

These are the core types. Rust structs in `src-tauri/src/models/` must match the TypeScript types in `src/types/`.

```typescript
// Repo
interface Repo {
  id: string               // "{provider}:{owner}/{name}"
  provider: 'github' | 'gitlab'
  owner: string
  name: string
  fullName: string         // "owner/name"
  url: string
  defaultBranch: string
  isPrivate: boolean
  lastScannedAt: string | null
  tags: string[]           // user-defined labels
}

// RepoList
interface RepoList {
  id: string
  name: string
  description: string
  repoIds: string[]
  parentId: string | null  // for nested groups
  excludePatterns: string[] // org-level ("ORG/*") and repo-level exclusion rules
  createdAt: string
  updatedAt: string
}

// ScanResult
interface ScanResult {
  repoId: string
  scannedAt: string
  manifestPaths: string[]    // all package.json/composer.json paths (monorepo-aware)
  nodeVersion: string | null
  nodeVersionSource: string | null  // ".nvmrc" | ".node-version" | ".tool-versions" | "engines.node" | "CI workflow"
  phpVersion: string | null
  packageManager: 'npm' | 'pnpm' | 'yarn' | 'bun' | 'composer' | null
  packageManagerVersion: string | null
  hasDevelop: boolean        // whether develop branch exists (for PR targeting)
  lastPushed: string | null  // repo's last push timestamp (staleness detection)
  hasDotEnvExample: boolean
  workflowFiles: string[]
  healthScore: number        // 0-100
  flags: ScanFlag[]
  excluded: boolean          // auto-excluded (e.g. no relevant manifests)
  excludeReason: string | null
}

// Package (from dependency files)
interface RepoPackage {
  repoId: string
  ecosystem: 'npm' | 'composer' | 'pip' | 'cargo' | 'go'
  name: string
  version: string
  isDev: boolean
  scannedAt: string
}

// CVE Alert
interface CveAlert {
  id: string               // CVE ID e.g. "CVE-2024-12345"
  packageName: string
  ecosystem: string
  severity: 'critical' | 'high' | 'medium' | 'low'
  summary: string
  affectedVersionRange: string
  fixedVersion: string | null
  publishedAt: string
  detectedAt: string       // when Flotilla first saw it
  affectedRepos: string[]  // repoIds using this package
  status: 'new' | 'acknowledged' | 'patched' | 'dismissed'
}

// BatchOperation
interface BatchOperation {
  id: string
  type: 'file_update' | 'package_pin' | 'package_bump' | 'workflow_sync' | 'script_run' | 'pr_create' | 'commit'
  mode: 'pin' | 'bump' | null      // pin = exact version + overrides, bump = range + remove overrides
  status: 'pending' | 'running' | 'completed' | 'failed' | 'rolled_back' | 'paused'
  targetRepoIds: string[]
  completedRepoIds: string[]        // for resumability — tracks per-repo progress
  versionMap: Record<string, string> | null  // multi-major version targeting: { "0": "0.30.3", "1": "1.13.6" }
  createdAt: string
  completedAt: string | null
  results: OperationResult[]
  isDryRun: boolean
  skipCi: boolean                   // append [skip ci] to commit messages
}
```

---

## Key Features & Implementation Notes

### 1. Authentication
- Support GitHub (Personal Access Token + OAuth App) and GitLab (PAT + OAuth)
- Store tokens in OS keychain via `keyring` crate — **never** in config files or SQLite
- Support multiple accounts (e.g. two GitHub accounts, one GitLab)
- Validate token scopes on save and warn if required scopes are missing
- Required GitHub scopes: `repo`, `workflow`, `read:org`
- Required GitLab scopes: `api`, `read_repository`, `write_repository`

### 2. Repo Lists & Discovery
- Auto-discover all repos under authenticated orgs/groups
- Support nested list hierarchy: Client → Project → Repos
- Import/export repo lists as YAML for team sharing
- Tag repos with arbitrary labels for dynamic filtering
- Repo lists stored in `.flotilla/repo-lists/*.yaml` (committable)

### 3. Scanning
- Scan target: individual repo, repo list, or all repos
- **Monorepo-aware**: discover all manifest files (`package.json`, `composer.json`, etc.) excluding `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `.nuxt/`, `.cache/` — store as `manifestPaths[]`
- What to scan per repo:
  - `package.json`, `composer.json`, `requirements.txt`, `Cargo.toml`, `go.mod` → extract all dependencies + versions
  - `.nvmrc`, `.node-version`, `.tool-versions`, CI workflows, `package.json#engines.node` → runtime versions (in priority order), track `nodeVersionSource`
  - `.github/workflows/*.yml` → workflow file inventory, detect floating action tags
  - `composer.json` → PHP version constraint
  - Detect package manager version from lockfile headers, `packageManager` field, `engines.pnpm`/`engines.npm`
  - Detect `develop` branch existence (`hasDevelop`) for PR targeting
  - Presence of: `.env.example`, `CODEOWNERS`, `SECURITY.md`, `.editorconfig`
- **Auto-exclude**: mark repos without relevant manifests as excluded with reason
- **Rate limit awareness**: display remaining API quota, auto-pause when low (<100), configurable inter-request delay (default: 200ms)
- **Incremental scan**: only re-check repos pushed since last scan (`lastPushed` comparison)
- Store scan results in SQLite with timestamp — keep history for drift detection
- Scan diff: compare latest scan vs previous to show what changed
- Fingerprint profiles: user-defined "healthy repo" definitions that flag deviations
- Scheduled scans: configurable interval (default: daily), runs in background via scheduler

### 4. Package Overlap & Dependency Intelligence
- After scanning, build a cross-repo dependency matrix per ecosystem
- Show: which repos share a package, version used per repo, latest available version
- Highlight version drift across repos for same package
- Identify packages unique to one repo vs shared across many
- Flag packages that have been superseded (e.g. `node-fetch` → native fetch)
- **Changelog aggregation**: when proposing a bump, fetch and display changelog entries between current and target version (from GitHub Releases API or `CHANGELOG.md`); highlight breaking changes
- "Standardise version" action: select package + target version → open PRs across all selected repos
- Export matrix as CSV or JSON

### 5. CVE Monitoring
- **Trigger:** after every scan completes, immediately run CVE check against all detected packages
- **Polling:** hourly background job (user-configurable: off / 15min / 30min / 1hr / 6hr / daily)
- **Sources to scrape:**
  - OSV.dev API (https://api.osv.dev/v1/query) — primary, covers npm/composer/pip/cargo/go
  - GitHub Advisory Database (https://api.github.com/advisories) — secondary
  - NVD NIST API (https://services.nvd.nist.gov/rest/json/cves/2.0) — tertiary
- Match CVEs against ALL packages in SQLite from latest scans
- Severity classification: critical / high / medium / low (from CVSS score)
- Alert UI: badge on sidebar nav, notification panel, per-repo CVE count on cards
- One-click "Patch affected repos" → pre-fills a batch operation to **pin** the package to `fixedVersion` (default to pin mode for security)
- **Incident timeline view**: unified timeline per CVE showing when published, when detected, which repos scanned, PRs created/merged/open
- **Blast radius analysis**: dependency graph showing direct + transitive exposure for an affected package; helps prioritise patching order
- CVE watchlist: user can subscribe to specific packages regardless of whether they appear in scans
- Snooze / dismiss individual CVEs per repo
- **Automated rollback detection**: monitor merged Flotilla PRs for reverts; alert immediately
- Audit log entry created for every CVE acknowledgement or patch action

### 6. Batch Operations
- **File update:** push a file (or file template with variable injection) to N repos via commit or PR
- **Workflow sync mode:** dedicated mode for pushing GitHub Actions workflows, `.nvmrc`/`.node-version` updates, `package.json` field updates across repos; includes built-in template library (lockfile regen, hotfix-back-to-develop)
- **Package pin:** set exact version + add `overrides`/`resolutions`/`pnpm.overrides` to lock entire dep tree — for active incidents
- **Package bump:** set version range + remove overrides — for after upstream fix lands
- **Pin-then-bump lifecycle:** track which repos are still pinned; surface when it's time to bump back
- **Version map:** target different safe versions per major version (e.g. major 0 → 0.30.3, major 1 → 1.13.6)
- **Monorepo-aware patching:** update all matching manifests within a repo; overrides only at root `package.json`
- **Validate mode:** audit whether a fix is already applied across all repos without making changes
- **Dry run mode:** always available — shows diff per repo without writing anything
- **Parallelism:** configurable worker count (default: 5 concurrent repos)
- **Resumability:** save per-repo progress to SQLite; resume from where it left off after crash/abort
- **PR options:**
  - Title template (supports `{{PACKAGE}}`, `{{VERSION}}`, `{{REPO}}`, `{{CVE}}`, `{{SEVERITY}}`, `{{DATE}}`, `{{MODE}}` variables)
  - Body template (Markdown) with **conditional sections**: `{{#FIELD}}content{{/FIELD}}` removed when field is empty
  - Separate default templates for pin vs bump operations
  - Draft PR toggle
  - **Skip CI toggle:** append `[skip ci]` to commit messages
  - Auto-assign reviewers (from CODEOWNERS or configured default)
  - Target branch override
  - **Multi-branch targeting:** create PRs against additional branches (develop, staging) in the same operation
  - **Divergence detection:** auto-detect when `develop` has diverged >N commits from main; offer separate PR
  - **Idempotent PR creation:** detect existing PR from same branch → close → delete stale branch → create fresh
- **Rollback:** for Flotilla-initiated commits, store the pre-change SHA and offer a revert PR
- **PR merge queue:** dedicated view for all open Flotilla PRs with CI status, conflict detection, one-click merge, and "merge all green" batch merge
- **Custom script runner:** run arbitrary shell commands across N repos (clone → run → collect output); preset library + save custom commands

### 7. Repo Health Scoring
Score 0–100 per repo based on configurable rules:
- Has `CODEOWNERS` (+10)
- Has `SECURITY.md` (+10)
- Has `.env.example` (+5)
- Has `.editorconfig` (+5)
- No floating Action tags in workflows (+15)
- Dependencies up to date within 1 major version (+20)
- No known CVEs (+20)
- Node/PHP version not EOL (+15)

### 8. Command Palette
- Global `Ctrl+K` / `Cmd+K` shortcut
- Search repos, repo lists, views, and actions
- Recent actions surfaced first
- Keyboard-navigable with arrow keys + Enter

### 9. Notifications & Reporting
- In-app notification centre for: scan completions, CVE alerts, PR merges/failures
- Webhook support: Slack / Teams / Discord (configurable URL + event filters)
- Weekly digest: exportable JSON/CSV summary of repo health across all lists
- Full audit log: every Flotilla action logged with timestamp, user, repos affected, outcome

### 10. Config Portability & Team Mode
- `.flotilla/config.yaml` — app-level settings (scan intervals, defaults, health score weights)
- `.flotilla/repo-lists/*.yaml` — committable repo list definitions
- Per-repo config cached after scan in `.flotilla/cache/{repoId}.json` (gitignored)
- Auth tokens: OS keychain only, never in `.flotilla/`
- Team sharing: commit `.flotilla/` (excluding `cache/`) to a shared repo; teammates pull to sync lists and config

---

## UI / Design System

### Aesthetic Direction
Git Flotilla is a **professional DevOps tool** — not a consumer app. The design should feel:
- Dense but not cluttered — information-rich like a terminal or IDE
- Dark-first (light theme available)
- Monospace accents for code values (versions, package names, SHAs)
- Status communicated through colour: green (healthy), amber (warning), red (critical), blue (info)
- Subtle animations only — no decorative motion

### Layout
```
┌──────────────────────────────────────────────────────────────┐
│  [Logo] Git Flotilla    [Search] [API ●4832] [Notif] [Auth]  │  ← Top bar (+ rate limit indicator)
├──────────┬───────────────────────────────────────────────────┤
│          │                                                    │
│ Sidebar  │  Main content area                                 │
│          │                                                    │
│ Dashboard│                                                    │
│ Repos    │                                                    │
│ Scanner  │                                                    │
│ Packages │                                                    │
│ CVE ●3   │                                                    │
│ Ops      │                                                    │
│ PRs      │  ← Merge queue                                     │
│ Scripts  │  ← Custom script runner                             │
│ Settings │                                                    │
│          │                                                    │
└──────────┴───────────────────────────────────────────────────┘
```

### Colour Tokens (Tailwind config)
```
primary:     #3B82F6  (blue-500)
success:     #22C55E  (green-500)
warning:     #F59E0B  (amber-500)
danger:      #EF4444  (red-500)
surface:     #0F1117  (dark bg)
surface-alt: #1A1D27  (card bg)
border:      #2A2D3A
muted:       #6B7280  (gray-500)
```

---

## Development Commands

```bash
# Install dependencies
pnpm install

# Run in development (Tauri dev server + Vite HMR)
pnpm tauri dev

# Type check frontend
pnpm typecheck

# Lint frontend
pnpm lint

# Run Rust tests
cargo test --manifest-path src-tauri/Cargo.toml

# Lint Rust
cargo clippy --manifest-path src-tauri/Cargo.toml

# Format Rust
cargo fmt --manifest-path src-tauri/Cargo.toml

# Build for production
pnpm tauri build

# Run DB migrations (dev)
cargo sqlx migrate run --manifest-path src-tauri/Cargo.toml
```

---

## Bootstrap Order

When starting from scratch, implement in this order:

1. **Project scaffold** — `pnpm create tauri-app`, configure Vite, Vue 3, Tailwind, Pinia, Router
2. **DB schema** — SQLite migrations for all tables (repos, repo_lists, scan_results, packages, cve_alerts, operations, audit_log, scripts, compliance)
3. **Auth** — GitHub PAT auth, keychain storage, token validation
4. **Repo discovery** — list repos from GitHub API, store in SQLite
5. **Repo lists** — CRUD for repo lists, YAML import/export, exclusion patterns
6. **Shell layout** — sidebar, top bar (with rate limit indicator), router views, command palette stub
7. **Scanner** — scan a single repo, monorepo-aware manifest discovery, parse package files, store results with enriched metadata
8. **Batch scanner** — parallel scan of repo list, rate limit awareness, incremental scan, auto-exclude
9. **Package matrix** — cross-repo dependency view, changelog aggregation
10. **CVE integration** — OSV.dev polling, match against scanned packages, alert UI, incident timeline, blast radius analysis
11. **CVE scheduler** — hourly background job, user-configurable interval, rollback detection
12. **Batch operations** — dry run, file update, package pin/bump with lifecycle tracking, version maps, monorepo-aware patching, resumability
13. **Workflow sync** — dedicated workflow/dotfile sync mode with template library
14. **PR workflow** — PR creation, conditional templates, idempotent creation, divergence detection, multi-branch targeting, skip CI
15. **PR merge queue** — dedicated PR management view, batch merge, CI status tracking
16. **Health scoring** — configurable rules, per-repo score, drift dashboard
17. **Repo similarity clustering** — auto-group repos by tech stack fingerprint
18. **Custom script runner** — run arbitrary commands across repos, preset library
19. **Compliance** — secret exposure scanner, licence compliance matrix, branch protection audit
20. **Notifications** — in-app (with rollback alerts), webhook
21. **GitLab support** — mirror GitHub implementation for GitLab API
22. **Reporting** — audit log view, weekly digest export, downloadable operation logs
23. **Repo archival** — stale repo detection, batch archive
24. **CLI companion** — expose core commands as a CLI binary

---

## Commit Conventions

Follow Conventional Commits:
```
feat(cve): add hourly OSV.dev polling scheduler
fix(scan): handle missing package.json gracefully
chore(deps): bump reqwest to 0.12
docs(planning): mark scanner as [implemented]
```

Scopes: `auth`, `repos`, `scan`, `packages`, `cve`, `ops`, `merge-queue`, `scripts`, `compliance`, `ui`, `db`, `cli`, `settings`

---

## What NOT to Do

- Do not store secrets in SQLite, config files, or environment variables — use OS keychain
- Do not call GitHub/GitLab APIs from the Vue frontend — all API calls go through Rust commands
- Do not use `unwrap()` or `expect()` in Rust production code paths
- Do not use the Options API in Vue components
- Do not use `any` in TypeScript
- Do not make UI decisions that require recompiling Rust — keep business logic configurable
- Do not open PRs without a dry run result being computed first
- Do not delete audit log entries

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/immersedone) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
