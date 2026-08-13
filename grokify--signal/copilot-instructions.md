## signal

> Project-specific instructions for Claude Code.

# CLAUDE.md

Project-specific instructions for Claude Code.

## Project Overview

**Signal** is a Go CLI tool for RSS/Atom feed aggregation, designed for static file hosting and AI agent consumption. It functions as a Planet-style blog aggregator, outputting JSON Feed 1.1 format with custom extensions.

- **Output**: JSON Feed 1.1, Atom XML, monthly archives
- **Input**: OPML (JSON format), priority links
- **Use case**: Aggregate multiple blogs into a single feed for static sites

## Project Structure

```
signal/
├── cmd/signal/             # CLI entry point
├── aggregator/             # Feed fetching and parsing (uses gofeed)
├── entry/                  # Internal entry types, deduplication
├── jsonfeed/               # JSON Feed 1.1 implementation
├── atom/                   # Atom feed generation
├── opml/                   # OPML parsing (JSON format)
├── priority/               # Hand-curated priority links
├── monthly/                # Monthly file splitting and merging
└── api/                    # Agent-friendly API structure generation
```

## Development

### Build

```bash
go build -o signal ./cmd/signal
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
go install ./cmd/signal
```

## CLI Commands

### `signal aggregate`

Main command for feed aggregation.

```bash
signal aggregate \
  --opml feeds.json \
  --priority priority.json \
  --output-dir data \
  --monthly \
  --latest-months 24 \
  --title "My Feed" \
  -v
```

### Key Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--opml, -o` | `feeds.json` | OPML file (JSON format) |
| `--priority, -p` | | Priority links file |
| `--output-dir, -d` | `data` | Output directory |
| `--output, -f` | `feeds.json` | Output JSON filename |
| `--monthly` | false | Split output into monthly files |
| `--latest-months` | 3 | Months in latest feed (0 = all) |
| `--merge` | true | Merge with existing monthly files |
| `--atom` | | Generate Atom feed file |
| `--max-entries` | 50 | Max entries per feed |
| `--concurrency` | 10 | Concurrent feed fetches |
| `-v, --verbose` | false | Show progress bar |

### `signal init`

Initialize a new project with sample files.

## Key Packages

### `aggregator/`

Feed fetching using `gofeed` library:
- Concurrent fetching with configurable concurrency
- 30-second timeout per feed
- Progress callbacks for CLI output

### `entry/`

Internal entry types and processing:
- `Entry` - Single feed entry with Signal extensions
- `Feed` - Collection with deduplication and sorting
- Conversion to JSON Feed 1.1

### `monthly/`

Monthly archive handling:
- Split entries by month
- Merge with existing files (preserves history)
- Generate index.json

### `jsonfeed/`

JSON Feed 1.1 types with Signal extensions:
- `_signal_feed_title` - Source feed name
- `_signal_feed_url` - Source feed URL
- `_signal_priority` - Hand-curated flag
- `_signal_rank` - Priority rank
- `_signal_discussions` - Discussion links (HN, Reddit, etc.)

## Input Formats

### OPML (feeds.json)

```json
{
  "version": "2.0",
  "title": "My Feeds",
  "outlines": [
    {
      "text": "Category",
      "outlines": [
        {
          "text": "Feed Name",
          "type": "rss",
          "xmlUrl": "https://example.com/feed.xml",
          "htmlUrl": "https://example.com",
          "categories": ["AI", "Tech"]
        }
      ]
    }
  ]
}
```

### Priority Links (priority.json)

```json
{
  "title": "Curated Articles",
  "links": [
    {
      "title": "Article Title",
      "url": "https://example.com/article",
      "author": "Author",
      "date": "2026-04-01T00:00:00Z",
      "tags": ["AI"],
      "summary": "Brief summary",
      "rank": 1,
      "discussions": [
        {
          "platform": "hackernews",
          "url": "https://news.ycombinator.com/item?id=12345",
          "id": "12345",
          "score": 100
        }
      ]
    }
  ]
}
```

## Output Structure

```
data/
├── feeds.json              # Latest N months (JSON Feed 1.1)
├── feeds-YYYY-MM.json      # Monthly archives
├── index.json              # Index of monthly files
└── atom.xml                # Optional Atom feed
```

## Example Sites

- [Planet AI](https://github.com/grokify/planet-ai) - AI blog aggregator

## Related Projects

- [SocialPulse](https://github.com/grokify/socialpulse) - Discussion analysis site generator (complementary tool)

---
> Source: [grokify/signal](https://github.com/grokify/signal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
