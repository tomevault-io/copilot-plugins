## ripgit

> This file is for AI coding agents. It covers the architecture, key design decisions, important constraints, and gotchas that are not obvious from reading the code in isolation.

# ripgit — agent context

This file is for AI coding agents. It covers the architecture, key design decisions, important constraints, and gotchas that are not obvious from reading the code in isolation.

## What this is

A self-hosted git server running on Cloudflare Durable Objects. Each repository is one DO with a SQLite database. The Worker entry point routes `/:owner/:repo/*` to the right DO by name. An optional TypeScript auth worker in `examples/github-oauth/` sits in front via Service Binding.

## Repository layout

```
src/
  lib.rs        Worker entry point. Routes /:owner/:repo/* to DO, /:owner/ to profile page.
                Also contains Actor/auth helpers, check_write_access, list_repos (KV query),
                handle_issue_action (issue/PR POST handler).
  pack.rs       Git pack file parser (two-pass: index + resolve). ResolveCache (Arc-based,
                budget-limited). Pack generator for upload-pack.
  git.rs        Smart HTTP protocol handlers: receive-pack (push) and upload-pack (fetch/clone).
                Pack size gate (50 MB). Calls into pack.rs and store.rs.
  store.rs      All SQLite operations: blob storage (xpatch delta compression), commit/tree
                storage, FTS index rebuild, commit graph build, ref management.
  api.rs        Read-only JSON API endpoints. parse_search_query() handles @prefix: syntax.
  diff.rs       Recursive tree diff, line-level unified diffs (similar crate), commit comparison.
  web.rs        Server-rendered HTML for all 9 web pages. layout() is the shared shell.
                Key helpers are pub(crate): layout(), html_escape(), html_response(),
                format_time(), render_markdown(), resolve_default_branch(), render_file_diff().
  schema.rs     Schema initialisation (runs in DO::new). All CREATE TABLE/INDEX statements.
  issues.rs     Issues and PR storage + merge logic. CRUD functions, three-way tree merge,
                BFS merge-base finder, git object serialization (SHA-1 via sha1_smol),
                parse_form() URL-decode utility.
  issues_web.rs Server-rendered HTML pages for issues and PRs. Uses web::layout() and helpers.

examples/github-oauth/
  src/index.ts  Auth worker: GitHub OAuth flow, session cookies, agent tokens, forwards to
                ripgit via Service Binding with X-Ripgit-Actor-* headers.
  src/types.ts  ActorProps, Env types.
  src/github.ts GitHub OAuth utilities.

scripts/
  push-test.sh  Incremental push script for large repos. Splits packs at 30 MB.
```

## Key constraints

**DO SQLite authorizer** — Cloudflare's DO SQLite blocks `PRAGMA` statements and other privileged operations. Use `sql.database_size()` (the `SqlStorage` method) for DB size — raw `PRAGMA page_count`/`PRAGMA page_size` will return `SQLITE_AUTH: not authorized`. All regular `SELECT`/`INSERT`/`UPDATE`/`DELETE` work fine.

**100 parameter limit** — DO SQLite allows at most 100 bound parameters per statement. Tree entry INSERTs are batched 25 per statement (4 params each = 100). Don't exceed this limit when adding batch operations.

**2 MB row limit** — DO SQLite rows can't exceed 2 MB. Large blobs that compress to over 2 MB are chunked across a `blob_chunks` overflow table and reassembled transparently. See `store::store_blob` and `store::read_blob`.

**128 MB DO memory limit** — the pack processing budget is designed around this. Pack body limit: 50 MB. ResolveCache budget: 20 MB. Pack generator buffers all objects before writing, so fetch/clone of large repos is a memory concern.

**Response::redirect requires absolute URLs** — `Response::redirect("/path", 302)` fails in Cloudflare Workers (Rust). Use `Response::error("", 302)` and set the `Location` header manually, or build a `new Response(null, { status: 302, headers: { Location: url } })` equivalent. There is a `unauthorized_401()` helper in `lib.rs` showing this pattern.

**DO names are permanent** — the DO is named `"{owner}/{repo}"`. This is its storage identity forever. Renaming a user or repo would orphan the DO. Don't add logic that renames DOs.

## Authentication model

The auth worker sets trusted headers before calling ripgit via Service Binding:

```
X-Ripgit-Actor-Name     GitHub username (or agent owner's username for agents)
X-Ripgit-Actor-Id       stable ID: "github:12345" or "agent:uuid"
X-Ripgit-Actor-Kind     "user" | "agent"
X-Ripgit-Actor-Scopes   comma-separated: "repo:read,repo:write,admin,..."
X-Ripgit-Actor-Owner    for agents: the owning user's actorId
```

`actor_from_request()` in `lib.rs` reads these headers. `X-Ripgit-Actor-Name` is what ripgit uses for ownership checks — it must equal the `owner` segment in the URL for writes to be allowed.

**Agent tokens** — when an agent token is created, `ownerActorName` (the owner's GitHub username) is stored alongside `actorName` (the token display name). `forwardToRipgit` in the auth worker sets `X-Ripgit-Actor-Name` to `ownerActorName` for agents, not `actorName`. This is critical — without it, `check_write_access` would compare the token name against the repo owner and always fail.

The auth worker is optional. If no actor headers are present, all reads are allowed and all writes return 401.

## Storage model

### Blobs (xpatch delta compression)

Blobs are grouped by file path (`blob_group` table). Within a group, versions are stored as:
- **Keyframes** (`is_keyframe = 1`): full content, zlib-compressed. Every 50th version.
- **Deltas** (`is_keyframe = 0`): xpatch diff from previous version.

`store::store_blob` checks if the content is identical to the previous version (skip), otherwise computes a delta. `store::read_blob` walks back through the chain to the nearest keyframe, then forward-applies deltas.

If a compressed keyframe exceeds 2 MB, it's split into `blob_chunks` rows. The threshold is `CHUNK_SIZE = 1_900_000` bytes.

### Commits and trees

Commits are stored both:
1. **Parsed** — `commits` table (hash, message, author, time, tree_hash), `commit_parents` table, `trees` table (path per row), `refs` table — used for queries, web UI, API.
2. **Raw** — `raw_objects` table (hash → raw bytes) — for byte-identical fetch/clone.

### Commit graph (binary lifting)

`commit_graph` table enables O(log N) ancestor lookups. Level 0 = direct first-parent. Level k = ancestor at distance 2^k. Used for push validation. Rebuilt via `PUT /:owner/:repo/settings/rebuild-graph` or `PUT /:owner/:repo/admin/rebuild-graph`.

### Issues and pull requests

`issues` table (unified): `id`, `number` (sequential per-repo), `kind` (`"issue"` or `"pr"`), `title`, `body`, `author_id`, `author_name`, `state` (`"open"` / `"closed"` / `"merged"`), `source_branch`, `target_branch`, `source_hash`, `merge_commit_hash`, timestamps.

`issue_comments` table: `id`, `issue_id`, `author_id`, `author_name`, `body`, timestamps.

**Auth**: any authenticated user can create issues/PRs and comment. Only the issue/PR author or repo owner can close/reopen. Only the repo owner can merge PRs.

**Merge commit algorithm** (`issues::merge_pr`):
1. BFS merge-base finder (`find_merge_base`) — walks `commit_parents` from both branch HEADs
2. `flatten_tree` — recursively reads `trees` table into a flat `HashMap<path, (mode, blob_hash)>`
3. `merge_three_way` — three-way file-level merge; returns conflict error listing paths if any file diverged in both branches
4. `build_tree_from_files` — builds new tree objects bottom-up; calls `store::store_tree` which also writes to `raw_objects`
5. `serialize_commit_content` + `git_sha1("commit", ...)` — creates merge commit bytes and SHA-1
6. `store::store_commit(..., bulk_mode=false)` — persists commit, updates `commit_graph` and `fts_commits`
7. Updates `refs` and sets `issues.state = "merged"` + `merge_commit_hash`

Fast-forward case (target is ancestor of source): merged tree = source tree directly, no three-way merge needed.

### FTS5

Three FTS5 virtual tables:
- `fts_head` — `(path UNINDEXED, content)` — file content at HEAD. Rebuilt incrementally on each push (diff engine detects changed files). Files >1 MiB are skipped.
- `fts_commits` — `(hash UNINDEXED, message, author)` — all commit messages.

Symbol-heavy queries (containing `.`, `_`, `(`, `:`) bypass FTS5 and use `INSTR` for exact substring matching. See `store::search_files_fts`.

## Routing

**Worker entry** (`lib.rs::fetch`):
- `/:owner/` → `list_repos(env, owner).await` then `web::page_owner_profile`
- `/:owner/:repo/*` → DO stub named `"{owner}/{repo}"`
- `/` → health JSON

**DO handler** (`lib.rs::Repository::fetch`):
- Parses `owner = parts[0]`, `repo_name = parts[1]`, `action = parts[2]`
- Calls `actor_from_request(&req)` early — actor is passed to all handlers
- Git protocol: `info/refs`, `git-receive-pack`, `git-upload-pack`
- JSON API: `refs`, `file`, `search`, `stats`, `log`, `commit`, `tree`, `blob`, `diff`, `compare`
- Web UI: `""` (home), `commits`, `log` (alias), `tree`, `blob`, `raw`, `search-ui`, `settings`
- Issues: `GET issues` (list), `GET issues/new`, `GET issues/:n`, `POST issues` (create), `POST issues/:n/comment`, `POST issues/:n/close`, `POST issues/:n/reopen`
- PRs: `GET pulls` (list), `GET pulls/new`, `GET pulls/:n`, `POST pulls` (create), `POST pulls/:n/comment`, `POST pulls/:n/close`, `POST pulls/:n/reopen`, `POST pulls/:n/merge` (owner only)
- Admin: `PUT admin/...` — set-ref, config, rebuild-fts, rebuild-graph, rebuild-fts-commits
- Settings actions: `POST settings/rebuild-graph`, `settings/rebuild-fts`, etc.

## Web UI (`web.rs`)

`layout(title, owner, repo_name, default_branch, actor_name, content)` generates the shared HTML shell. It takes `actor_name: Option<&str>` and uses `actor_name == Some(owner)` to decide whether to show the Settings tab and owner-specific UI.

**Two-row header:**
- Row 1 (`.global-nav`): ripgit logo left, username + sign out right
- Row 2 (`.repo-bar`): owner/repo breadcrumb, search input, then Code/Commits/Issues/PRs/Settings tabs right-anchored

**Content negotiation** in the DO handler: `wants_html` is checked for routes that can return either HTML or JSON (log, commit, tree/blob by hash, diff). HTML-only routes (`""`, `commits`, `tree/:ref/`, `blob/:ref/`, `search-ui`, `settings`, `raw`) don't check it.

**`Response::redirect` workaround** — in Rust workers, use:
```rust
let mut resp = Response::error("", 302)?;
resp.headers_mut().set("Location", &url)?;
Ok(resp)
```

## Streaming pack parser (`pack.rs`)

**Two-pass approach:**
1. **Index pass** — walks pack bytes, decompresses to a sink (discards data), records `PackEntryMeta` (offset, type, size, delta base). O(1) memory.
2. **Resolve pass** — for each entry, decompresses from pack bytes + resolves delta chains. `ResolveCache` (Arc<[u8]>, 20 MB budget) caches shared bases.

**Pack size gate** — packs over 50 MB are rejected before processing with a pkt-line `ng` response (not an HTTP error, which would confuse git).

**`ResolveCtx`** — bundles `cache: &mut ResolveCache` and `external: &ExternalObjects` (blobs already stored in SQLite, for thin packs). Passed by `&mut` through `resolve_entry` to avoid parameter explosion.

## Known gotchas / past mistakes

- **`SQLITE_AUTH`** — raw `PRAGMA` SQL is blocked. Use `sql.database_size()` for DB size. Never use `pragma_page_count()` or `pragma_page_size()` as table-valued functions.
- **`SqlStorageValue::Null`** — DO SQLite's Rust bindings throw "unrecognized JavaScript object" when passing `SqlStorageValue::Null` as a query parameter. Store empty strings `""` for absent optional text fields and map back to `None` on read via a `nonempty()` helper. NULL *in result sets* (from columns with no DEFAULT) deserializes fine into `Option<String>` — the issue is parameter binding only.
- **`Response::error("", 302)` is unreliable** — creates an "unrecognized JavaScript object" error on some Cloudflare runtimes. Use `make_redirect(req_url, path)` which constructs an absolute URL and calls `Response::redirect(url)` instead. See `lib.rs`.
- **Agent `actorName` vs owner name** — the token display name is not the owner's username. Always store and forward `ownerActorName` for agent tokens in the auth worker.
- **`Response::redirect` relative URLs** — `Url::parse("/path")` fails because relative URLs can't be parsed standalone. Always construct an absolute URL from the request origin: `format!("{}{}", req_url.origin().ascii_serialization(), path)`.
- **`url.pathname` percent-encoding** — Cloudflare Workers preserves `%3A` in pathnames rather than decoding to `:`. Always `decodeURIComponent()` path segments before using them as KV keys.
- **DO name stability** — once a repo is pushed, the DO name is `{owner}/{repo}` forever. Never add rename logic that would change the DO name — it would create a new empty DO and orphan the old one.

## Development

```bash
# Build
cargo build --target wasm32-unknown-unknown

# Run both workers locally (auth on :8787, ripgit as service binding)
cd examples/github-oauth && npm run dev:full

# Run ripgit alone (no auth, all writes open)
wrangler dev

# Push a test repo with auth
./scripts/push-test.sh -u username -t TOKEN -w http://localhost:8787 -r /path/to/repo
```

After a bulk push, rebuild indexes via the Settings page (`/:owner/:repo/settings`) or curl:
```bash
curl -H "Authorization: Bearer TOKEN" -X PUT http://localhost:8787/owner/repo/admin/rebuild-graph
curl -H "Authorization: Bearer TOKEN" -X PUT http://localhost:8787/owner/repo/admin/rebuild-fts-commits
curl -H "Authorization: Bearer TOKEN" -X PUT http://localhost:8787/owner/repo/admin/rebuild-fts
```

---
> Source: [deathbyknowledge/ripgit](https://github.com/deathbyknowledge/ripgit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
