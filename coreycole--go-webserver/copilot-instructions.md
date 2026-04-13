## go-webserver

> Go server (Echo framework) serving a personal portfolio site with resume, markdown rendering, WASM games, and a real-time status dashboard.

# go_webserver — Developer Guide

## Architecture

Go server (Echo framework) serving a personal portfolio site with resume, markdown rendering, WASM games, and a real-time status dashboard.

**Stack**: Go + Echo + templ (HTML) + Datastar (reactivity) + datastar-go (SSE) + Tailwind v4 + Chroma (syntax highlighting)

### Key Packages

| Package | Purpose |
|---------|---------|
| `webserver/handle/` | HTTP handlers (welcome, markdown, games, status) |
| `webserver/view/` | templ templates (home, components, status) |
| `webserver/view/layout/` | Root layout: Head, Header, Footer, RootArgs |
| `webserver/view/css/` | Tailwind v4 CSS with design tokens |
| `webserver/lib/` | Markdown renderer, resume JSON parser, html-to-component |
| `webserver/middleware/` | ZeroLog request logging |
| `webserver/webhook/` | GitHub webhook handler for auto-deploy |
| `public/` | Static assets (CSS, JS, favicons, markdown files, images) |

### Routes

| Route | Handler | Purpose |
|-------|---------|---------|
| `GET /` | `GetWelcome` | Homepage — renders `welcome.md` + resume JSON |
| `GET /health` | `GetHealth` | Returns `{"status": "ok"}` |
| `GET /md/*` | `GetMarkdownFile` | Renders markdown from `public/md/` (supports nested paths) |
| `GET /games/:gameName/game` | `GetGame` | WASM game page (assets from S3) |
| `GET /status` | `status.GetStatus` | Real-time system metrics page |
| `GET /status/events` | `status.GetStatusEvents` | SSE stream for metrics |
| `POST /status/graph` | `status.PostGraphUpdate` | Graph range update |
| `POST /webhook/github` | `webhook.HandleGitHub` | GitHub push webhook for auto-deploy |
| `GET /*` | Echo static | Fallback: serves `public/` directory |

## Build

```bash
just build    # templ generate + tailwind + go build
just watch    # air hot-reload
just run      # build + run
just install  # install all dependencies (air, templ, go deps, pnpm, node deps)
just check    # fmt + generate + lint + build (scripts/check.sh)
```

**Note**: Air hot-reload only watches `include_dir: ["webserver"]` — root-level file changes (justfile, package.json) don't trigger rebuilds. Restart air manually for those.

**Note**: `_templ.go` files are gitignored and must be regenerated via `templ generate view`.

## Datastar

Datastar is a hypermedia framework. All reactivity is declarative via `data-*` attributes. Version: `1.0.0-rc-8` (vendored at `public/js/datastar-1-0-0-rc-8-1df5dc253a746506.js`).

### Attribute Syntax

**IMPORTANT — Colon syntax required**: All plugin suffixes use colons: `data-on:click`, `data-bind:value`, `data-on:keydown__window`. Do NOT use dashes (`data-on-click`) — dashes break the plugin lookup because HTML's dataset API converts `data-bind-foo` to `bindFoo` via camelCase, mangling the plugin name.

### Signals (Reactive State)

```html
<!-- Initialize signals with JSON -->
<body data-signals="{theme: initTheme()}">

<!-- Signal access uses $ prefix -->
<div data-show="$theme === 'dark'">Only visible in dark mode</div>
<span data-text="$count"></span>
```

### Key Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `data-signals` | Initialize reactive signals | `data-signals="{open: false}"` |
| `data-init` | Run action on element mount | `data-init="@get('/status/events')"` |
| `data-show` | Conditional visibility | `data-show="$isVisible"` |
| `data-text` | Bind text content | `data-text="$count"` |
| `data-class` | Conditional CSS classes (object syntax) | `data-class="{'active': $tab === 'chat'}"` |
| `data-on:click` | Click handler | `data-on:click="$count++"` |
| `data-on:click__outside` | Click outside handler | Goes on ROOT wrapper div |
| `data-on:keydown__window` | Global keyboard handler | `data-on:keydown__window="evt.key === 'Escape' && ..."` |

### Theme Toggle Pattern

The theme is managed via a Datastar `$theme` signal initialized from `initTheme()` in `<head>`:

1. `initTheme()` runs in `<head>` before paint — reads localStorage/prefers-color-scheme, toggles `.dark` class on `<html>`, returns theme string
2. `<body data-signals="{theme: initTheme()}">` makes the theme available as `$theme` signal
3. Theme toggle button updates signal, class, and localStorage in one expression:
   ```
   data-on:click="$theme = $theme === 'dark' ? 'light' : 'dark'; document.documentElement.classList.toggle('dark', $theme === 'dark'); localStorage.setItem('theme', $theme);"
   ```

### Dropdown Pattern (from datastarui)

- `data-on:click__outside` goes on the ROOT wrapper div
- `data-show` + `data-on:keydown__window` go on the content div
- Initial state: `style="display: none;"` on content div prevents FOUC

### Theme-Aware Server-Rendered Content

For server-rendered content that needs to switch between light/dark variants (e.g., syntax-highlighted code blocks), render both versions and use `data-show` with `$theme`:

```go
// Both start hidden; Datastar shows the correct one
w.Write([]byte(`<div style="display:none" data-show="$theme !== 'dark'">light version</div>`))
w.Write([]byte(`<div style="display:none" data-show="$theme === 'dark'">dark version</div>`))
```

Do NOT use Tailwind's `dark:` variant for this — Tailwind v4 defaults to media-query-based dark mode, not class-based, and server-rendered HTML in Go files won't be in the Tailwind content scan.

### SSE with datastar-go

The status page uses `github.com/starfederation/datastar-go` for server-sent events. Two mechanisms:

1. **Signal patching** — push scalar values to client signals:
   ```go
   sse := datastar.NewSSE(w, r)
   sse.MarshalAndPatchSignals(signalsStruct) // struct fields become signal values
   ```

2. **Element patching** — replace DOM elements with server-rendered templ components:
   ```go
   sse.PatchElementTempl(component, datastar.WithSelectorID("element-id"), datastar.WithMergeInnerElement())
   ```

Client-side, SSE connections are initiated via `data-init="@get('/status/events')"`.

## Status Page

Real-time system metrics dashboard at `/status`.

- **Metrics**: CPU%, Memory%, Disk%, uptime, totals — updated every 2 seconds via SSE
- **Graph**: SVG line chart (CPU + Memory + Views) with time range buttons (1h, 6h, 24h, 3d) — updated every 30 seconds
- **Graph hover**: Tooltip shows "Line: value at HH:MM" with a colored dot snapping to the nearest data point
- **Legend toggle**: Click legend items to show/hide lines; hover shows anchor-positioned eye icon tooltip
- **Points data**: `buildGraph` computes coordinates once in a `pointCoords` struct, reused for both SVG paths and a sampled JSON array (max 360 points) embedded as a Datastar signal for client-side tooltip lookup
- **Ring buffer**: Max 8640 snapshots (3 days at 30s intervals)
- **macOS fix**: `memUsed()` uses `Active + Wired` instead of `Used` (which inflates due to VM compressor)
- **StatusHandler** starts a background goroutine on creation (`NewStatusHandler()`)

## Dark Mode

Dark mode uses CSS custom properties in `@layer base`, NOT Tailwind's `dark:` variant:

- Design tokens defined in `webserver/view/css/index.css` under `:root` (light) and `.dark` (dark)
- Tailwind utilities like `bg-background`, `text-foreground`, `text-primary` resolve to `hsl(var(--background))` etc. via the `@theme` block
- When `.dark` class toggles on `<html>`, CSS variables change and everything updates automatically
- For conditional visibility based on theme, use Datastar `data-show="$theme === 'dark'"` (see above)

## Tailwind v4

- Config is CSS-based via `@theme` directive in `webserver/view/css/index.css` (no `tailwind.config.js`)
- Content scanning uses `--content` CLI flag in justfile, NOT a config file `content` array
- Import: `@import "tailwindcss"` (not `@tailwind base/components/utilities`)
- Build: `pnpm exec tailwindcss -i webserver/view/css/index.css -o public/build.css --content "./webserver/**/*.go" --content "./webserver/**/*.templ"`

## Syntax Highlighting

Code blocks are rendered with dual themes via Chroma v2:
- **Light mode**: `github` style
- **Dark mode**: `github-dark` style
- Both versions rendered server-side, toggled client-side via Datastar `data-show="$theme"` signal

## templ Patterns

- `.templ` files compile to Go with `templ generate`
- `{ children... }` allows component composition
- Layout hierarchy: `Root(RootArgs)` → `Head` + `Header` + `<main>` + `Footer`
- `Default(title)` wraps `Root` for backward compatibility
- `BevyPage` is standalone (no shared layout) — game pages have no header/footer/theme

## WASM Games

Two Bevy-compiled games hosted on S3 (`coreycole-games` bucket, `us-west-2`):
- `giga_platformer` → `giga_platformer-7143ed686304a07e` (114MB)
- `nessyclothes` → `nessyclothes-3d0f9d8535e29267` (10MB)

Assets served from `https://coreycole-games.s3.us-west-2.amazonaws.com/games/`. The `public/games/` directory is gitignored.

## Webhook Auto-Deploy

`POST /webhook/github` receives GitHub push events and self-rebuilds:

1. Verifies HMAC-SHA256 signature via `WEBHOOK_SECRET` env var
2. On push to `main`: `git fetch/reset` → `sqlc generate` → `templ generate` → `tailwindcss` → `go build`
3. Replaces running binary and sends `SIGTERM` for systemd restart
4. Mutex prevents concurrent rebuilds; 5-minute timeout per command

Secret is loaded via `EnvironmentFile=/home/ubuntu/.config/go_webserver/env` in the systemd unit.

## CSS Anchor Positioning

The `@oddbird/css-anchor-positioning` polyfill is loaded in `<head>` when native support is absent. Used for legend tooltips on the status page. Pattern:

- Trigger: `style="anchor-name: --my-anchor"` + `data-on:mouseenter="document.getElementById('tip').showPopover()"`
- Content: `popover="manual"` + `style="position: absolute; position-anchor: --my-anchor; top: anchor(top); left: anchor(center); translate: -50% calc(-100% - 4px);"`

See `context/datastarui/` for the full tooltip/popover component reference.

## Configuration

Port and webhook secret are configurable via environment variables. Uses `kelseyhightower/envconfig`.

```bash
PORT=8080 just run
WEBHOOK_SECRET=xxx  # for GitHub webhook HMAC verification
```

## Known Quirks

- **Resume JSON field swap**: `Company` and `Position` struct fields in `lib/resume_json_to_html.go` have intentionally swapped JSON tags (`"position"` and `"company"`) for PDF export compatibility.
- **SQLite timestamps are UTC**: `CURRENT_TIMESTAMP` stores UTC. Use `.Local()` when formatting `CreatedAt` for display alongside `time.Now()` (which is local time). The graph tooltip had a timezone mismatch until this was fixed.
- **SVG `preserveAspectRatio="none"` distorts circles**: The status graph SVG uses `viewBox="0 0 100 100"` with `preserveAspectRatio="none"`, so `<circle>` elements render as ellipses. The hover dot is an absolutely-positioned `<div>` outside the SVG instead, using CSS `%` positioning (viewBox 0-100 maps to 0%-100%).
- **`data-signals` inside SSE-patched elements overwrites signals**: Putting `data-signals="{graphPoints:[...]}"` on an element inside a `PatchElementTempl(WithModeInner())` target causes Datastar's MutationObserver to pick up the new attribute and overwrite the signal value. This is used intentionally to update `$graphPoints` on each 30s graph re-render.
- **Stale hover flash prevention**: When mouse moves from line A to line B, `mouseenter` fires before `mousemove`. Gate tooltip/dot visibility on a `$hoverActive` boolean set only by `mousemove` (not `mouseenter`), cleared by `mouseleave`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CoreyCole)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CoreyCole)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
