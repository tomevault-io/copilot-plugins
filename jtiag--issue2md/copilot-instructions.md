## issue2md

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@constitution.md
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**issue2md** is a Go tool that converts GitHub issues, pull requests, and discussions into Markdown format. It has two modes:
- **CLI mode**: Command-line tool (`cmd/issue2md/main.go`)
- **Web mode**: HTTP server with web UI (`main.go`)

The project uses GitHub's GraphQL API for data fetching and `github.com/google/uuid` for UUID generation.

## Build and Development Commands

```bash
# Build both CLI and web versions
make all

# Build CLI version only
make cmdline

# Build web version only
make web

# Build Docker image
make buildimage

# Clean built binaries
make clean

# Install CLI tool from source
go install github.com/bigwhite/issue2md/cmd/issue2md@latest

# Run web server (after building)
./issue2mdweb
# With custom IP/port:
./issue2mdweb -ip 127.0.0.1 -port 3000
```

## Architecture

### GitHub GraphQL API

The project uses GraphQL instead of REST API for all GitHub data fetching:
- **Single endpoint**: `https://api.github.com/graphql`
- **Cursor-based pagination**: Uses `pageInfo.hasNextPage` and `pageInfo.endCursor`
- **Inline reactions**: Fetched with parent objects, no separate API calls needed
- **Authentication**: `Authorization: Bearer TOKEN` (requires `GITHUB_TOKEN` env var)

### Package Structure

- `internal/github/` - GitHub GraphQL API client
  - `graphql.go` - GraphQL infrastructure (`GraphQLRequest`, `GraphQLResponse`, `executeGraphQL()`)
  - `queries.go` - GraphQL query templates for issues, PRs, discussions, and comments
  - `github.go` - Main fetch functions (`FetchIssue()`, `FetchPullRequest()`, `FetchDiscussion()`, `FetchComments()`, `FetchDiscussionComments()`)
  - `ParseURL()` - Parses GitHub URLs to extract owner, repo, number, and type

- `internal/converter/` - Markdown generation
  - `IssueToMarkdown()`, `PullRequestToMarkdown()`, `DiscussionToMarkdown()` - Convert GitHub data to Markdown
  - `writeComment()` - Formats individual comments
  - `writeReactions()` - Formats reaction summaries

- `internal/utils/` - Utility functions
  - `NewUUID()` - Generates UUID v4 strings using `github.com/google/uuid`

- `web/handlers/` - HTTP handlers for web mode
  - `HomeHandler` - Serves the web UI
  - `ConvertHandler` - Handles conversion requests and returns downloadable files

- `cmd/issue2md/` - CLI entry point with flag parsing
- `web/static/` and `web/templates/` - Web UI assets

### Key Implementation Details

1. **Shared HTTP Client**: `internal/github/graphql.go:41` uses a package-level `sharedHTTPClient` with 30-second timeout.

2. **GraphQL Response Types**: All GraphQL responses use `json.RawMessage` for flexible unmarshaling into nested structs.

3. **Pagination**: Comments use cursor-based pagination - fetch in batches of 100, check `pageInfo.hasNextPage`, and use `pageInfo.endCursor` for next page.

4. **ID Type Change**: GraphQL returns string IDs for comments (not integers), so `Comment.ID` and `DiscussionComment.ID` are `string` type.

5. **Reaction Content Format**: GraphQL returns native reaction names (e.g., `THUMBS_UP`, `LAUGH`, `HOORAY`) instead of REST API's emoji-style names.

### CLI Flags

- `-enable-reactions` - Include GitHub reactions in output (requires auth token)
- `-enable-user-links` - Convert usernames to profile links (`@user` → `[@user](https://github.com/user)`)

## Dependencies

```bash
# View dependencies
go list -m all

# Add new dependency
go get github.com/example/pkg
```

Current external dependency:
- `github.com/google/uuid v1.6.0`

## Environment Variables

- `GITHUB_TOKEN` - GitHub personal access token (required for GraphQL, needed for reactions)

## Docker Deployment

```bash
# Build image
make buildimage

# Run container
docker run -d -p 8080:8080 bigwhite/issue2mdweb
```

See `scripts/issue2md.service` for systemd service configuration.

---
> Source: [Jtiag/issue2md](https://github.com/Jtiag/issue2md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
