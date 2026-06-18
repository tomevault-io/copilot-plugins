## socialpulse

> Project-specific instructions for Claude Code.

# CLAUDE.md

Project-specific instructions for Claude Code.

## Project Overview

**SocialPulse** is a Go CLI tool that generates static discussion analysis sites from YAML/JSON content files. It's similar to MkDocs but focused on article discussion summaries with persona modeling, sentiment analysis, and trend visualization.

- **Output**: Static HTML with ECharts visualizations
- **Input**: YAML/JSON content files (summaries, digests)
- **Use case**: Track and analyze discussions on HN, Reddit, etc.

## Project Structure

```
socialpulse/
├── cmd/socialpulse/        # CLI entry point
├── internal/
│   ├── cli/                # Command implementations (new, build, serve, validate, verify, fetch-quotes, gh-deploy)
│   ├── site/               # Site builder (template rendering, page generation)
│   ├── server/             # Dev server with live reload
│   └── theme/              # Embedded templates and assets
├── types/                  # Data structures (Summary, Digest, Site)
├── schema/                 # JSON schemas for validation
└── textutil/               # Text processing utilities
```

## Development

### Build

```bash
go build -o socialpulse ./cmd/socialpulse
```

### Test

```bash
go test -v ./...
```

### Lint

```bash
golangci-lint run
```

### Install Locally

```bash
go install ./cmd/socialpulse
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `new <name>` | Create new site scaffold |
| `build` | Generate static HTML site |
| `serve` | Dev server with live reload (port 8000) |
| `validate` | Validate content against schemas |
| `verify` | Verify URLs against platform APIs |
| `fetch-quotes` | Fetch real quotes from HN discussions |
| `gh-deploy` | Deploy to GitHub Pages |

### Common Flags

- `-c, --config` - Path to configuration file (default: `socialpulse.yaml`)
- `-v, --verbose` - Verbose output

## Key Packages

### `internal/site/builder.go`

Main site generation logic:
- Loads summaries and digests from configured directories
- Builds tag index and statistics
- Renders HTML using embedded Go templates

### `internal/theme/`

Embedded templates and assets:
- `templates/` - HTML templates (index, article, digest, tag)
- `assets/` - CSS (dashboard.css) and JS (charts.js with ECharts)

### `types/`

Data structures matching the YAML schema:
- `Summary` - Article summary with personas
- `Digest` - Weekly/monthly aggregation
- `Site` - Site configuration

### `schema/`

JSON schemas generated from Go types using `invopop/jsonschema`.

## Content Schema

### Summary Structure

```go
type Summary struct {
    SchemaVersion string
    Meta          Meta        // URLs, platform, dates, author
    Article       Article     // Title, thesis, key arguments, tags, sentiment
    Discussion    Discussion  // Personas, consensus points, open questions
}
```

### Sentiment Enums

- **Article**: optimistic, cautious, pessimistic, neutral, provocative
- **Discussion**: supportive, divided, critical, exploratory
- **Stance**: agrees, disagrees, nuanced, tangential
- **Prevalence**: dominant, significant, minority

## Anti-Hallucination Features

### `verify` Command

Checks URLs against platform APIs to ensure articles and discussions exist.

### `fetch-quotes` Command

Fetches real comments from HN discussions and matches them to personas using keyword scoring. Replaces placeholder quotes with actual quotes and usernames.

## Example Sites

- [Frontier Pulse](https://github.com/grokify/frontierpulse) - AI discussion analysis

## Related Projects

- [Signal](https://github.com/grokify/signal) - Feed aggregation CLI (complementary tool)

---
> Source: [grokify/socialpulse](https://github.com/grokify/socialpulse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
