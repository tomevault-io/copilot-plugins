## nebulous

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Overview

Nebulous is a NewsBlur MCP server written in Go. It serves feed and story data
from a local persistent index, enabling Claude to interact with feeds, stories,
subscriptions, folders, and OPML import/export over JSON-RPC stdio. The same
local index is also exposed as a structured `newsblur://` tree to the
[cutting-garden](https://github.com/amarbel-llc/cutting-garden) capture/traversal
framework via the `nebulous-cg` plugin binary.

Built on `go-mcp` from `github.com/amarbel-llc/purse-first/libs/go-mcp`.

## Build & Run

``` sh
just build-go            # Debug build → build/debug/{nebulous,migrate-cache,nebulous-cg}
just build-go release    # Release build (stripped)
just build-nix           # Nix build (reproducible, generates plugin.json)
just install-dev         # Nix build + install MCP server to ~/.claude.json
just cg list newsblur://feeds   # Drive the cutting-garden newsblur plugin
```

The Nix build uses `buildGoApplication` with `gomod2nix.toml` (not vendorHash).
After changing Go dependencies: `go mod tidy && gomod2nix` (the devShell's
go-sync-wrap hook regenerates `gomod2nix.toml` automatically after `go get` /
`go mod tidy`).

## Authentication

`NEWSBLUR_TOKEN` env var (NewsBlur session cookie) is required at runtime for
`serve mcp` and `fetch`. Store it in `.secrets.env` (gitignored, loaded by
direnv via `.envrc`). The subcommands `generate-plugin`, `hook`, `install-mcp`,
`corpus-*`, and the `nebulous-cg` plugin read only the local store and do not
require a token.

## Architecture

    cmd/nebulous/main.go           Entry point: parses args, creates client, starts MCP server
    cmd/nebulous-cg/main.go        cutting-garden CLI with the newsblur:// plugin baked in
    internal/0/madder/             `madder` CLI wrapper for blob store shell-outs
    internal/0/manifest/           SHA256 manifest tracking (leaf package)
    internal/alfa/newsblur/        HTTP client wrapping NewsBlur REST API
      client.go                    Client struct, request helpers, cache access
      cache.go                     Madder-backed persistent store keyed by a SHA256 manifest
      feeds.go, stories.go, ...    One file per API domain
    internal/bravo/tools/          MCP tool registration + handlers
      registry.go                  RegisterAll() → *command.App + ResourceProvider
      read_index.go                ReadIndex read façade over feedIndex/storyStore (for the cg plugin)
      feeds.go                     feed_query tool (word search over feeds)
      story_store.go               Flat story store with typed records and word index
      story_query.go               Query engine with structured filters + word search
      story_query_tool.go          story_query MCP tool handler
      facets.go                    Aggregate counts by year/tag/feed/status
      reader.go                    Mutation tools (mark read/unread, star/unstar)
      subscriptions.go             subscribe/unsubscribe/rename_feed
      folders.go                   Folder management
      import_export.go             OPML import/export
      resources.go                 MCP Resource provider with template URI resolution
      feed_index.go                In-memory word index over feed metadata
    internal/charlie/cgplugin/     cutting-garden newsblur:// scheme plugin
      plugin.go                    Plugin identity + Index injection (SetIndex)
      traversal.go                 Types / Roots / ListRoots
      leaf.go                      ReadLeaf (story content + original)
      url.go                       newsblur:// URL build/parse

### Two-Phase Architecture: Sync + Serve

The server operates in two distinct modes:

- **`nebulous fetch`** (sync phase): Sequential CLI command that populates the
  local persistent store by fetching from the NewsBlur API. Handles rate
  limiting with adaptive backoff. Fetches feeds metadata, starred story pages,
  and original article text. This is the sole ingestion pipeline --- the MCP
  server and the cutting-garden plugin never hit the API for reads.

- **MCP server** (serve phase): Reads exclusively from the local persistent
  store. In-memory indices (`feedIndex`, `storyStore`) are built from cached
  responses on first use via `sync.Once`. All query tools and resources operate
  against these local indices. The `nebulous-cg` cutting-garden plugin reads the
  same indices through `tools.ReadIndex`.

### Data Flow

Sync: `nebulous fetch` → `newsblur.Client` → HTTP to `newsblur.com/api/*` → JSON
response → persistent store (SHA256 manifest at `$XDG_DATA_HOME/nebulous/manifest.json`
+ a `nebulous` madder blob store).

Serve: MCP JSON-RPC (stdio) → `command.App` → `tools/*` handlers → in-memory
index (built from persistent store) → MCP response.

Traverse: `nebulous-cg <cmd>` → cutting-garden SDK → `cgplugin` (RootProvider /
LeafReader) → `tools.ReadIndex` → in-memory index → cutting-garden node/leaf.

### Key Patterns

- **Nil client convention**: `RegisterAll(nil)` is used for offline subcommands
  (`generate-plugin`, `hook`, `install-mcp`). Tool handlers and indices are only
  initialized when client is non-nil.
- **Story store**: `storyStore` holds all stories as typed records with a word
  acceleration index. Built once from cached starred story pages via
  `sync.Once`.
- **All newsblur client methods return `json.RawMessage`** --- parsing happens
  in tool handlers.
- **Persistent store**: a SHA256-keyed manifest plus a madder blob store under
  the XDG tree. The 1h TTL applies to API-fetched responses; the `fetch` command
  and index builders read without TTL checks for immutable content (original
  text, starred story pages).
- **Rate limiting**: `RateLimitError` type parses HTTP 429 + `Retry-After`
  header. `adaptiveBackoff` learns optimal wait times from rate limit bursts.

## Nix Flake

Follows the stable-first nixpkgs convention from the parent eng repo. Devenvs are
imported from `purse-first` (go + shell). `madder` is wired as a flake input and
ldflags-injected into `internal/0/madder.Bin`; `cutting-garden` is a vendored Go
dependency (gomod2nix).

---
> Source: [amarbel-llc/nebulous](https://github.com/amarbel-llc/nebulous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
