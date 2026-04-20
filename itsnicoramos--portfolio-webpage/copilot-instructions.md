## portfolio-webpage

> Static personal portfolio site for Nico Ramos, deployed on Netlify at [itsnico.dev](https://itsnico.dev). No framework, no SSR, no database. The main portfolio is plain HTML/CSS/JS. The blog uses markdown files managed by a TypeScript manifest compiled to a browser script.

# CLAUDE.md — Portfolio Webpage

## Project overview

Static personal portfolio site for Nico Ramos, deployed on Netlify at [itsnico.dev](https://itsnico.dev). No framework, no SSR, no database. The main portfolio is plain HTML/CSS/JS. The blog uses markdown files managed by a TypeScript manifest compiled to a browser script.



---

## File structure

```
index.html          Single-page portfolio layout
styles.css          All styles (CSS custom properties, responsive, themes)
script.js           Theme toggle, scroll animations, Google Translate, nav
tsconfig.json       Compiles blog/blog.ts → blog/blog.js (module: None, target: ES2017)
package.json        devDependencies: typescript

blog/
  blog.ts           SOURCE — post manifest (POSTS[]) + renderPostList() + renderPost()
  blog.js           COMPILED — committed, loaded by blog HTML pages; regenerate with npx tsc
  blog.css          Blog-specific styles (shared by index + post pages)
  index.html        Blog listing — calls renderPostList('posts-list') from blog.js
  post.html         Post template — calls renderPost() from blog.js; fetches posts/<slug>.md
  posts/
    *.md            One file per post — pure markdown, no frontmatter needed

netlify/            Netlify serverless functions
netlify.toml        Netlify config: headers, CSP, redirects
img/                Local image assets
```

---

## Blog system

### How it works

- `blog/blog.ts` is the single source of truth for post metadata (title, date, tag, excerpt, read time, view counter key).
- `blog/index.html` renders the post list dynamically by calling `renderPostList('posts-list')`.
- `blog/post.html` renders a post by reading `?slug=` from the URL, fetching `posts/<slug>.md`, and parsing it with `marked` (loaded from `cdn.jsdelivr.net`).
- View counts use [counterapi.dev](https://api.counterapi.dev) — namespace `itsnico-dev`.

### Adding a new post

1. Create `blog/posts/<slug>.md` — write the post in markdown
2. Add an entry to `POSTS` in `blog/blog.ts`:
   ```ts
   {
     slug: 'my-post',
     title: 'Post Title',
     date: '2026-03-23',
     dateLabel: 'Mar 23, 2026',
     tag: 'Tag',
     excerpt: 'One sentence shown in the card.',
     readTime: '3 min read',
     counterKey: 'blog-my-post',
   }
   ```
3. Run `npx tsc`
4. Commit `blog/blog.ts`, `blog/blog.js`, and `blog/posts/<slug>.md`

### Updating the homepage blog preview

The `index.html` blog section has a hardcoded card for the featured post. When a new featured post is promoted, update that card manually (link, title, excerpt, date) — it is not driven by `blog.ts`.

---

## Theming

Light/dark mode uses CSS custom properties on `data-theme` attribute (`light` | `dark`) set on `<html>`. Theme is persisted in `localStorage` and applied before first paint (inline `<script>` in `<head>`) to prevent flash.

---

## Content Security Policy

The CSP is defined in `netlify.toml`. Allowed external origins include:
- `cdnjs.cloudflare.com` — Font Awesome, Three.js
- `cdn.jsdelivr.net` — `marked` (markdown parser for blog posts)
- `fonts.googleapis.com` / `fonts.gstatic.com` — Google Fonts
- `api.counterapi.dev` — blog view counters (`connect-src`)

If adding a new external dependency, add it to the CSP in `netlify.toml` before deploying.

---

## Deployment

Netlify auto-deploys from the `main` branch. Publish directory is `.` (repo root). No build command is needed on Netlify — `blog/blog.js` is committed and served directly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/itsnicoramos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
