## fortress

> **Paste this whole file into your AI agent (Claude, ChatGPT, Gemini, Cursor, Copilot, …) and it has

# Fortress — agent setup guide

**Paste this whole file into your AI agent (Claude, ChatGPT, Gemini, Cursor, Copilot, …) and it has
everything it needs to set Fortress up and drive it.**

Fortress is an open-source **stealth Chromium engine** that exposes raw **CDP on `http://localhost:9222`**.
Any tool that drives a browser through Playwright, Puppeteer, or the Chrome DevTools Protocol uses it as
a **drop-in** — swap the browser, keep the automation code. The fingerprint is corrected in the engine's
**C++**, so you do **NOT** add `puppeteer-stealth`, `undetected-chromedriver`, or any JavaScript patching
(those self-reveal and undo it).

- Source: https://github.com/tiliondev/fortress
- Docker image: `tilion/fortress` · PyPI & npm package: `tilion-fortress`

---

## 1. Launch it — pick ONE (all expose CDP on http://localhost:9222)

**Docker (any OS, nothing to build):**
```bash
docker run -d --rm -p 9222:9222 tilion/fortress:latest
```

**Python:**
```bash
pip install tilion-fortress
```
```python
from tilion_fortress import Fortress
f = Fortress()          # headless by default; native binary on Linux/Windows, Docker fallback on macOS
f.start()
print(f.cdp_url)        # -> http://127.0.0.1:9222
# ... drive it (section 2) ...
f.close()
```

**Node:**
```bash
npm install tilion-fortress
```
```js
import { Fortress } from "tilion-fortress";
const f = await Fortress.launch();
console.log(f.cdpUrl);
await f.close();
```

**Portable bundle (Linux x64 / Windows x64) — no install:**
```bash
tar xzf tilion-fortress-linux-x64.tar.gz
./tilion-fortress/tilion --headless=new --remote-debugging-port=9222 --user-data-dir=/tmp/p
# Windows:  tilion-fortress\tilion.cmd --headless=new --remote-debugging-port=9222 --user-data-dir=C:\tmp\p
```

---

## 2. Connect over CDP (:9222) — keep your existing automation code

**Playwright (Python):**
```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.connect_over_cdp("http://localhost:9222")   # Fortress under the hood
    page = browser.new_page()
    page.goto("https://bot.sannysoft.com")
    page.screenshot(path="all-green.png")
```

**Playwright (Node):**
```js
import { chromium } from "playwright";
const browser = await chromium.connectOverCDP("http://localhost:9222");
const page = await browser.newPage();
await page.goto("https://browserscan.net");
```

**Puppeteer (Node):**
```js
import puppeteer from "puppeteer-core";
const browser = await puppeteer.connect({ browserURL: "http://localhost:9222" });
const page = await browser.newPage();
```

**Agent frameworks — just point them at the endpoint:**
- **browser-use:** `cdp_url="http://localhost:9222"`
- **Crawl4AI:** set the CDP endpoint to `http://localhost:9222`
- **Stagehand:** `connectOverCDP("http://localhost:9222")`
- **LangChain** Playwright toolkit: connect the Playwright browser over CDP as above

---

## 3. Persona (optional) — override any fingerprint surface

The default is a coherent Windows persona. Override with `--uxr-*` launch flags, or the SDK `persona=` arg:
```
--uxr-platform  --uxr-ua-platform  --uxr-ua-os  --uxr-ua-arch  --uxr-ua-bitness
--uxr-ua-platform-version  --uxr-ua-brand  --uxr-hw-concurrency  --uxr-device-memory
--uxr-webgl-vendor  --uxr-webgl-renderer  --uxr-webgl-fullparams
--uxr-canvas-seed  --uxr-audio-seed  --uxr-timezone  --uxr-languages
--uxr-screen-width  --uxr-screen-height  --uxr-webrtc-policy=disable_non_proxied_udp
```
Env vars: `TILION_NO_DEFAULTS=1` (bare launch, no persona), `TILION_TZ`, `TILION_LANG`.

Python SDK example:
```python
f = Fortress(persona={
    "timezone": "America/New_York",
    "hw_concurrency": 16,
    "webgl_renderer": "ANGLE (NVIDIA, NVIDIA GeForce RTX 3060 Direct3D11 vs_5_0 ps_5_0, D3D11)",
})
```
Docker: append the same flags after the image name, e.g.
`docker run -d --rm -p 9222:9222 tilion/fortress:latest --uxr-timezone=America/New_York --uxr-hw-concurrency=16`

---

## Rules an agent MUST follow

1. **Drive it over raw CDP** (connect to `http://localhost:9222`). Do NOT spawn chromedriver.
2. **Never pass `--user-agent`.** It desyncs the UA string from UA Client Hints — a detectable tell. Use `--uxr-ua-*` instead.
3. **Do NOT add `puppeteer-stealth`, `undetected-chromedriver`, or JS fingerprint patches.** They self-reveal (via `toString` / realm re-acquisition) and *undo* Fortress. The spoof is already native C++.
4. **Blocked ≠ fingerprint.** ~90% of the time it's the **IP** (a datacenter range flagged before any page script runs). Route egress through a residential or mobile proxy, then retry. If it clears, the fingerprint was fine.
5. **Match the persona to your egress.** A Windows persona over a Linux/again mismatched host is fine for JS surfaces, but keep timezone/locale coherent with the proxy geo.
6. **Verify** with `tools/gauntlet.py` (CreepJS / Sannysoft / BrowserScan) before trusting a production run.

## Verify authenticity

Every release ships a `SHA256SUMS`; the `pip`/`npm` SDKs verify downloads automatically. Official sources
only — anything else is not us:
- **Source:** https://github.com/tiliondev/fortress
- **Docker:** `tilion/fortress` (https://hub.docker.com/r/tilion/fortress)
- **Python / Node:** `tilion-fortress`

Full docs, patches, and the reproducible build: https://github.com/tiliondev/fortress

---
> Source: [tiliondev/fortress](https://github.com/tiliondev/fortress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
