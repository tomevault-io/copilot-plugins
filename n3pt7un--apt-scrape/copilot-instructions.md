## apt-scrape

> MCP server for scraping Italian rental/sale property listings from multiple real estate portals (Immobiliare.it, Casa.it, and extensible to others) using Camoufox (anti-detection browser) and BeautifulSoup. Plugin architecture — each site is a self-contained adapter with its own URL templates, query param mappings, and CSS selector chains. Use this skill whenever the user wants to search for apartments, houses, or properties in Italy, scrape real estate listings, compare rental prices across Italian cities, or extract property data from Italian portals. Triggers on mentions of immobiliare, casa.it, Italian rentals, affitto, vendita, apartment hunting in Italy, or property scraping.


# Rental Scraper MCP Server

A local MCP server (stdio transport) that scrapes Italian real estate listings using Camoufox for anti-bot evasion and BeautifulSoup for HTML parsing.

## Architecture

```
rental-scraper-mcp/
├── server.py              # Thin MCP shell — tools + browser manager
├── cli.py                 # Standalone CLI (search, detail, dump, sites)
├── sites/
│   ├── __init__.py        # Adapter registry (auto-discovery, lookup)
│   ├── base.py            # Abstract SiteAdapter + SiteConfig + SelectorGroup
│   ├── immobiliare.py     # Immobiliare.it adapter
│   ├── casa.py            # Casa.it adapter
│   └── _template.py       # Copy this to add a new site
├── requirements.txt
└── mcp_config_example.json
```

## Plugin System

Each site is a **SiteAdapter** subclass with a **SiteConfig** dataclass. The config holds:

- URL template + query param mappings (how filters become URL parameters)
- CSS selector chains with fallbacks (how HTML becomes structured data)
- Property type / operation slug mappings
- Wait selectors for the stealth browser

To add a new site: copy `sites/_template.py`, fill in the config, register in `sites/__init__.py`.

## Tools

| Tool | Description |
|------|-------------|
| `rental_search_listings` | Search with filters → structured listing summaries |
| `rental_get_listing_detail` | Fetch one listing URL → full detail extraction |
| `rental_list_sites` | Show all registered site adapters |
| `rental_dump_page` | Dump raw HTML for debugging selectors |

## Setup

```bash
pip install -r requirements.txt
camoufox fetch
```

### Claude Code
```json
{
  "mcpServers": {
    "rental-scraper": {
      "command": "python",
      "args": ["server.py"],
      "cwd": "/path/to/rental-scraper-mcp"
    }
  }
}
```

---
> Source: [n3pt7un/apt_scrape](https://github.com/n3pt7un/apt_scrape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
