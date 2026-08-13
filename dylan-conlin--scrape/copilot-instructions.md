## scrape

> Intelligent URL-to-content CLI. Picks the best extraction strategy per URL type.

# scrape

Intelligent URL-to-content CLI. Picks the best extraction strategy per URL type.

## Architecture

```
cmd/scrape/main.go       # Entry point: URL in, markdown out
pkg/deps/
└── deps.go              # Auto-install missing tools (pip/brew)
pkg/strategy/
├── strategy.go          # Strategy interface + router
├── match.go             # URL pattern matchers
├── depcheck.go          # Dependency check (delegates to pkg/deps)
├── httpfetch.go         # Go-native HTTP fetching (replaces curl)
├── htmlconv.go          # Go-native HTML-to-markdown (replaces markitdown for HTML)
├── api.go               # JSON API extraction (reddit, HN, github)
├── playwright.go        # Full browser render (SPAs, JS-heavy)
├── screenshot.go        # Screenshot + vision model (visual content)
├── youtube.go           # Video transcript extraction
├── pdf.go               # PDF-to-markdown
└── simple.go            # Go HTTP + html-to-markdown (static HTML, default)
```

## Strategy Selection

| URL Pattern | Strategy | Why |
|-------------|----------|-----|
| reddit.com | API (.json) | Structured data, no rendering needed |
| news.ycombinator.com | API (firebase) | Clean JSON, includes comments |
| github.com | API (gh CLI) | Structured, handles auth |
| youtube.com, youtu.be | YouTube (yt-dlp) | Transcript extraction |
| *.pdf | PDF (markitdown) | Direct document conversion |
| twitter.com, x.com | Screenshot + vision | Visual-first, resists text extraction |
| everything else | Simple (Go HTTP + html-to-markdown) | Fast default, no external deps |

If simple extraction fails or returns low-quality output, escalate to playwright.

## Usage

```bash
scrape <url>                    # auto-select strategy, output markdown to stdout
scrape <url> -o output.md       # save to file
scrape <url> --strategy simple  # force specific strategy
scrape --auth-browser <url>     # attach to a running Chrome debug session for logged-in scraping
scrape --auth-browser --tab <url-prefix> <url>  # adopt an existing tab by URL prefix instead of opening a new page
```

### Authenticated Chrome Mode

Use `--auth-browser` when the page only works from an already logged-in Chrome profile.

1. Launch Chrome with the DevTools port exposed:
   ```bash
   open -na "Google Chrome" --args --remote-debugging-port=9222 --user-data-dir=~/.chrome-debug-profile
   ```
2. Log into the target site in that Chrome window.
3. Run `scrape --auth-browser <url>`.

Optional: override the debug endpoint with `--browser-debug-url http://127.0.0.1:9222`.

### Interactive Browser Mode

Use `scrape interact` to click, fill, navigate, and inspect pages through an authenticated Chrome session. All commands run against `browser.contexts()[0]` — the real auth context with cookies and session state.

```bash
scrape interact goto <url>                  # navigate to URL
scrape interact snapshot                    # accessibility tree snapshot (like playwright-cli)
scrape interact click <selector>            # click element
scrape interact fill <selector> <text>      # fill input field
scrape interact screenshot [path]           # full-page screenshot (default: screenshot.png)
scrape interact tabs                        # list open tabs via CDP
```

Override the debug endpoint: `scrape interact --browser-debug-url http://127.0.0.1:9333 goto <url>`

## Development

```bash
go build -o scrape ./cmd/scrape
go test ./...
```

## External Dependencies

Missing pip tools are auto-installed on first use via `pkg/deps`.

- `markitdown` — PDF-to-markdown conversion (auto-installed via pip, only needed for PDFs)
- `shot-scraper` — Playwright wrapper for browser rendering (auto-installed via pip)
- `playwright-cli` — Required for `--auth-browser` Chrome session attachment
- `yt-dlp` — YouTube transcript/subtitle extraction (auto-installed via pip/brew)

### Go-Native Replacements (no external tools needed)
- HTTP fetching — Go `net/http` (replaced curl)
- HTML-to-markdown — `html-to-markdown` Go library (replaced markitdown for HTML)

## Browser Handoff Convention

The standard way tools adopt Dylan's authenticated Chrome session is the **CDP endpoint + tab selector** pattern. This is a runtime capability passed as CLI flags, not prompt-time context injected into agent prompts.

### Debug Port Convention

Chrome must be running with `--remote-debugging-port=9222`. This is the ecosystem-wide default — all tools assume `http://127.0.0.1:9222` unless overridden with `--browser-debug-url`.

### Tab Selection

Tools select a tab by URL prefix match (`--tab <url-prefix>`). The lookup scans all pages across all contexts and picks the first page whose URL starts with the given prefix. If no `--tab` flag is provided, the tool falls back to the first page in `contexts()[0]`.

### Read-Only vs Interactive Modes

| Mode | Command | Behavior |
|------|---------|----------|
| Read-only | `scrape --auth-browser <url>` | Connects via CDP, extracts content from a new or matched tab, returns markdown |
| Interactive | `scrape interact <action>` | Connects via CDP, runs actions (click, fill, navigate) against the matched tab |

Both modes use `browser.contexts()[0]` — the real authenticated context with cookies and session state. Never use the default context from `connectOverCDP()`, which is a fresh context without auth cookies.

### Distinction from playwright-cli Headless Browsers

| | CDP Session Handoff (`--auth-browser`) | Capability Grant (`playwright-cli`) |
|---|---|---|
| Session state | Dylan's cookies/login | Fresh, no auth |
| Browser lifecycle | Dylan owns (agent connects) | Agent owns (open/close) |
| Safety | Shared — can see/mutate Dylan's session | Isolated sandbox |
| Use case | Authenticated content extraction | Scraping public pages, automated testing |

The spawn pipeline's `playwright-cli` integration gives agents their own headless browser. That is a **capability grant**, not a session handoff. CDP session handoff is orthogonal — it connects to Dylan's existing browser, not a fresh one.

## Accretion Boundaries

This project enforces file-size governance to prevent unbounded growth.

### Thresholds
- **Warning:** Files exceeding 600 lines with significant growth trigger warnings
- **Critical:** Files exceeding 1500 lines block commits (pre-commit gate)

### Rules
- Stage files explicitly by name — never use `git add -A` or `git add .`
- When a file approaches 600 lines, extract cohesive subsets into new files
- Pre-existing bloat is flagged as warnings, not blocks

### Bypass
- Pre-commit gate: `SKIP_ACCRETION_GATE=1 git commit -m '...'`
- Git-add gate: `SKIP_GIT_ADD_ALL_GATE=1` in environment

---
> Source: [dylan-conlin/scrape](https://github.com/dylan-conlin/scrape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
