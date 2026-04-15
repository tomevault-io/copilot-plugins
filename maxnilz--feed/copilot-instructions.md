## feed

> A Go-based tool for fetching RSS/Atom feeds and sending notifications via email. It supports multiple subscribers, multiple RSS sources per subscriber, cron-based scheduling, and semantic filtering using AI.

# feed

A Go-based tool for fetching RSS/Atom feeds and sending notifications via email. It supports multiple subscribers, multiple RSS sources per subscriber, cron-based scheduling, and semantic filtering using AI.

## Project Overview

*   **Language:** Go (1.20+)
*   **Purpose:** RSS Aggregator and Email Notifier.
*   **Key Libraries:**
    *   `github.com/mmcdole/gofeed`: RSS/Atom parsing.
    *   `github.com/robfig/cron/v3`: Job scheduling.
    *   `github.com/mattn/go-sqlite3`: SQLite storage.
    *   `gopkg.in/yaml.v3`: Configuration parsing.
    *   `github.com/google/generative-ai-go`: AI Embeddings.

## Architecture

The application follows a decoupled architecture with the following components:

*   **Main (`main.go`):** Entry point. Parses flags, loads configuration, initializes dependencies, and schedules jobs.
*   **Fetcher (`fetcher.go`):** Runs on a frequent interval (e.g., every 10 minutes) to:
    1.  Fetch feeds from configured URLs.
    2.  Filter items based on semantic similarity (optional).
    3.  Save new items to the database (`item` table).
    4.  Updates the fetch cursor (`subscription_cursor`) to track progress.
*   **NotifierJob (`notifier_job.go`):** Runs on a user-defined schedule (e.g., daily digest) to:
    1.  Query the database for unacknowledged items (`ack=0`).
    2.  Send an email summary using `Notifier`.
    3.  Mark items as acknowledged (`ack=1`).
*   **ArchiverJob (`archiver_job.go`):** Runs daily to:
    1.  Move old acknowledged items from `item` table to `history_item` table to maintain performance.
*   **Storage (`storage.go`, `sqlite.go`):**
    *   Abstracts persistence via the `Storage` interface.
    *   Uses SQLite (`sqlite3://`) with tables `item`, `history_item`, and `subscription_cursor`.
    *   Handles deduplication via unique constraints.
*   **Notifier (`notifier.go`):**
    *   Sends HTML emails using SMTP.
    *   Uses `html/template` for safe and flexible content generation.
*   **AI (`ai/`):**
    *   Provides `Embedder` interface and Gemini implementation for semantic analysis.

## Configuration

Configuration is handled via a YAML file (default: `config.yaml`). See `config.example.yaml` for a template.

Key sections:
*   `dsn`: Database connection string (e.g., `sqlite3:///path/to/feed.db`).
*   `geminiAPIKey`: API key for Google Gemini (required for semantic filtering).
*   `fetchInterval`: Frequency for the Fetcher job (default: 10m).
*   `subscribers`: List of users, their emails, feed sources, cron schedules, and semantic filters.
*   `mailSender`: SMTP server details.

## Building and Running

### Local Development

1.  **Install Dependencies:**
    ```bash
    go mod download
    ```

2.  **Run:**
    ```bash
    go run . -config config.yaml -verbose
    ```
    *   Ensure `config.yaml` exists and is valid.

3.  **Build:**
    ```bash
    go build -o feed .
    ```

4.  **Test:**
    ```bash
    go test ./...
    ```

### Docker

Build the image:
```bash
docker build -t feed .
```

Run with volume mounts for config and database:
```bash
docker run --rm \
  -v ${PWD}/config.yaml:/usr/local/feed/config.yaml \
  -v ${PWD}/feed.db:/usr/local/feed/feed.db \
  --name feed feed:latest
```

## Development Conventions

*   **Dependency Injection:** Dependencies (Storage, Notifier, Embedder) are manually injected in `main.go`.
*   **Error Handling:** Uses a custom `errors` package (`github.com/maxnilz/feed/errors`).
*   **Logging:** Custom logger interface (`Logger`) with `DefaultLogger` and `VerboseLogger`.
*   **Formatting:** Standard `gofmt`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maxnilz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/maxnilz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
