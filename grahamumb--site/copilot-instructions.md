## site

> Vanilla static blog hosted on GitHub Pages at **flush-my-john.com**. No

# FlushMyJohn — personal website

Vanilla static blog hosted on GitHub Pages at **flush-my-john.com**. No
frameworks, no runtime dependencies. Posts are markdown; a small Node build
script converts them to HTML and a GitHub Action deploys on every push.

## Environment & dependencies

Zero dependencies — no `npm install`, no venv. Node 22 (already in the
container) is all that's needed.

```
node build.js                      # build dist/
cd dist && python3 -m http.server 8000   # local preview at http://localhost:8000
```

## How it works

```
push posts/*.md ─▶ GitHub Action (.github/workflows/deploy.yml)
                      └─ node build.js ─▶ dist/ ─▶ deploy to GitHub Pages
```

- `build.js` — zero-dep builder. Reads `posts/*.md` + `about.md`, converts each
  with a tiny inline markdown converter, writes `dist/posts/<slug>.html`,
  `dist/about.html`, and a `dist/posts.json` manifest (newest-first by date
  prefix). Copies `src/*` and `images/` into `dist/`, plus `.nojekyll`.
- `src/` — static site source, copied verbatim into `dist/`:
  - `index.html` — shell: nav (Home / About Me), `#home-title`, `<canvas>`, `#app`.
  - `app.js` — hash router. Home is **hash-less** (bare domain); `#about` and
    `#post/<slug>` are sub-views. Home nav / back links clear the hash via the
    History API so the URL stays clean.
  - `physics.js` — hand-rolled canvas sim (`window.Sim`). One box per post falls
    into a parabolic bowl, collides with walls + other boxes, settles. Clicking a
    box routes to its post. Spawn queue is reversed so the **newest post drops
    last** (lands on top). Tunables (`GRAVITY`, `REST`, `WALL_FRICTION`, etc.)
    are at the top of the file.
  - `styles.css` — dark theme, nav, post typography, home heading overlay.
  - `CNAME` — `flush-my-john.com`; copied into `dist/` so Actions deploys don't
    drop the custom domain.
- `posts/` — markdown posts, named `YYYY-MM-DD-title.md`. First `# Heading` is
  the title (shown on the falling box); the date prefix sets ordering.
- `images/` — referenced from markdown as `images/<file>`; copied to `dist/images`.
- `about.md` — the About Me page content.
- `dist/` — generated output, git-ignored. Never edit by hand.

## Markdown subset

Headings (`#`/`##`/`###`), paragraphs, single-newline `<br>`, `![alt](src)`
images, `[text](url)` links, `**bold**`, `*italic*`, with HTML-escaping. To
extend, edit the converter at the top of `build.js`.

## Deploy notes

- GitHub Pages **Source = GitHub Actions** (one-time repo setting).
- Custom domain uses apex A/AAAA records to GitHub's IPs at Namecheap, plus the
  `CNAME` file above. HTTPS is auto-provisioned; enable "Enforce HTTPS" once the
  cert issues.
- `git push` from inside the container has no credentials — commit here, push
  from the host.

---
> Source: [grahamumb/site](https://github.com/grahamumb/site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
