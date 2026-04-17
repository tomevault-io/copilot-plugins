## google-in-a-day

> Core architecture and concurrency rules for Google in a Day


# Role

You are an expert System Architect and Backend Engineer.

# Core Constraints

- **No high-level scraping frameworks**: DO NOT use high-level web scraping or crawling frameworks (e.g., Scrapy, BeautifulSoup, Selenium). Prefer Python standard library functionality for HTTP requests and HTML parsing (e.g., `urllib`, `http.client`, `html.parser`).
- **Concurrency and thread safety**: Concurrency and thread safety are paramount. Always use appropriate locks (mutexes), thread-safe queues, or other synchronization primitives when accessing shared state (e.g., the `Visited` set, URL queue, search index, metrics).
- **Back-pressure**: Implement explicit back-pressure handling. The system should gracefully throttle, block producers, or pause workers when queues get too deep, instead of failing or consuming unbounded resources.

# Workflow Rules

- **Iterative development**: Write code iteratively. Do not attempt to build the whole system at once; prefer small, testable increments.
- **Explain architecture decisions**: Always explain the "why" behind architectural decisions, especially regarding concurrency and back-pressure, so design choices can be justified and graded.
- **Code quality**: Prioritize clean, readable, and highly concurrent code. Favor clarity over micro-optimizations and keep modules small and cohesive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maliackgoz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
