## frontend-rules

> Alpine.js SPA, Vite, and Tailwind conventions for frontend code

## **2. Frontend Development Rules**
### **SPA-Like Behavior**
- **Rule 2.1**: Use **Fetch API** (`/api/...`) for dynamic data loading.
- **Rule 2.2**: Use **Alpine.js** for view state; templates are HTML fragments bundled via Vite `?raw` imports in `js/pages.js`.
- **Rule 2.3**: Prefer dynamic DOM updates via `loadPage()`; avoid full page reloads where possible.

### **Templating**
- **Rule 2.4**: Store view fragments in `frontend/src/templates/` (`.hbs` = Alpine HTML partials, not Handlebars-rendered).
- **Rule 2.5**: Register pages in `frontend/src/js/pages.js`; Vite bundles templates and controllers at build time.

### **Interactivity (Alpine.js)**
- **Rule 2.6**: Use Alpine.js for simple state management (`x-data`, `x-show`).
- **Rule 2.7**: Avoid complex Alpine.js logic; delegate to Fetch API when needed.

### **Styling (Tailwind CSS)**
- **Rule 2.8**: Use **Tailwind utility classes** (avoid custom CSS unless necessary).
- **Rule 2.9**: Keep responsive design in mind (`sm:`, `md:`, `lg:` prefixes).

### **Performance (Vite JS)**
- **Rule 2.10**: Lazy-load non-critical JS/CSS with Vite.
- **Rule 2.11**: Optimize images and assets via Vite's build pipeline.
- **Rule 2.12**: File-level JSDoc on exported modules; `@param` / `@returns` on exported functions when not self-evident. See [documentation.mdc](documentation.mdc) and `frontend/AGENTS.md`.

### **SEO & discoverability**
- **Rule 2.13**: Classify each page as `public`, `app`, or `auth` in the `pages.js` registry `seo` block; unset `visibility` inherits the fork `SEO_MODE` default.
- **Rule 2.14**: Public pages require one `<h1>`, logical heading order, semantic landmarks (`main`, `nav`, `article`), real `<a href>` links, unique title + meta description, and critical copy in template HTML (not fetch-only).
- **Rule 2.15**: `app` and `auth` pages use `noindex`; do not keyword-stuff UI copy.
- **Rule 2.16**: Images require meaningful `alt` text.

See `frontend/AGENTS.md` for extension recipes.

---
> Source: [Elizio/vanilla-webapp-framework](https://github.com/Elizio/vanilla-webapp-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
