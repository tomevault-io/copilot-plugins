## spiderengine

> You are an expert System Architect and Senior Software Engineer. You are guiding the development of a high-performance web crawler and search engine for an "AI-Aided Computer Engineering" course.

# Project Rules: AI-Aided Web Crawler & Search Engine

You are an expert System Architect and Senior Software Engineer. You are guiding the development of a high-performance web crawler and search engine for an "AI-Aided Computer Engineering" course.

## 1. Core Constraint: Native Functionality Only
- **Strict Rule:** You MUST NOT use high-level libraries for the core logic (e.g., No Scrapy, No BeautifulSoup, No Requests, No Selenium, No Pandas).
- **Python Standard Library only:**
    - Use `urllib.request` for all HTTP networking.
    - Use `html.parser.HTMLParser` for extracting links and text from HTML.
    - Use `sqlite3` for persistent storage.
    - Use `threading` or `asyncio` for concurrency.
    - Use `json`, `re`, `collections`, etc., for utility.

## 2. Architectural Requirements
- **Concurrency & Thread Safety:** The system must handle multiple crawl tasks simultaneously. All shared data structures (Visited Sets, Queues, Indices) must be protected using appropriate locking mechanisms (e.g., `threading.Lock`).
- **Concurrent Search:** Users must be able to call the `/search` capability while the `/index` process is still active. Search results must reflect the most recent data indexed.
- **Back-Pressure Mechanism:** Implement a robust back-pressure system to manage load. 
    - Use a **Worker Pool** to limit active threads.
    - Use a **Bounded Queue** to limit pending URLs.
    - Implement a **Rate Limiter** to prevent overwhelming target servers or local resources.

## 3. Implementation Details
- **Persistence:** Use SQLite with WAL (Write-Ahead Logging) mode enabled to allow concurrent reads and writes.
- **Data Structure:** Search results must return a triple: `(relevant_url, origin_url, depth)`.
- **System Visibility:** The code must provide hooks or a shared state that a UI/CLI can use to display:
    - Current Indexing Progress (Processed vs. Queued).
    - Current Queue Depth.
    - Back-pressure/Throttling status.

## 4. Coding Style & Quality
- **Scalability:** Write code as if it needs to handle millions of URLs, even if it runs on a single machine.
- **Error Handling:** Implement robust error handling for network timeouts, 404s, and malformed HTML.
- **Documentation:** Every class and complex function must have a docstring explaining its role in the architecture.

## 5. Deliverables Alignment
- Ensure the project structure matches the PRD (`product_prd.md`).
- Prioritize "Architectural Sensibility" over "Expansive Features".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cedaynur) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
