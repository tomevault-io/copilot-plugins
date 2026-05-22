## cliniciq

> > **Scope**: Project-specific identity and background.

# ClinicIQ Solutions Website

> **Scope**: Project-specific identity and background.  
> **Override**: Defers to global `AGENTS.md` (`~/AGENTS.md`) and global `CLAUDE.md` (`~/.claude/CLAUDE.md`) for universal principles.  
> **Karpathy**: Global Karpathy Priority Doctrine applies — safety > speed, simplicity first, surgical changes.

---

## Project Identity

**ClinicIQ Solutions Website** — A business automation solutions website for Australian healthcare clinics. Production-ready static site deployed on Netlify with Cloudflare DNS.

## Tech Stack

- **Frontend**: HTML5, CSS3, Vanilla JavaScript
- **Styling**: Custom CSS with luxury color palette
- **Dependencies**: Google Fonts (Inter, Playfair Display)
- **Build**: None (vanilla static files)
- **Deployment**: Netlify (static hosting)

## Infrastructure & Deployment Architecture

```
User Browser
    ↓
Cloudflare DNS (DNS-only, NOT proxied)
    ↓
Netlify CDN (handles EVERYTHING)
    ↓
Static Site (HTML, CSS, JS)
```

### Component Responsibilities

| Component | Provider | Purpose | Mode |
|-----------|----------|---------|------|
| **Hosting** | Netlify | SSL/TLS, CDN, edge caching, analytics, optimizations | Primary (FULL CONTROL) |
| **DNS** | Cloudflare | DNS resolution only | DNS-only (gray cloud ☁️) |
| **Nameservers** | VentraIP | DNS management | Account-based |
| **Email** | VentraIP | MX records, email hosting | Via Cloudflare DNS |

### Critical Rules

1. **Let Netlify handle everything**: SSL/TLS, CDN edge caching, automatic optimizations, analytics, Cache-Control headers, Netlify Functions
2. **Keep Cloudflare in DNS-only mode** (gray cloud ☁️) — NO orange cloud proxy mode
3. **Use browser caching via `_headers` file**:
   - HTML: `max-age=3600` (1 hour)
   - Static assets: `max-age=31536000, immutable` (1 year)
4. **NEVER enable ServiceWorker** — intentionally disabled to prevent bypassing Netlify CDN
5. **NEVER add caching layers that bypass Netlify** — no CDN proxies, no reverse proxies, no edge workers

### DNS Configuration

```
Domain: cliniciq.com.au
Nameservers: VentraIP account → Cloudflare DNS
Mode: DNS-only (gray cloud ☁️)

Records:
A       cliniciq.com.au     [Netlify IP]                   DNS only (gray)
CNAME   www                 [Netlify domain].netlify.app   DNS only (gray)
MX      cliniciq.com.au     mail.ventraip.com.au           N/A
```

## Performance Optimizations

### Cache Strategy (`_headers` file)
```nginx
/*.html
  Cache-Control: public, max-age=3600, must-revalidate

/*.js, /*.css, /*.jpg, /*.png, /*.webp, /*.svg, /*.woff2
  Cache-Control: public, max-age=31536000, immutable
```

### Critical Rendering Path (LCP < 2.5s)
- Preload critical resources (CSS, hero images, fonts)
- Font-display swap for web fonts
- Defer non-critical JavaScript
- Progressive component initialization (critical first, defer non-critical)

### Performance Metrics (Current)

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| **LCP (First visit)** | < 2.5s | 1.5-2.5s | ✅ Achieved |
| **LCP (Repeat, < 1hr)** | < 0.5s | < 0.1s | ✅ Exceeded |
| **FCP** | < 1.8s | < 1s | ✅ Achieved |
| **CLS** | < 0.1 | 0.0 | ✅ Perfect |
| **TTI** | < 3.8s | < 3s | ✅ Achieved |
| **Browser Cache Hit** | > 80% | ~95% | ✅ Exceeded |

## Image Optimization Rules

1. **Explicit dimensions** on ALL `<img>` tags (prevents CLS)
2. **Loading strategy**: Above-the-fold = `loading="eager"`, Below-the-fold = `loading="lazy"`, LCP candidates = `fetchpriority="high"` + `decoding="async"`
3. **WebP first**, PNG/JPG fallback in `<picture>`
4. **Accurate `sizes` attribute** for responsive images
5. **No preload on non-critical images** — only preload CSS, hero images, fonts
6. **Logo**: `photos/branding/cliniciq-logo copy.webp` (84 KB, 1600×900px, displayed at 80×45px)

## Security Headers (via `_headers`)

```nginx
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  X-XSS-Protection: 1; mode=block
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  Strict-Transport-Security: max-age=31536000; includeSubDomains
```

## ServiceWorker Status: DISABLED

There is **no `sw.js` file** in the project. `script.js` unregisters any previously installed workers.

**Only re-enable if** offline functionality is absolutely required AND you accept Netlify Analytics accuracy loss.

## Key Files for Infrastructure

| File | Purpose | Critical Settings |
|------|---------|-------------------|
| `_headers` | Netlify cache headers | Cache-Control rules, security headers |
| `_redirects` | Netlify redirects/rewrites | API endpoint routing to Netlify Functions |
| `netlify.toml` | Netlify build config | Mirrors `_headers` rules for Netlify UI |
| `index.html` | Main HTML | Cache-Control meta tag, preload hints |
| `script.js` | Main JavaScript | Unregisters legacy service workers, defers non-critical components |

## Deployment Process

1. Push to main branch
2. Netlify detects changes and auto-deploys
3. Applies `_headers` and `_redirects` rules
4. Deploys Netlify Functions, invalidates CDN cache
5. Site live at: cliniciq.com.au

### Post-Deploy Verification
```bash
# Check Cache-Control headers
curl -I https://cliniciq.com.au/

# Check static asset headers
curl -I https://cliniciq.com.au/styles.css

# Check ServiceWorker status (DevTools → Application → Service Workers)
# Should show: "No service workers registered"
```

## Color Palette

- **Primary Green**: #2C4A3C (luxury nature theme)
- **Gold Accent**: #C4A661 (premium luxury)
- **Cream Background**: #F5F1E6 (elegant neutral)

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

---
> Source: [jsaenz03/cliniciq](https://github.com/jsaenz03/cliniciq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
