## scrapling-mcp

> 1. [Project Overview](#1-project-overview)

# Scrapling MCP Server - Project Documentation

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Project Scope](#2-project-scope)
3. [Architecture & Structure](#3-architecture--structure)
4. [Key Components](#4-key-components)
5. [MCP Tools Reference](#5-mcp-tools-reference)
6. [Stealth Levels](#6-stealth-levels)
7. [Configuration](#7-configuration)
8. [Security Considerations](#8-security-considerations)
9. [Development Guidelines](#9-development-guidelines)
10. [Usage Examples](#10-usage-examples)

---

## 1. Project Overview

### What is the Scrapling MCP Server?

The Scrapling MCP Server is a Model Context Protocol (MCP) server that provides web scraping capabilities through an integrated stealth-aware scraping engine. Built on top of the [FastMCP](https://github.com/jlowin/fastmcp) framework and leveraging the [scrapling](https://github.com/D4Vinci/Scrapling) library, this server exposes powerful web scraping tools that AI agents and applications can invoke through a standardized MCP interface.

The project bridges the gap between AI agents that need to fetch web content and the complex reality of modern web scraping—including anti-bot protections, JavaScript rendering requirements, Cloudflare challenges, and the need for stealthy request patterns.

### Scrapling Library

The server leverages [Scrapling](https://github.com/D4Vinci/Scrapling), an adaptive web scraping framework with **9.1k GitHub stars** that provides multiple fetcher types:

| Fetcher | Use Case |
|---------|----------|
| `Fetcher` | Fast HTTP requests with TLS fingerprinting and HTTP/3 support |
| `DynamicFetcher` | Full browser automation using Playwright |
| `StealthyFetcher` | Advanced anti-bot bypass using Camoufox (modified Firefox) |
| `AsyncStealthySession` | Concurrent stealth browsing with tab pooling |

### Purpose and Goals

The primary purpose of this MCP server is to enable AI agents to:

- **Fetch web content reliably** from websites with varying levels of anti-bot protection
- **Render JavaScript** when necessary to access dynamically loaded content
- **Bypass common anti-bot measures** through configurable stealth settings
- **Handle session-based scraping** for websites requiring authentication or stateful interactions
- **Extract structured data** using CSS selectors from scraped pages

The project aims to provide a balance between:
- **Ease of use** - Simple API for common scraping tasks
- **Flexibility** - Extensive configuration options for advanced use cases
- **Reliability** - Built-in retry logic and error handling
- **Security** - URL validation and safe defaults

### Key Features and Capabilities

| Feature | Description |
|---------|-------------|
| **JavaScript Rendering** | Full browser-based rendering for dynamic content |
| **Stealth Modes** | Multiple pre-configured stealth levels (Minimal, Standard, Maximum) |
| **Cloudflare Support** | Automatic Cloudflare challenge detection and solving |
| **Session Management** | Persistent sessions for stateful scraping |
| **Proxy Rotation** | Support for proxy lists with automatic rotation |
| **Retry Logic** | Exponential backoff with configurable retry attempts |
| **CSS Extraction** | Structured data extraction using CSS selectors |
| **URL Validation** | Built-in SSRF protection and security checks |
| **MCP Integration** | Native MCP protocol support for AI agent integration |
| **Spider Framework** | Scrapy-like API with async callbacks, concurrent crawling, and pause/resume support |
| **Adaptive Parsing** | Smart element tracking that survives website design changes |
| **Camoufox Integration** | Modified Firefox browser with stealth patches for maximum anti-detection |

---

## 2. Project Scope

### What the Project Does

The Scrapling MCP Server provides a collection of MCP tools that allow AI agents to:

1. **Simple Scraping** - Fetch a URL and retrieve HTML content
2. **Stealth Scraping** - Fetch URLs with configurable anti-detection measures
3. **Session-Based Scraping** - Maintain cookies and state across multiple requests
4. **Structured Extraction** - Extract specific data using CSS selectors
5. **Batch Scraping** - Process multiple URLs in sequence

### Target Use Cases

The server is designed for the following use cases:

- **AI Agent Web Research** - Enabling AI agents to gather information from the web
- **Data Collection** - Automated gathering of publicly available web data
- **Content Aggregation** - Building datasets from multiple web sources
- **Monitoring & Alerting** - Watching web pages for changes
- **API Alternative** - Accessing websites that lack public APIs

### Supported Scraping Modes

#### Simple Mode
- Basic HTTP requests without stealth features
- Fastest performance
- Suitable for well-behaved websites without anti-bot protection
- No JavaScript rendering

#### Stealth Mode
- Configurable anti-detection features
- User-Agent randomization
- Human-like behavior simulation
- Browser automation with headless Chrome

#### Session-Based Mode
- Persistent cookie storage
- State maintenance across requests
- Authentication handling
- Ideal for authenticated scraping

### What's In Scope

- **HTTP/HTTPS scraping** with JavaScript rendering support
- **Stealth configuration** with multiple preset levels
- **Session management** for stateful interactions
- **Error handling** with automatic retry logic
- **URL validation** for security
- **MCP protocol** integration

### What's Out of Scope

- **Authentication handling** - While sessions are supported, credential management is outside scope
- **CAPTCHA solving** - No built-in CAPTCHA solving capabilities (Cloudflare challenges only)
- **Distributed scraping** - Single-instance operation
- **Data storage** - The server fetches and returns data but doesn't persist it
- **Legal compliance** - Users are responsible for ensuring their scraping activities are legal

---

## 3. Architecture & Structure

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         AI Agent / Client                              │
│                  (Claude, GPT, or other MCP clients)                   │
└─────────────────────────────────┬───────────────────────────────────────┘
                                  │ MCP Protocol (JSON-RPC)
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        MCP Server (FastMCP)                            │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                      MCP Tools Layer                            │   │
│  │  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────────┐  │   │
│  │  │  scrape   │ │  stealth  │ │  session  │ │  extract      │  │   │
│  │  │  simple   │ │  scrape   │ │  scrape   │ │  structured   │  │   │
│  │  └───────────┘ └───────────┘ └───────────┘ └───────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                  │                                      │
│                                  ▼                                      │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    Core Logic Layer                             │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────────┐  │   │
│  │  │   Stealth   │ │   Session   │ │     Retry & Error       │  │   │
│  │  │  Config     │ │  Management │ │     Handling            │  │   │
│  │  └─────────────┘ └─────────────┘ └─────────────────────────┘  │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                  │                                      │
└──────────────────────────────────┼──────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    Scrapling Integration Layer                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │              AsyncStealthySession (scrapling library)          │   │
│  │  ┌──────────────┐ ┌──────────────┐ ┌────────────────────────┐   │   │
│  │  │   Browser   │ │   HTTP       │ │   Anti-Detection       │   │   │
│  │  │   Pool      │ │   Client     │ │   │   │   Features             │
│  │  └──────────────┘ └──────────────┘ └────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                  │                                      │
└──────────────────────────────────┼──────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           Target Website                                │
│                  (Any HTTP/HTTPS accessible URL)                        │
└─────────────────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
mcp-scraper/
├── .env.example           # Example environment configuration
├── .gitignore             # Git ignore patterns
├── pyproject.toml         # Python project configuration
├── README.md              # Project README
├── src/
│   └── mcp_scraper/
│       ├── __init__.py    # Package initialization
│       ├── config.py      # Configuration classes and settings
│       └── stealth.py     # Stealth utilities and scraping logic
└── tests/                 # Test directory (to be added)
```

### Key Components and Their Responsibilities

#### MCP Server (`__init__.py`)
- **Responsibility**: Package initialization and exports
- **Exports**: Settings, StealthConfig, version information

#### Configuration Module (`config.py`)
- **Responsibility**: Define settings and configuration classes
- **Components**:
  - `StealthConfig` dataclass - Detailed stealth configuration options
  - `Settings` class - Environment-based settings using Pydantic
  - `StealthProfiles` class - Pre-configured stealth profiles

#### Stealth Module (`stealth.py`)
- **Responsibility**: Core scraping logic, session management, and utilities
- **Components**:
  - `StealthConfig` class - Stealth configuration with all options
  - `StealthLevel` enum - Preset stealth levels (MINIMAL, STANDARD, MAXIMUM)
  - `scrape_with_retry()` - Main scraping function with retry logic
  - `get_session()` - Session management
  - `validate_url()` - URL security validation
  - `format_response()` - Response formatting utility

### Data Flow

1. **Request Receipt**: Client sends MCP request with URL and optional parameters
2. **URL Validation**: System validates URL for security (SSRF protection)
3. **Configuration**: Stealth settings are applied based on parameters
4. **Session Management**: Get or create stealth session
5. **Scraping**: Execute HTTP request through scrapling engine
6. **Response Processing**: Format response with requested data
7. **Error Handling**: Apply retry logic if needed
8. **Return**: Send formatted response back to client

---

## 4. Key Components

### MCP Server (FastMCP)

The MCP server is built using FastMCP, a modern framework for creating MCP servers in Python. FastMCP provides:

- **Simple tool definition** using decorators
- **Automatic type conversion** between Python and JSON
- **Built-in error handling** for tool execution
- **Async support** for concurrent operations

The server exposes scraping functionality as MCP tools that clients can invoke.

### Scrapling Integration

The server integrates with the [scrapling](https://github.com/D4Vinci/Scrapling) library, which provides:

- **AsyncStealthySession**: An async session with built-in anti-detection features and tab pooling
- **StealthyFetcher**: Advanced anti-bot bypass using Camoufox (modified Firefox)
- **Page object**: Unified interface for accessing page content
- **Browser automation**: Headless browser with stealth features
- **JavaScript rendering**: Full DOM rendering for dynamic content
- **Spider Framework**: Scrapy-like API with concurrent crawling, pause/resume, and streaming mode
- **Adaptive Parsing**: Smart element tracking that survives website design changes

### Stealth Configuration

The stealth system provides multiple configuration options:

| Option | Description | Default |
|--------|-------------|---------|
| `headless` | Run browser in headless mode | `True` |
| `solve_cloudflare` | Attempt Cloudflare challenges | `False` |
| `humanize` | Human-like behavior simulation | `True` |
| `humanize_duration` | Maximum cursor movement duration in seconds | `1.5` |
| `geoip` | GeoIP-based routing | `False` |
| `os_randomize` | Randomize OS fingerprint | `True` |
| `block_webrtc` | Block WebRTC to prevent IP leaks | `True` |
| `allow_webgl` | Allow WebGL fingerprinting | `True` |
| `google_search` | Simulate Chrome browser | `True` |
| `block_images` | Block image loading | `False` |
| `block_ads` | Block advertisements | `True` |
| `disable_resources` | Disable CSS/JS resources | `False` |
| `network_idle` | Wait for network inactivity before returning | `False` |
| `load_dom` | Wait for DOMContentLoaded event | `False` |
| `wait_selector` | Wait for specific element to appear | `None` |
| `wait_selector_state` | Element state to wait for (visible/hidden/attached) | `None` |
| `timeout` | Request timeout in milliseconds | `30000` |
| `proxy` | Proxy URL for requests | `None` |

### Session Management

The session management system:

- **Global session cache**: Maintains a single session instance
- **Config-aware**: Recreates session when configuration changes
- **Proper cleanup**: Ensures resources are released on close
- **Cookie persistence**: Maintains cookies across requests

### Error Handling & Retry Logic

The retry system implements:

1. **Exponential backoff**: Delay increases exponentially between retries
2. **Proxy rotation**: Automatic proxy switching on block detection
3. **Cloudflare handling**: Detection and optional solving of challenges
4. **Block detection**: Identifies when requests are blocked
5. **Custom exceptions**: Specific error types for different failure modes

**Exception Hierarchy**:
```
ScrapeError (base)
├── CloudflareError - Cloudflare protection detected
├── BlockedError - Request blocked by anti-bot
└── TimeoutError - Request timed out
```

---

## 5. MCP Tools Reference

### Available Tools

The following MCP tools are available. Note: The exact tool names and parameters depend on the implementation. Below are the conceptual tools provided by the server.

#### Tool: `scrape_simple`

Simple web scraping without stealth features. Uses the `Fetcher` class for fast HTTP requests with TLS fingerprinting.

**When to use**:
- Fast scraping of static content
- Well-behaved websites without protection
- Initial testing and development

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL to scrape |
| `selector` | string | No | CSS selector for targeted extraction |
| `extract` | string | No | What to extract: "text", "html", or "both" (default: "text") |
| `timeout` | integer | No | Request timeout in milliseconds (default: 30000) |

**Example**:
```json
{
  "url": "https://example.com",
  "timeout": 30000
}
```

#### Tool: `scrape_stealth`

Stealth web scraping with configurable anti-detection. Uses the `StealthyFetcher` class with Camoufox (modified Firefox) for maximum stealth.

**When to use**:
- Websites with basic anti-bot measures
- When avoiding detection is important
- Rate-limited endpoints
- Cloudflare-protected sites

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL to scrape |
| `stealth_level` | string | No | "minimal", "standard", or "maximum" (default: "standard") |
| `solve_cloudflare` | boolean | No | Attempt Cloudflare challenges (default: false) |
| `network_idle` | boolean | No | Wait for network inactivity (default: true) |
| `load_dom` | boolean | No | Wait for DOMContentLoaded (default: true) |
| `timeout` | integer | No | Request timeout in milliseconds (default: 30000) |
| `proxy` | string | No | Proxy URL for requests |

**Example**:
```json
{
  "url": "https://example.com",
  "stealth_level": "maximum",
  "solve_cloudflare": true,
  "network_idle": true,
  "timeout": 60000
}
```

#### Tool: `scrape_session`

Session-based scraping with persistent state.

**When to use**:
- Websites requiring authentication
- Multi-step interactions
- Maintaining login state

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL to scrape |
| `session_id` | string | No | Session identifier for persistence |
| `cookies` | object | No | Initial cookies to set |
| `stealth_level` | string | No | Stealth level (default: "standard") |

**Example**:
```json
{
  "url": "https://example.com/dashboard",
  "session_id": "user-session-123",
  "cookies": {"auth": "token-value"}
}
```

#### Tool: `extract_structured`

Extract structured data using CSS selectors.

**When to use**:
- Extracting specific data from pages
- Building datasets from web content
- Structured data acquisition

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL to scrape |
| `selectors` | object \| string | Yes | Map of name → CSS selector. Can be either a JSON object or a JSON string representation. |
| `stealth_level` | string | No | Stealth level (default: "standard") |

**Selector Syntax**:

The `selectors` parameter supports CSS selector syntax with the following extensions:

| Syntax | Description | Example |
|--------|-------------|---------|
| `selector` | Extract text content | `"title": "h1"` |
| `selector::html` | Extract HTML content | `"content": "div::html"` |
| `selector::text` | Extract text using ::text pseudo-element | `"text": "p::text"` |
| `selector::attr(name)` | Extract attribute value | `"link": "a::attr(href)"` |
| `selector@attr` | Extract attribute (alternative syntax) | `"image": "img@src"` |
| `selector@attr1@attr2` | Extract multiple attributes | `"data": "img@src@alt"` |

**Example with dict input**:
```json
{
  "url": "https://example.com/blog",
  "selectors": {
    "title": "h1.article-title",
    "content": "div.article-content",
    "author": "span.author-name",
    "date": "time.publish-date",
    "link": "a.read-more::attr(href)",
    "image": "img.featured@src@alt"
  }
}
```

**Example with JSON string input**:
```json
{
  "url": "https://example.com/blog",
  "selectors": "{\"title\": \"h1.article-title\", \"content\": \"div.article-content\", \"link\": \"a.read-more::attr(href)\"}"
}
```

#### Tool: `scrape_batch`

Scrape multiple URLs in sequence.

**When to use**:
- Processing multiple pages
- Building site-wide datasets
- Bulk data collection

**Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `urls` | array | Yes | List of URLs to scrape |
| `stealth_level` | string | No | Stealth level (default: "standard") |
| `delay` | float | No | Delay between requests in seconds (default: 1.0) |

**Example**:
```json
{
  "urls": [
    "https://example.com/page1",
    "https://example.com/page2",
    "https://example.com/page3"
  ],
  "stealth_level": "minimal",
  "delay": 2.0
}
```

---

## 6. Stealth Levels

### Overview

The server provides three pre-configured stealth levels, each balancing speed, anonymity, and success rate differently.

### Minimal Stealth

**Profile**: `StealthLevel.MINIMAL` or `get_minimal_stealth()`

| Setting | Value |
|---------|-------|
| Headless | Yes |
| Humanize | No |
| Humanize Duration | N/A |
| Cloudflare solving | No |
| OS randomization | No |
| WebRTC blocking | No |
| Chrome simulation | No |
| Image blocking | Yes |
| Resource disabling | Yes |
| Ad blocking | Yes |
| Network Idle | No |
| Load DOM | No |
| Timeout | 15s |

**When to use**:
- Simple websites without anti-bot protection
- High-speed scraping where stealth is not critical
- Testing and development
- Static content and APIs

**Performance**: Fastest - suitable for high-volume scraping of cooperative sites

### Standard Stealth

**Profile**: `StealthLevel.STANDARD` or `get_standard_stealth()`

| Setting | Value |
|---------|-------|
| Headless | Yes |
| Humanize | Yes |
| Humanize Duration | 1.5s |
| Cloudflare solving | No |
| OS randomization | Yes |
| WebRTC blocking | Yes |
| Chrome simulation | Yes |
| Image blocking | No |
| Resource disabling | No |
| Ad blocking | Yes |
| Network Idle | Yes |
| Load DOM | Yes |
| Timeout | 30s |

**When to use**:
- Most web scraping tasks
- Sites with basic anti-bot protection
- General-purpose scraping
- Balance of speed and anonymity required

**Performance**: Moderate - suitable for most common scraping scenarios

### Maximum Stealth

**Profile**: `StealthLevel.MAXIMUM` or `get_maximum_stealth()`

| Setting | Value |
|---------|-------|
| Headless | Yes |
| Humanize | Yes |
| Humanize Duration | 1.5s |
| Cloudflare solving | Yes |
| OS randomization | Yes |
| WebRTC blocking | Yes |
| Chrome simulation | Yes |
| Image blocking | No |
| Resource disabling | No |
| Ad blocking | Yes |
| GeoIP routing | Yes |
| Network Idle | Yes |
| Load DOM | Yes |
| Wait Selector | body |
| Wait Selector State | visible |
| Timeout | 60s |

**When to use**:
- Heavily protected websites
- Cloudflare-protected sites
- Rate-limited endpoints
- Maximum anonymity required
- Challenging anti-bot systems

**Performance**: Slowest - but highest success rate on protected sites

### Configuration Options

You can also create custom stealth configurations:

```python
from mcp_scraper.stealth import StealthConfig

custom_config = StealthConfig(
    headless=True,
    solve_cloudflare=True,
    humanize=True,
    humanize_duration=1.5,
    geoip=False,
    os_randomize=True,
    block_webrtc=True,
    allow_webgl=True,
    google_search=True,
    block_images=False,
    block_ads=True,
    disable_resources=False,
    network_idle=True,
    load_dom=True,
    wait_selector="body",
    wait_selector_state="visible",
    timeout=45000,
    proxy="http://proxy:8080"
)
```

---

## 7. Configuration

### Environment Variables

Create a `.env` file based on `.env.example`:

```bash
# Proxy URL for requests (optional)
# Format: http://user:pass@host:port or socks5://host:port
PROXY_URL=

# Default timeout for requests in seconds (1-300)
DEFAULT_TIMEOUT=30

# Logging level: DEBUG, INFO, WARNING, ERROR, CRITICAL
LOG_LEVEL=INFO

# Maximum number of retries for failed requests (0-10)
MAX_RETRIES=3
```

### Configuration Files

#### pyproject.toml

The project uses `pyproject.toml` for Python package configuration:

```toml
[project]
name = "mcp-scraper"
version = "0.1.0"
requires-python = ">=3.10"

dependencies = [
    "scrapling[all]",
    "fastmcp>=2.0",
    "httpx>=0.25",
    "pydantic>=2.0",
    "pydantic-settings>=2.0",
    "python-dotenv>=1.0",
    "loguru>=0.7",
]
```

### Proxy Setup

#### Single Proxy

Set via environment variable:
```bash
PROXY_URL=http://proxy.example.com:8080
```

Or programmatically:
```python
config = StealthConfig(proxy="http://proxy.example.com:8080")
```

#### Proxy Rotation

For proxy rotation, pass a list of proxies to the scraping function:

```python
proxy_list = [
    "http://proxy1:8080",
    "http://proxy2:8080",
    "http://proxy3:8080",
]

page = await scrape_with_retry(
    url="https://example.com",
    proxy_list=proxy_list,
    max_retries=3
)
```

#### Supported Proxy Formats

| Protocol | Format |
|----------|--------|
| HTTP | `http://host:port` |
| HTTPS | `https://host:port` |
| SOCKS5 | `socks5://host:port` |
| With auth | `http://user:pass@host:port` |

### Timeout Settings

#### Request Timeout

Set per-request or globally:

```python
# Per-request (in milliseconds)
page = await session.fetch(url, timeout=60000)

# Global (via Settings)
# DEFAULT_TIMEOUT=60 in .env (in seconds)
```

**Recommended values**:
| Scenario | Timeout |
|----------|---------|
| Simple static pages | 15-30s (15000-30000ms) |
| Standard scraping | 30-45s (30000-45000ms) |
| Complex JavaScript | 45-60s (45000-60000ms) |
| Slow/blocked sites | 60-120s (60000-120000ms) |

---

## 8. Security Considerations

### URL Validation

The server implements robust URL validation to prevent Server-Side Request Forgery (SSRF) attacks:

**Allowed**:
- `http://` and `https://` protocols only
- Public IP addresses
- Public domain names

**Blocked**:
- `file://`, `ftp://`, and other protocols
- Private IP addresses (10.x.x.x, 172.16-31.x.x, 192.168.x.x)
- Localhost variants (localhost, 127.0.0.1, ::1)
- Internal hostnames (*.local, *.internal, *.corp)
- Link-local addresses (169.254.x.x)

The validation function `validate_url()` is called automatically before any scraping operation.

### Proxy Security

When using proxies:

1. **Use trusted proxies** - Avoid free/public proxy lists
2. **Encrypt credentials** - Don't hardcode proxy credentials
3. **Validate proxy URLs** - Ensure proxy URLs are valid format
4. **Rotate responsibly** - Don't abuse proxy rotation

### Rate Limiting

To avoid overwhelming target sites:

1. **Use appropriate delays** - Set random_delay between requests
2. **Implement backoff** - Use exponential backoff on failures
3. **Respect robots.txt** - Check and follow site policies
4. **Monitor responses** - Watch for rate limit indicators

### Legal Compliance

**Users are responsible for**:

- Ensuring their scraping activities are legal in their jurisdiction
- Respecting website Terms of Service
- Complying with robots.txt directives
- Not bypassing authentication mechanisms they don't have access to
- Handling personal data appropriately

**Best practices**:
- Only scrape publicly available data
- Identify your scraper in User-Agent when appropriate
- Cache responses to minimize repeated requests
- Consider using official APIs when available

---

## 9. Development Guidelines

### Best Practices for Undetectable Scraping

Follow these best practices to maximize scraping success while minimizing detection:

1. **Always use sessions**: Reuse browser instances to maintain consistent fingerprints
2. **Enable geoip with proxies**: Match browser locale to proxy location for better anonymity
3. **Use solve_cloudflare sparingly**: Only when needed - it increases detection surface and slows down requests
4. **Implement exponential backoff**: Start slow, increase speed gradually on successful requests
5. **Rotate user agents**: Even with Camoufox, periodic rotation helps avoid pattern detection
6. **Monitor for blocks**: Track 403/429 responses and adjust strategy accordingly
7. **Enable network_idle and load_dom**: Wait for page to fully load before extracting data
8. **Use wait_selector for dynamic content**: Wait for specific elements to appear before extraction

#### Recommended Configuration Patterns

```python
# Pattern 1: Simple static pages
simple_config = StealthConfig(
    headless=True,
    disable_resources=True,
    timeout=10000
)

# Pattern 2: Protected sites (Cloudflare)
protected_config = StealthConfig(
    headless=True,
    solve_cloudflare=True,
    humanize=True,
    geoip=True,
    os_randomize=True,
    timeout=60000,
    google_search=True,
    network_idle=True,
    load_dom=True
)

# Pattern 3: High-anonymity scraping
anonymous_config = StealthConfig(
    headless=True,
    block_webrtc=True,
    block_images=True,
    disable_resources=True,
    os_randomize=True,
    geoip=True,
    solve_cloudflare=True,
    humanize=True,
    humanize_duration=1.5,
    proxy=rotation.next()  # Use rotating proxies
)

# Pattern 4: Debugging (visible browser)
debug_config = StealthConfig(
    headless=False,  # Visible browser
    timeout=120000   # Long timeout for manual intervention
)
```

### How to Extend the Server

#### Adding New Tools

To add a new MCP tool, follow this pattern:

```python
from fastmcp import FastMCP
from mcp_scraper.stealth import scrape_with_retry, get_stealth_config

mcp = FastMCP("My Scraper")

@mcp.tool()
async def scrape_with_custom_option(
    url: str,
    custom_option: bool = False
) -> dict:
    """Description of what this tool does.
    
    Args:
        url: The URL to scrape
        custom_option: Description of custom option
        
    Returns:
        Dictionary with scraping results
    """
    # Validate URL
    if not validate_url(url):
        raise ValueError(f"Invalid URL: {url}")
    
    # Get stealth config
    config = get_standard_stealth()
    
    # Apply custom options
    if custom_option:
        # Custom logic
        
    # Scrape
    page = await scrape_with_retry(url, config)
    
    # Format and return
    return format_response(page, url)
```

#### Adding New Stealth Profiles

Add new preset configurations in `config.py`:

```python
@staticmethod
def custom_profile() -> StealthConfig:
    """Custom profile description.
    
    Suitable for: Your specific use case
    """
    return StealthConfig(
        # Custom settings
        enable_js=True,
        # ... other options
    )
```

#### Adding Error Types

Extend the exception hierarchy in `stealth.py`:

```python
class RateLimitError(ScrapeError):
    """Exception raised when rate limited."""
    pass
```

### Testing Approach

**Unit Tests**:
- Test URL validation
- Test configuration classes
- Test response formatting

**Integration Tests**:
- Test scraping with mock servers
- Test retry logic
- Test error handling

**Example test structure**:
```
tests/
├── __init__.py
├── test_config.py
├── test_stealth.py
├── test_validation.py
└── test_integration.py
```

### Code Style

The project follows:

- **Black** for code formatting (100 character line length)
- **Ruff** for linting
- **MyPy** for type checking
- **PEP 8** naming conventions

**Key style rules**:
- Use type hints for all function parameters and return values
- Use dataclasses for configuration objects
- Use async/await for I/O operations
- Use Loguru for logging
- Document all public functions with docstrings

**Pre-commit hooks**:
```bash
pip install pre-commit
pre-commit install
```

---

## 10. Usage Examples

### Basic Scraping Example

Simple scraping of a static webpage:

```python
from mcp_scraper.stealth import scrape_with_retry, format_response

async def basic_example():
    url = "https://example.com"
    
    # Simple scrape
    page = await scrape_with_retry(url)
    
    # Format response
    result = format_response(page, url)
    
    print(f"Title: {result.get('title')}")
    print(f"Text content: {result.get('text')[:500]}")
    print(f"Status: {result.get('status')}")
```

### Stealth Scraping Example

Scraping a protected website with maximum stealth:

```python
from mcp_scraper.stealth import (
    scrape_with_retry,
    get_maximum_stealth,
    format_response
)

async def stealth_example():
    url = "https://protected-site.com/data"
    
    # Use maximum stealth
    config = get_maximum_stealth()
    
    try:
        page = await scrape_with_retry(
            url,
            config=config,
            max_retries=3
        )
        
        result = format_response(page, url)
        print(f"Success! Content length: {len(result.get('html', ''))}")
        
    except Exception as e:
        print(f"Scraping failed: {e}")
```

### Batch Scraping Example

Processing multiple URLs:

```python
from mcp_scraper.stealth import scrape_with_retry, format_response, validate_url
import asyncio

async def batch_example():
    urls = [
        "https://example.com/page1",
        "https://example.com/page2",
        "https://example.com/page3",
    ]
    
    results = []
    delay = 1.0  # Delay between requests
    
    for url in urls:
        # Validate first
        if not validate_url(url):
            print(f"Skipping invalid URL: {url}")
            continue
            
        try:
            page = await scrape_with_retry(url)
            result = format_response(page, url)
            results.append(result)
            print(f"Scraped: {url}")
            
        except Exception as e:
            print(f"Failed to scrape {url}: {e}")
            
        # Delay between requests
        await asyncio.sleep(delay)
    
    print(f"Successfully scraped {len(results)}/{len(urls)} URLs")
    return results
```

### Structured Extraction Example

Extracting specific data using CSS selectors:

```python
from mcp_scraper.stealth import scrape_with_retry, format_response

async def structured_example():
    url = "https://example.com/blogposts"
    
    # Define selectors for data extraction
    selectors = {
        "titles": "h2.post-title",
        "authors": "span.author",
        "dates": "time.published",
        "summaries": "p.summary",
        "links": "a.read-more@href"
    }
    
    # Scrape with selectors
    page = await scrape_with_retry(url, selectors=selectors)
    
    # Get formatted response with extracted data
    result = format_response(page, url, selectors=selectors)
    
    # Access extracted data
    extracted = result.get("selectors", {})
    for i, title in enumerate(extracted.get("titles", [])):
        print(f"Post {i+1}: {title}")
        print(f"  Author: {extracted.get('authors', [None])[i]}")
        print(f"  Date: {extracted.get('dates', [None])[i]}")
```

### Custom Configuration Example

Using custom stealth settings:

```python
from mcp_scraper.stealth import StealthConfig, scrape_with_retry

async def custom_config_example():
    # Create custom stealth configuration
    config = StealthConfig(
        headless=True,
        solve_cloudflare=True,  # Attempt Cloudflare challenges
        humanize=True,
        humanize_duration=1.5,
        geoip=False,
        os_randomize=True,
        block_webrtc=True,
        allow_webgl=True,
        google_search=True,
        block_images=True,  # Reduce bandwidth
        block_ads=True,
        disable_resources=False,
        network_idle=True,
        load_dom=True,
        timeout=45000,
        proxy="http://my-proxy:8080"  # Use specific proxy
    )
    
    url = "https://cloudflare-protected-site.com"
    page = await scrape_with_retry(url, config=config, max_retries=5)
    print(f"Success! Content: {page.text[:200]}")
```

### Proxy Rotation Example

Using multiple proxies with automatic rotation:

```python
from mcp_scraper.stealth import scrape_with_retry, get_standard_stealth

async def proxy_rotation_example():
    # List of proxy servers
    proxy_list = [
        "http://proxy1:8080",
        "http://proxy2:8080", 
        "http://proxy3:8080",
    ]
    
    config = get_standard_stealth()
    
    try:
        page = await scrape_with_retry(
            url="https://example.com",
            config=config,
            max_retries=3,
            proxy_list=proxy_list
        )
        print(f"Success with proxy rotation!")
        
    except Exception as e:
        print(f"All proxies failed: {e}")
```

---

## Appendix: API Reference

### Core Classes

#### `StealthConfig`

Configuration class for stealth web scraping.

**Attributes**:
- `headless` (bool): Run browser in headless mode
- `solve_cloudflare` (bool): Attempt Cloudflare challenges
- `humanize` (bool): Add human-like behavior
- `humanize_duration` (float): Maximum cursor movement duration
- `geoip` (bool): Use geoIP-based routing
- `os_randomize` (bool): Randomize OS fingerprint
- `block_webrtc` (bool): Block WebRTC
- `allow_webgl` (bool): Allow WebGL
- `google_search` (bool): Simulate Chrome
- `block_images` (bool): Block images
- `block_ads` (bool): Block advertisements
- `disable_resources` (bool): Disable CSS/JS
- `network_idle` (bool): Wait for network inactivity
- `load_dom` (bool): Wait for DOMContentLoaded
- `wait_selector` (str): Wait for specific element
- `wait_selector_state` (str): Element state to wait for
- `timeout` (int): Request timeout in milliseconds
- `proxy` (str): Proxy URL

#### `StealthLevel`

Enum for preset stealth levels.

**Values**:
- `MINIMAL`: Fast, minimal protection
- `STANDARD`: Balanced protection
- `MAXIMUM`: Highest protection

### Core Functions

#### `scrape_with_retry(url, config, max_retries, backoff_factor, proxy_list, selectors)`

Scrape a URL with retry logic.

**Parameters**:
- `url` (str): URL to scrape
- `config` (StealthConfig): Stealth configuration
- `max_retries` (int): Maximum retry attempts
- `backoff_factor` (float): Exponential backoff multiplier
- `proxy_list` (list): List of proxy URLs
- `selectors` (dict): CSS selectors for extraction

**Returns**: Page object

**Raises**: `ScrapeError`, `CloudflareError`, `BlockedError`, `TimeoutError`

#### `validate_url(url)`

Validate URL for security.

**Parameters**:
- `url` (str): URL to validate

**Returns**: bool - True if URL is safe

#### `format_response(page, url, selectors)`

Format scraping response.

**Parameters**:
- `page` (Page): Scraped page object
- `url` (str): Original URL
- `selectors` (dict): Optional CSS selectors

**Returns**: dict with response data

#### `get_element_text(element)`

Extract text content from a scraping element with fallbacks.

**Parameters**:
- `element` (Any): A page element object from scrapling

**Returns**: str - The text content of the element

**Description**: Checks for `.text` property first, then `.inner_text`, and falls back to `str()`.

#### `get_element_html(element)`

Extract HTML content from a scraping element with fallbacks.

**Parameters**:
- `element` (Any): A page element object from scrapling

**Returns**: str - The HTML content of the element

**Description**: Checks for `.html` property first, then `.innerHTML`.

#### `get_element_attribute(element, attribute)`

Extract an attribute value from a scraping element with fallbacks.

**Parameters**:
- `element` (Any): A page element object from scrapling
- `attribute` (str): The name of the attribute to retrieve

**Returns**: str | None - The attribute value, or None if not found

**Description**: Checks for `.get_attribute()` method first, then direct property access.

---

## Additional Resources

- [FastMCP Documentation](https://github.com/jlowin/fastmcp)
- [Scrapling Library](https://github.com/D4Vinci/Scrapling)
- [MCP Specification](https://spec.modelcontextprotocol.io/)

---

*This documentation was generated for the Scrapling MCP Server project.*

## Active Technologies
- Python 3.10+ (already configured in pyproject.toml) + FastMCP (MCP framework), Scrapling (scraping engine), Pydantic (config), Loguru (logging) (001-mcp-server-implementation)
- N/A - stateless server with optional session caching in memory (001-mcp-server-implementation)

## Recent Changes
- 001-mcp-server-implementation: Added Python 3.10+ (already configured in pyproject.toml) + FastMCP (MCP framework), Scrapling (scraping engine), Pydantic (config), Loguru (logging)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seszele64)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/seszele64)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
