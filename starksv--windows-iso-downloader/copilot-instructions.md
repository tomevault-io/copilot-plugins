## windows-iso-downloader

> Open-source Windows ISO downloader. React + Go. Live at msdl.tech-latest.com.

# MSDL — Claude Context

## Project

Open-source Windows ISO downloader. React + Go. Live at msdl.tech-latest.com.
Frontend on Cloudflare Pages, backend on Hetzner via Coolify.
Outbound Microsoft API calls route through a Cloudflare Worker (`cloudflare-worker/worker.js`)
controlled by `CF_WORKER_URL` + `CF_WORKER_SECRET` env vars on the backend.

## Conventions

- Small config/one-liner fixes → commit directly to main
- Any meaningful feature or multi-file change → feature branch → PR → merge
- GitHub issues are for bugs and community contributions only — planned features tracked here and in PROGRESS.md
- Commit messages follow conventional commits: `feat:`, `fix:`, `docs:`, `chore:`
- **Always test locally before committing or pushing** — research feasibility before implementing (e.g. file size was planned but Microsoft CDN doesn't return it; caught after commit)

## Backlog (implement when ready, in priority order)

### #12 — Two-layer in-memory caching ✅ DONE (feat/caching-layer, merged)

All 7 items shipped: singleflight, SKU cache (7d TTL), link cache (dynamic TTL from `se` param),
negative cache (60s), dynamic TTL, stale-on-failure, jitter. Verified in production.

### Beyond #12 — Observability & resilience

- [x] **Cache hit/miss logging** — logged on every request (fetched vs cached, cached until timestamp).
- [x] **`/metrics` endpoint** (auth-protected) — exposes cache hit rate, miss count, stale serves. Auth via `METRICS_SECRET` env var. (feat/metrics, merged)
- [x] **README architecture section** — caching layer, `/metrics`, env vars documented.
- [ ] **Per-IP / per-product rate limiter** — deferred; revisit after 1 month of production traffic data.

### Frontend UX improvements ✅ DONE (feat/ux-improvements, merged — v1.2.0)

- [x] **Expiry countdown** — parses `se` param from signed URL where available; falls back to "24 hours" for consumer links (Microsoft doesn't include `se` in consumer CDN URLs).
- [x] **Refresh links** — `?force=true` on `/proxy` bypasses cache, fetches fresh from Microsoft. Button shown when under 6h remaining.
- [x] **CLI command tabs** — wget / curl / aria2 tabs, persists selection in localStorage. Replaces old aria2-only tip. Applied to both consumer and eval pages.
- [x] **Recently viewed** — localStorage only. Shows on homepage. Consumer + eval pages both tracked. Expired state shown when link expiry known.
- ~~**File size**~~ — not feasible. Microsoft CDN does not return file size in the API response.

### Sentinel WAF resilience (ongoing)

Confirmed via three weekly `/metrics` + docker-log checkpoints (2026-07-14 → 07-21 → 07-31):
the backend's own direct link-fetch (`/proxy` → Microsoft) is **100% blocked**, permanently —
`link.ms_fetches: 0` every time, and zero even-attempted-and-failed fetches in the raw logs
(the lockdown gate short-circuits before reaching that code path). The site stays alive
entirely on cached/stale entries plus CLI-contributed links (286 accepted, 0 rejected,
confirmed 2026-07-31). `/skuinfo` and `/evallinks` are unaffected — only the download-link
endpoint is targeted.

The CLI's own residential-IP requests also see a stable ~20% Sentinel rejection rate across
all three checkpoints — not improving, not worsening. Microsoft's own API error response
literally names the system (`"Sentinel marked this request as rejected."`, `Type: 9` in
`Errors[]`), confirming it's a real, named product, not our guess. The clean structured-JSON
deny (not an HTML/JS challenge page) suggests a signature/reputation gate rather than full
interactive bot-management — TLS ClientHello fingerprinting is a plausible contributing
signal, since Go's stdlib `crypto/tls` doesn't look like any real browser, independent of IP.

- [x] **Back off Sentinel retries** — bumped `lockdownTTL` 90min → 5h (`backend/main.go`).
      181 retries over 17 days, 0 successes; retrying that often was pure noise.
      (fix/sentinel-lockdown-and-proxy-validation, merged)
- [x] **Validate `product_id` in `/proxy`** — was passing unknown IDs straight through to a
      real Microsoft session attempt (found via a stray `product_id=2861`, never a real
      product, in the logs). Now rejected with 404 before any outbound call. (same PR, merged)
- [x] **CLI TLS/HTTP2 fingerprint hardening** — swapped the CLI's transport from stdlib
      `net/http` to `github.com/bogdanfinn/tls-client` (wraps `utls` with a maintained Chrome
      profile). Shipped in `cli/v0.3.7` (merged, released 2026-07-31). **Checked in on
      2026-08-17, 17 days post-release: inconclusive-to-negative.** Aggregate CLI
      Sentinel-rejection rate is still ~21%, statistically unchanged from every pre-fix
      checkpoint. Real caveat: mixed-version population (`0.3.6`: 1156 actions vs `0.3.7`: 493
      in that window) means a `0.3.7`-only improvement could still be masked — `/metrics`
      doesn't break the error down by CLI version, so this can't be fully isolated yet. Revisit
      once `0.3.7`+ dominates usage share; if the aggregate still hasn't moved by then, treat
      the fingerprint theory as disproven.
- [ ] **`/needs-warming` community page** — proposed, not built. Surfaces products currently
      failing web users (active Sentinel/rate-limit lockdown, no cached/stale link available)
      with a one-click CLI command to fix it. See
      `docs/superpowers/specs/2026-07-13-needs-warming-design.md`.
- [ ] **Per-language SHA256 checksums on product pages** — Microsoft's own download pages
      (e.g. `/software-download/windows11`) publish a static hash table (one SHA256 per
      locale) for the *current* ISO build, confirmed live via direct network inspection
      2026-08-18. It's not in any API response MSDL/CLI call (`GetProductDownloadLinksBySku`
      never includes a hash, confirmed against Microsoft's own live page too) — it's static
      HTML on the Windows-version download page, tied to the product/build, not the session.
      Since it only changes when a product ID is replaced (same trigger as adding a new
      catalog entry), no scraper/refresh job needed: copy the hash table by hand at the same
      time a new product ID is added, per the existing "Adding a new consumer Windows release"
      steps in CONTRIBUTING.md. Open questions before building: where to store it
      (`products.json` field vs. separate file), which languages to cover (all 38 vs. just
      the popular ones), and where to surface it (collapsible "Verify your download" section
      on the product page, mirroring Microsoft's own UX; maybe CLI output too).

### Known bugs / open issues

- [x] **`_redirects` Cloudflare Pages** — investigated 2026-07-21. The rule was indeed ignored
      (Cloudflare's deploy log flags it as a false-positive infinite loop), but this turned out
      to be a non-issue: Cloudflare Pages' own default fallback already serves `index.html` for
      any unmatched path, so the explicit rule was redundant. Verified live: direct navigation to
      `/about`, a dynamic route (`/product/3262`), and a genuinely invalid path all return HTTP 200
      with the correct content (including React Router's own 404 page for the invalid one). Removed
      `frontend/public/_redirects` entirely to stop the recurring deploy warning.

---
> Source: [starkSV/windows-iso-downloader](https://github.com/starkSV/windows-iso-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
