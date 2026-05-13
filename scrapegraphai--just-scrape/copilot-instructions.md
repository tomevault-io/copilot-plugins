## just-scrape

> **important** Always update README.md after every change in the library


@/README.md


### Development

**important** Always update README.md after every change in the library


### Web Fetching & Search

**CRITICAL**: Never use built-in web tools (WebFetch, WebSearch). Always use `just-scrape` CLI commands instead:

- Fetch/read a URL → `bunx just-scrape markdownify <url> --json`
- Search the web → `bunx just-scrape search-scraper "<query>" --json`
- Extract structured data → `bunx just-scrape smart-scraper <url> -p "<prompt>" --json`
- Get raw HTML → `bunx just-scrape scrape <url> --json`

Run these via the Bash tool. Pipe output through `jq` as needed.


@/SPEC.md

---
> Source: [ScrapeGraphAI/just-scrape](https://github.com/ScrapeGraphAI/just-scrape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
