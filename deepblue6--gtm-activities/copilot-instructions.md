## gtm-activities

> This repo contains two self-hosted scraping tools. Users cloning this are often non-technical or semi-technical. Your job is to get them up and running with minimal friction.

# CLAUDE.md — Guide for assisting users with this repo

This repo contains two self-hosted scraping tools. Users cloning this are often non-technical or semi-technical. Your job is to get them up and running with minimal friction.

## What this repo does

1. **website-crawler/** — A high-speed website text extractor (like Firecrawl but self-hosted). It takes a URL and returns the page's text content. Uses TLS fingerprinting for speed, falls back to real browsers for protected sites.

2. **searxng-search/** — A self-hosted Google search API. Runs SearXNG in Docker with proxy rotation so you can do unlimited Google searches programmatically.

## Prerequisites users need BEFORE setup

- **Node.js 18+** — check with `node --version`
- **Python 3.8+** — check with `python3 --version`
- **Docker** — check with `docker --version` (must be running, not just installed)
- **Proxies** — residential proxies in the format specified in each tool's `proxies.txt.example`

If a user doesn't have these, walk them through installing:
- Node: `brew install node` (Mac) or download from nodejs.org
- Python: `brew install python3` (Mac) or python.org
- Docker: Docker Desktop from docker.com (they need to open the app after installing)

## Setup flow

### Website Crawler

```bash
cd website-crawler
npm install
npx camoufox fetch
cp proxies.txt.example proxies.txt
# user adds their proxies here
node server.js
```

### SearXNG Search

```bash
cd searxng-search
pip install httpx
cp proxies.txt.example proxies.txt
# user adds their proxies here
python3 setup.py
python3 search.py
```

## Common issues and fixes

### "npm install fails" / node-gyp errors
- This is usually because `tlsclientwrapper` or `camoufox-js` need native compilation
- Fix: Make sure Xcode CLI tools are installed: `xcode-select --install` (Mac)
- On Linux: `sudo apt install build-essential python3`

### "npx camoufox fetch" hangs or fails
- It downloads a ~80MB Firefox binary. Needs internet access.
- If it fails, retry. If behind a corporate proxy, it won't work — download on personal network.

### "Cannot find module" errors when running server.js
- User forgot `npm install`. Run it again.
- If cheerio/camoufox-js/tlsclientwrapper still missing, try: `rm -rf node_modules && npm install`

### "Missing proxies.txt" error
- User forgot to create it. They need to: `cp proxies.txt.example proxies.txt` then paste their proxies in.
- Format for website-crawler: `ip:port:username:password` (one per line, no http:// prefix)
- Format for searxng-search: `http://username:password@ip:port` (one per line, WITH http:// prefix)

### "Docker not running" / "Cannot connect to Docker daemon"
- User needs to open Docker Desktop app (Mac/Windows) or start the daemon (`sudo systemctl start docker` on Linux)
- Verify with: `docker info`

### "Port 8080 already in use" (SearXNG)
- Something else is on port 8080. Either stop it or change the port:
  - `docker run -d -p 9090:8080 ...` (in setup.py, edit the port mapping)
  - Then change SEARXNG in search.py to `http://localhost:9090`

### "Port 9004 already in use" (Website Crawler)
- Run with a different port: `PORT=9005 node server.js`

### SearXNG returns 0 results
- Proxies might be dead/blocked. Test one manually: `curl -x http://user:pass@ip:port https://www.google.com`
- Check container logs: `docker logs searxng`
- Restart: `docker restart searxng`

### Website crawler returns 202 for everything
- This means TLS tier is failing and everything is queuing for browsers
- Check proxy connectivity — if all proxies are dead, nothing will work
- Look at the console logs for error messages

### "ECONNREFUSED" when curling the crawler
- Server isn't running. Start it: `cd website-crawler && node server.js`
- Or wrong port — check what PORT it printed on startup

## How to help users who are stuck

1. First check: do they have the prerequisites installed? (`node --version`, `python3 --version`, `docker --version`)
2. Did they create `proxies.txt` with actual proxies? (not the .example file)
3. Are the proxies in the right format for each tool?
4. Is Docker running? (for SearXNG only)
5. Read the error message — it usually says exactly what's wrong

## Proxy sourcing

Users often ask "where do I get proxies?" — this repo doesn't provide proxies. They need to buy residential proxies from a provider. Common ones:
- Bright Data, Oxylabs, Smartproxy, IPRoyal, Webshare

The format they get from their provider needs to be converted to match the `proxies.txt.example` format. Most providers give you a list in `ip:port:user:pass` format which works directly for the website crawler. For SearXNG, prepend `http://user:pass@` and restructure as `http://user:pass@ip:port`.

## Architecture notes (for debugging)

**Website Crawler:**
- Tier 1 uses `tlsclientwrapper` which spawns worker threads for TLS connections
- Tier 2 uses `camoufox-js` (a Playwright-compatible Firefox fork with anti-detection)
- If Tier 1 gets a Cloudflare challenge or too-short content, it queues for Tier 2
- Tier 2 is async — returns 202 immediately, caches result for next request

**SearXNG Search:**
- Just a Docker container running the open-source SearXNG metasearch engine
- Proxies are injected into its config so it rotates them per-request
- The Python client is a thin wrapper around its JSON API with local file caching

---
> Source: [deepblue6/gtm-activities](https://github.com/deepblue6/gtm-activities) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
