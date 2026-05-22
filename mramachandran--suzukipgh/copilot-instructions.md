## suzukipgh

> A static HTML/CSS/JS website for the Suzuki Association of Pittsburgh, a small nonprofit music education organization. The site is hosted on **GitHub Pages** with an **AI-powered admin interface** hosted on **Netlify** that lets non-technical admins update the site in plain English.

# Suzuki Association of Pittsburgh — Website

## What This Project Is

A static HTML/CSS/JS website for the Suzuki Association of Pittsburgh, a small nonprofit music education organization. The site is hosted on **GitHub Pages** with an **AI-powered admin interface** hosted on **Netlify** that lets non-technical admins update the site in plain English.

---

## Live URLs

| Purpose | URL |
|---|---|
| Live website | `https://mramachandran.github.io/suzukipgh/` |
| Admin interface | `https://<your-netlify-site>.netlify.app/admin` |
| GitHub repo | `https://github.com/mramachandran/suzukipgh` |

---

## How the System Works

```
Admin types a request  →  Netlify function (admin-ai.js)  →  Claude AI edits HTML
                                                                       ↓
        Site updates live  ←  GitHub Pages deploys  ←  Commit pushed to GitHub
```

1. Admin visits the `/admin` page on Netlify, enters the admin password
2. They type a plain-English request (e.g. "Add an event on June 5 at 2pm...")
3. The Netlify serverless function (`netlify/functions/admin-ai.js`) fetches the current HTML from GitHub, sends it to Claude with the request, and gets back a JSON response with the updated file content
4. The function commits the changes back to GitHub
5. GitHub Pages auto-deploys — site is live in ~2 minutes

---

## File Structure

```
suzukipgh/
├── index.html              # Homepage (testimonials, welcome, announcements)
├── about.html              # About the Suzuki Method
├── teachers.html           # Teachers directory with contact info
├── events.html             # Events calendar (most frequently updated)
├── board.html              # Board members
├── programs.html           # Programs offered
├── faq.html                # FAQ accordion
├── resources.html          # Downloadable guides and links for parents
├── contact.html            # Contact page with Google Maps
├── admin.html              # AI-powered admin interface (password protected)
├── 404.html                # Custom error page
├── styles.css              # Main stylesheet
├── css/styles.css          # Enhanced stylesheet
├── js/main.js              # Main JavaScript (nav, forms, back-to-top, etc.)
├── images/
│   ├── logo.jpg            # Organization logo
│   └── logo.svg            # SVG logo
├── favicon.svg             # Site favicon
├── sitemap.xml             # SEO sitemap
├── robots.txt              # Search engine directives
├── netlify.toml            # Netlify config (functions dir, redirects, headers)
├── netlify/
│   └── functions/
│       └── admin-ai.js     # Serverless function: AI + GitHub integration
└── docs/
    ├── README.md           # Overview and setup instructions
    ├── DEPLOYMENT.md       # GitHub Pages deployment steps
    ├── ADMIN-SETUP.md      # Admin interface one-time setup guide
    ├── UPDATING-EVENTS.md  # How to update events manually
    ├── GITHUB-PAGES-SETUP.md
    └── QUICK-START.md
```

---

## Tech Stack

- **Frontend**: Pure HTML5, CSS3, vanilla JavaScript — no frameworks, no build step
- **Fonts**: Crimson Pro (headings) + Work Sans (body) via Google Fonts
- **Hosting**: GitHub Pages (static site)
- **Admin backend**: Netlify Functions (Node.js serverless)
- **AI**: Claude API (`claude-haiku-4-5-20251001`) via Anthropic — Haiku is fast and more than capable for the surgical-op workload. Don't upgrade to Sonnet without a specific reason; Haiku keeps response times short.
- **Version control**: Git / GitHub (`mramachandran/suzukipgh`)

---

## Environment Variables (Netlify)

Set these in Netlify → Site configuration → Environment variables:

| Variable | Description |
|---|---|
| `ADMIN_PASSWORD` | Password for the `/admin` login screen |
| `ANTHROPIC_API_KEY` | Anthropic API key for Claude |
| `GITHUB_TOKEN` | GitHub Personal Access Token with `repo` scope |
| `GITHUB_REPO` | Repo in `owner/name` format — e.g. `mramachandran/suzukipgh` |

### Netlify config gotchas

- **Function timeout is NOT configurable via `netlify.toml`.** It's determined by the Netlify plan (≈10s Free, 26s Pro, up to 30s higher tiers). Do **not** add `timeout = 26` (or similar) under `[functions]` — it's invalid syntax and will fail every deploy at the config-parse stage, silently leaving the last successful deploy live. We've been bitten by this before.
- With the ops-based admin-ai rewrite, actual function time is ~5–8s, so no extension is needed.
- Keep the `[functions]` table minimal or absent. Valid keys are things like `node_bundler`, `external_node_modules`, `included_files` — not `timeout`.

### Custom domain + SSL

- `suzukipittsburgh.org` DNS points correctly to Netlify (A → 75.2.60.5), but the Let's Encrypt cert for the custom domain has not been provisioned yet — TLS requests get a `*.netlify.app` cert, which fails SAN validation in `curl`. Browsers mostly paper over this.
- Fix when ready: Netlify dashboard → Domain management → add/verify `suzukipittsburgh.org` → provision certificate.
- Until fixed, **test curl / API calls against the default `<site>.netlify.app` URL** — it has a valid cert and is the source of truth for the function.

---

## Design Tokens (CSS Variables)

Defined in `css/styles.css`:

```css
:root {
    --primary-color: #2C5F7C;     /* Main blue */
    --secondary-color: #8B4789;   /* Purple accent */
    --accent-color:  #D4A574;     /* Gold accent */
}
```

Admin UI uses its own color scheme: dark green `#1a3a2a` / `#2d5a3d`.

---

## Pages & What They Contain

**index.html** — Homepage: hero banner, welcome message, featured events preview, testimonials from parents/students, newsletter signup form.

**events.html** — Full events calendar. Each event uses `.event-item` cards with `data-category` attributes (`recital`, `workshop`, `masterclass`, etc.) for filtering. Events should be kept in chronological order.

**teachers.html** — Teacher directory. Each teacher uses a `.teacher-card` with name, instrument, bio, and contact info.

**about.html** — Background on the Suzuki Method philosophy.

**programs.html** — Programs and instruments offered.

**board.html** — Board member listing (update annually).

**faq.html** — Accordion-style FAQ for prospective families.

**resources.html** — Links and downloadable guides for enrolled families.

**contact.html** — Contact form + Google Maps embed.

**admin.html** — Password-protected AI chat interface. Talks to `/.netlify/functions/admin-ai`. The `FUNCTION_URL` constant in the script is set to `/.netlify/functions/admin-ai` (relative — works because both the page and the function are on Netlify).

---

## Admin Interface — How the AI Function Works

File: `netlify/functions/admin-ai.js`

1. **Auth**: Checks `password` in request body against `ADMIN_PASSWORD` env var.
2. **Ping**: A `__ping__` message is used at login just to validate the password (no Claude or GitHub calls — returns in <1s).
3. **Smart file loading**: Keyword-matches the admin's request to decide which HTML files to fetch from GitHub. Always loads `events.html`; only adds `teachers.html` / `index.html` / `about.html` / `programs.html` / `contact.html` when the message mentions those areas. Keeps input tokens small.
4. **Call Claude** (model: `claude-haiku-4-5-20251001`): The system prompt asks for **surgical operations, not full-section rewrites**. For each change, Claude emits ONE of:
   - `{"op":"add","section":"EVENTS_LIST","item":"<div class='event-item'...>...</div>"}` — one item only
   - `{"op":"remove","section":"EVENTS_LIST","match":"Spring Recital"}` — case-insensitive substring match against the `<h3>` text
   - `{"op":"edit","section":"EVENTS_LIST","match":"...","item":"..."}` — match + full replacement item

   This design keeps Claude's output to ~200 tokens per change instead of ~5000 for a full-list rewrite. That's the difference between a 30s+ timeout and a 5–8s response. **Do not revert to the "return the whole list" approach** — it will time out on any decent-sized events list.
5. **Apply ops server-side**: The function splits the marked section into items (tracking `<div>` depth so nested divs inside each item work; handles both `'` and `"` quoting), applies the op, sorts events chronologically by parsing the date badge (`<span class='month'>`…), and rebuilds the section. See `splitItems`, `itemTitle`, `eventDateMs`, `applyOp`, and the `SECTION_CONFIG` map in `admin-ai.js`.
6. **Commit**: Uses GitHub Git Data API to create a new commit on `main` with the changed files (blob → tree → commit → ref update). Reuses already-fetched content — no second round-trip to GitHub.
7. **Return**: Sends commit URL and deploy URL back to the admin UI.

The admin UI (`admin.html`) stores the session password in `sessionStorage` and maintains a chat history (last 8–10 messages) sent with each request for context.

### Managed sections (in `SECTION_CONFIG`)

| Section | File | Item selector | Sort |
|---|---|---|---|
| `EVENTS_LIST` | `events.html` | `<div class="event-item"...>` | By parsed date badge (chronological) |
| `TEACHERS_LIST` | `teachers.html` | `<div class="teacher-card">` | Insertion order |

To add a new managed section: add a `SECTION_CONFIG` entry (file, item-open regex, sort fn) and wrap the target block in `<!-- SECTION_NAME_START -->` / `<!-- SECTION_NAME_END -->` markers in the HTML file.

### Back-compat

The resolver still accepts the legacy `{"section":"...","content":"<full section>"}` shape as a fallback. If Claude ignores the prompt and emits a full list, the commit still works — just slower. This is a safety net, not a first-class path.

---

## Most Common Update Tasks

### Adding or Changing Events
Edit `events.html` — use existing `.event-item` structure:
```html
<div class="event-item" data-category="recital">
    <div class="event-date">...</div>
    <div class="event-details">
        <h3>Event Title</h3>
        <p>Description</p>
    </div>
</div>
```
Or just use the admin interface at `/admin`.

### Updating Teacher Info
Edit `teachers.html` — find the teacher's `.teacher-card` div and update name, bio, phone, email.

### Adding a New Page
1. Copy an existing HTML file
2. Update the `<title>` and content
3. Add the nav link in **all** HTML files (navbar section)
4. Add to footer links in all HTML files
5. Add to `sitemap.xml`

### Updating Social Media Links
Search all HTML files for `facebook.com/suzukipgh` and `instagram.com/suzukipgh` — update as needed.

---

## Deployment Flow

**Making changes via GitHub web UI:**
1. Go to `https://github.com/mramachandran/suzukipgh`
2. Click a file → pencil icon → edit → "Commit changes"
3. GitHub Pages rebuilds in ~2 minutes

**Making changes locally:**
```bash
# Windows
cd "C:\Users\Manoj Ramachandran\OneDrive\Documents\suzuki-website"
# Mac (Manojs-Mac-mini)
cd /Users/zebra/projects/suzukipgh

git add <files>
git commit -m "Description of changes"
git push origin main
```

**Checking deploy status:**
- Site content (GitHub Pages): https://github.com/mramachandran/suzukipgh/actions
- Admin function (Netlify): Netlify dashboard → Deploys. **Always verify a push actually built** — Netlify silently keeps the last good deploy live when a build fails, so broken code can appear to "not change anything" when it's really just not deployed. If the site seems stuck on old behavior, check here first.

---

## Maintenance Checklist

- **Events**: Update `events.html` before/after each event — remove past events, add upcoming
- **Teachers**: Keep contact info and bios current
- **Board**: Refresh `board.html` annually
- **Footer copyright**: Auto-updates via JS (no action needed)
- **GitHub token**: Expires per the expiration set at creation — renew before it expires and update the Netlify env var

---

## Things Still Pending / To Do

- [ ] Enable GitHub Pages (Settings → Pages → Source: main branch)
- [ ] Replace placeholder/sample events with real upcoming events
- [ ] Replace placeholder teacher info with real teacher details
- [ ] Replace sample board member info with real board members
- [ ] Add real organization logo (replace `images/logo.jpg`)
- [ ] Update social media links if handles differ
- [ ] **Provision SSL cert for `suzukipittsburgh.org`** on Netlify (Domain management → verify domain → provision Let's Encrypt cert). DNS is already pointed correctly; the cert step just hasn't been done. Until then, any API/curl testing must use the default `.netlify.app` URL.
- [ ] (Optional) Set up a custom domain (e.g. `suzukipgh.org`) in GitHub Pages settings
- [ ] (Optional) Add Google Analytics or similar tracking
- [ ] (Optional) Hook up the contact form to a real email service (Formspree, etc.)
- [ ] **(Long-term architecture)** Migrate `events.html` and `teachers.html` to JSON data files with client-side rendering. The admin AI would emit JSON deltas instead of HTML items, removing all the HTML parsing/quoting fragility, enabling filter/search UI for free, and making manual edits trivial. Requires refactoring those two pages and updating `admin-ai.js` to produce JSON ops. Roughly a weekend of work; not urgent now that the ops-based approach has unblocked the timeout.

---

## Troubleshooting

**Changes not showing up on live site** → First check that the deploy actually succeeded — don't assume. Site content: https://github.com/mramachandran/suzukipgh/actions. Admin function: Netlify dashboard → Deploys. Then hard-refresh (`Ctrl+F5` / `Cmd+Shift+R`) to bypass CDN cache.

**Builds silently not deploying / "my changes aren't taking effect"** → Usually caused by an invalid `netlify.toml` that fails the config-parse stage *before* the build runs. Netlify keeps the last successful deploy live and emails on failure, but to the user it looks like "same old behavior." Check Netlify dashboard → Deploys for a red "Failed" row. Most common culprit: adding unsupported keys under `[functions]` (see "Netlify config gotchas"). Fix the TOML, push again, watch for a green deploy.

**Admin times out / function returns `Sandbox.Timedout` after 30s** → Lambda hit its wall-clock limit. Almost always means the Claude call generated too much output. The current design should stay well under 10s because Claude emits surgical ops, not full-list rewrites. If you see this, check:
1. Has someone reverted the prompt to "return the whole list"? See `buildSystemPrompt` in `admin-ai.js` — it must describe `op` / `item` / `match`, not `content` for the whole section.
2. Is the events list unusually large (100+ items)? Even the ops path will slow down if the *input* section is huge. Consider the JSON-data refactor in the Pending list.

**`curl` returns an HTML "Inactivity Timeout" page (instead of JSON)** → Not from Netlify's Lambda. Likely a middlebox between the client and Netlify (CDN, proxy, corporate firewall, or a misconfigured custom domain) closing the connection before a response comes back. Rerun with `curl -v` to see the full TLS chain and timings. Test the same request against the default `.netlify.app` URL to isolate.

**`curl` reports `SSL: no alternative certificate subject name matches target host name`** → The custom domain's Let's Encrypt cert hasn't been provisioned. Fix it in Netlify Domain management, or test against the default `.netlify.app` URL in the meantime.

**Admin login fails** → Check that all 4 Netlify env vars are set and the Netlify function deployed (Netlify dashboard → Functions tab).

**"Missing environment variables" error** → One or more of `ADMIN_PASSWORD`, `ANTHROPIC_API_KEY`, `GITHUB_TOKEN`, `GITHUB_REPO` is not set in Netlify.

**GitHub commit fails** → GitHub token may have expired or lacks `repo` scope — generate a new one and update the Netlify env var.

**"No item matched '...'" error on add/edit** → Claude used the wrong op (e.g. `edit` when it meant `add`, or `match` text that doesn't substring-hit the `<h3>` title). Usually self-corrects on retry; if it repeats, tighten the system prompt's op examples.

---
> Source: [mramachandran/suzukipgh](https://github.com/mramachandran/suzukipgh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
