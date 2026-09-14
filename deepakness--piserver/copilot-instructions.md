## piserver

> One-page site showing this Raspberry Pi's vitals. Caddy serves it on

# piserver

One-page site showing this Raspberry Pi's vitals. Caddy serves it on
127.0.0.1:8080; a Cloudflare Tunnel publishes it as https://pi.deepakness.com.

## Rules

- Commit every change locally with a clear message. Never leave the tree dirty.
- Never commit `bin/`, `state/` or the generated JSON files. See `.gitignore`.
- Caddy rotates the access log (`roll_size` in the Caddyfile) and nothing else
  touches it. Never truncate that file from the collector or a script: Caddy
  keeps writing at its own position, the file becomes sparse and huge, and the
  reader ends up counting nothing while the site looks healthy.
- Version a changed asset by filename (`app-23.js`), not by `?v=`. `/assets/*`
  is cached immutable for a year, so the old URL keeps being served; a new
  filename is a new cache path and is picked up at once, with nothing to purge.
- A new `?v=` value only works if the edge is told about it first: it answers
  403 to any query string it does not recognise, and an asset behind that 403
  takes the page's live numbers down with it. Allowed today: `v=10`, `v=11`,
  `v=22`, `v=25`. Either add the value before deploying, or rename the file.
- The hostname must never appear on the site or in any served file.
- Colours inside generated SVG must be literal values. `stroke="var(--x)"` does
  not resolve in every browser and silently renders nothing.
- Light mode only.
- Copy is short and factual. No filler, no first-person claims.
- Only `stats.json`, `hits.json`, `uptime.json` and `speedtest.json` are public.
  Never expose hostnames, IPs or filesystem paths.
- Do not hardcode a network interface: the collector follows the default route
  so WiFi and Ethernet both work.

## After changing

    systemctl --user restart piserver-caddy      # Caddyfile or site files
    systemctl --user restart piserver-collector  # collector code

Then verify both:

    curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/
    curl -s -o /dev/null -w '%{http_code}\n' https://pi.deepakness.com/

## Counters

`state/hits-state.json` and `state/uptime-state.json` hold the page views and
the sampling history, each with a `.bak`. They flush every 60 s and heal from
the published file if both are gone. Never reset them, and keep them outside
the web root so a deploy cannot wipe them.

## Layout

    caddy/Caddyfile        site config, binds localhost only, writes the access log
    collector/             collector.py (stats, hits, uptime) and speedtest_run.py
    site/                  web root: index.html, assets/, robots.txt, sitemap.xml
    units/                 systemd user units, symlinked into ~/.config/systemd/user/
    state/                 counters and history (not in git)
    bin/                   downloaded binaries (not in git)

---
> Source: [deepakness/piserver](https://github.com/deepakness/piserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
