## distributed-web-crawler

> This document provides a comprehensive overview of the **Distributed Web Crawler** architecture, its running components ("agents"), the project structure, what each package is responsible for, and how to operate the system.

# Distributed Web Crawler: Agents & Architecture

This document provides a comprehensive overview of the **Distributed Web Crawler** architecture, its running components ("agents"), the project structure, what each package is responsible for, and how to operate the system.

---

## 1. Project Structure

The project is structured as a Go workspace containing three main modules: `scheduler`, `worker`, and `common`.

```text
distributed-web-crawler/
├── common/                # Shared utilities and configurations
│   ├── common.go          # Database & Broker connection handlers
│   ├── config.go          # Viper-based configuration loading
│   ├── constants.go       # Shared RabbitMQ queue/exchange names
│   └── models.go          # Shared struct definitions (CrawlMessage, UrlData)
├── scheduler/             # The central coordination agent
│   ├── cmd/scheduler/     # Main entry point for the Scheduler
│   └── internal/
│       ├── bloom/         # Redis-backed Bloom Filter for URL deduplication
│       ├── broker/        # RabbitMQ publishers and result consumers
│       ├── db/            # MongoDB batch insertion & Redis job state
│       ├── handlers/      # HTTP handlers for API endpoints (/crawl)
│       ├── metrics/       # Prometheus metrics definitions
│       └── router/        # Gin HTTP routing setup
├── worker/                # The distributed execution agents
│   ├── cmd/               # Main entry point for the Worker
│   └── internal/
│       ├── broker/        # RabbitMQ consumers for processing URLs
│       ├── crawler/       # Core scraping logic, HTML parsing (goquery)
│       ├── metrics/       # Prometheus metrics definitions
│       └── robots/        # robots.txt validation and compliance
├── compose.yaml           # Docker compose file for orchestrating the stack
├── Dockerfile             # Custom RabbitMQ image with plugins
├── prometheus.yml         # Prometheus scraping configurations
└── go.work                # Go workspace definition
```

---

## 2. System Agents & Packages

The crawler is composed of two primary agent types that work together asynchronously via RabbitMQ.

### A. The Scheduler Agent (`/scheduler`)
The **Scheduler** is the "brain" of the system. It handles incoming crawl requests, deduplicates URLs, and orchestrates the workers.

*   **`cmd/scheduler`**: Initializes connections, starts the HTTP server on `:8080`, and runs the background result consumer.
*   **`internal/router` & `internal/handlers`**: Exposes the `/crawl` POST endpoint to accept new crawl jobs and the `/metrics` endpoint.
*   **`internal/broker`**: 
    *   Publishes outgoing URLs to the RabbitMQ Consistent Hashing Exchange.
    *   Listens to the Result Queue for scraped data returning from workers.
    *   Evaluates crawl depths and schedules subsequent URLs discovered by the workers.
*   **`internal/bloom`**: Maintains a distributed Bloom filter in Redis to ensure the crawler never scrapes the same URL twice in a given job.
*   **`internal/db`**: Buffers and batch-inserts the final crawled results (HTML bodies, URLs, metadata) into MongoDB to optimize database I/O.

### B. The Worker Agents (`/worker`)
The **Workers** are the "muscle". They are horizontally scalable (running 3 replicas by default) and do the heavy lifting of fetching and parsing web pages.

*   **`cmd`**: Initializes connections and starts the RabbitMQ consumers. Exposes an internal HTTP server on `:8081` for Prometheus scraping.
*   **`internal/broker`**: Listens to assigned queues bound to the consistent hashing exchange. Passes received URLs to the crawler.
*   **`internal/crawler`**: 
    *   Executes HTTP GET requests with custom timeouts and connection pooling.
    *   Parses the returned HTML DOM using `goquery`.
    *   Extracts and normalizes all `href` links (resolving relative paths to absolute URLs).
*   **`internal/robots`**: Ensures the crawler acts ethically by fetching and parsing `robots.txt` for a given domain before making requests.

### C. The Common Package (`/common`)
*   **`common`**: A shared library imported by both agents to ensure they use exactly the same data structures (`UrlData`, `CrawlMessage`), load the same environment variables, and implement identical database connection logic.

---

## 3. Available Commands

You can interact with and manage the distributed crawler using the following commands:

### Infrastructure Management (Docker)
*   **Start the entire stack**:
    ```bash
    docker compose up -d
    ```
    *Starts MongoDB, Redis, RabbitMQ, 1 Scheduler, 3 Workers, Prometheus, and Grafana.*

*   **Rebuild Go Binaries** (Run this after making Go code changes):
    ```bash
    docker compose up -d --build
    ```

*   **View live logs** (Useful for debugging workers):
    ```bash
    docker compose logs -f worker
    # or
    docker compose logs -f scheduler
    ```

*   **Tear down the stack** (Will preserve data in volumes):
    ```bash
    docker compose down
    ```

### Interacting with the Crawler

*   **Start a Crawl Job**:
    Send a POST request to the Scheduler to begin crawling from a seed URL. Specify the URL and the maximum link depth to traverse.
    ```bash
    curl -X POST http://localhost:8080/crawl \
         -H "Content-Type: application/json" \
         -d '{"url":"https://example.com","depth":2}'
    ```

### Monitoring & Visualization

*   **Grafana Dashboards**: Navigate to `http://localhost:3000` (Default login: `admin` / `admin`).
*   **Prometheus Metrics**: Navigate to `http://localhost:9090`.
*   **RabbitMQ Management UI**: Navigate to `http://localhost:15672` (Default login: `guest` / `guest`) to watch queues fill and drain in real-time.

---
> Source: [chahatsagarmain/distributed-web-crawler](https://github.com/chahatsagarmain/distributed-web-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
