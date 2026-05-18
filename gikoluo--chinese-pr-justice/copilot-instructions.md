## chinese-pr-justice

> This document provides an overview of the **Justice for Chinese PR Applicants** website project. It's designed to help new team members quickly understand the codebase and start contributing.

# Project Documentation

This document provides an overview of the **Justice for Chinese PR Applicants** website project. It's designed to help new team members quickly understand the codebase and start contributing.

## Project Overview

This is an advocacy website for Chinese Permanent Residence applicants in Canada who are experiencing extended security screening delays. The site promotes a peaceful procession to Parliament Hill scheduled for March 13, 2026.

**Live site:** [chinese-pr-delay.ca](https://chinese-pr-delay.ca)

## Tech Stack

| Technology | Purpose |
|------------|---------|
| **Astro 5.x** | Static site generator / framework |
| **CSS** | Styling (scoped + global) |
| **Cloudflare Workers** | Hosting / deployment |
| **Wrangler** | Cloudflare CLI for deployment |

## Project Structure

```
chinese-pr-justice/
├── src/
│   ├── components/           # Reusable UI components
│   │   ├── Navbar.astro      # Navigation bar + announcement bar
│   │   ├── Footer.astro      # Site footer
│   │   ├── Hero.astro        # Main hero section (slogan + demands)
│   │   └── Hero2.astro       # Secondary hero (background image)
│   ├── layouts/
│   │   └── BaseLayout.astro  # HTML wrapper, meta tags, fonts
│   ├── pages/                # Each file = one route
│   │   ├── index.astro       # Home page (/)
│   │   ├── event.astro       # Event details (/event)
│   │   └── join.astro        # Join/volunteer (/join)
│   └── styles/
│       └── global.css        # CSS variables, shared styles
├── static/
│   └── images/               # All images (JPG, PNG, etc.)
├── dist/                     # Build output (auto-generated)
├── astro.config.mjs          # Astro configuration
├── wrangler.toml             # Cloudflare Workers config
├── worker.js                 # Cloudflare fetch handler
└── package.json              # Dependencies & scripts
```

## Key Files Explained

### Pages (`src/pages/`)

Each `.astro` file in `pages/` becomes a route:
- `index.astro` → `/` (home page with Declaration, Demands, Stories sections)
- `event.astro` → `/event` (procession details, schedule, photos)
- `join.astro` → `/join` (volunteer form, contact info)

### Components (`src/components/`)

| Component | Description |
|-----------|-------------|
| `Navbar.astro` | Sticky navigation + red announcement bar at top |
| `Footer.astro` | Four-column footer with links and contact info |
| `Hero.astro` | Main hero with slogan in 3 languages, event info, demands list |
| `Hero2.astro` | Image background hero with quote and CTA buttons |

### Layout (`src/layouts/`)

`BaseLayout.astro` wraps all pages with:
- HTML `<head>` (meta tags, title, Google Fonts)
- `<Navbar />` component
- Page content via `<slot />`
- `<Footer />` component
- Story modal and scroll animations (JavaScript)

### Styles (`src/styles/`)

**CSS Architecture:**
- **`global.css`** - CSS variables, base styles, buttons, section-specific styles
- **Scoped styles** - Each component has its own `<style>` block for component-specific CSS

**CSS Variables (defined in global.css):**
```css
--primary-color: #1e3a5f;      /* Dark blue */
--primary-dark: #0f1f33;       /* Darker blue */
--accent-color: #c9a227;       /* Gold */
--accent-bright: #fbbf24;      /* Bright gold */
--text-dark: #1a1a2e;
--text-light: #4a5568;
--bg-light: #f8f9fa;
--font-heading: 'Caudex', serif;
--font-body: 'Work Sans', sans-serif;
```

### Static Assets (`static/`)

All images go in `static/images/`. Reference them in code as `/images/filename.jpg`.

Example:
```html
<img src="/images/hero2-bg-feb-9.jpg" alt="..." />
```

## Common Tasks

### Adding a New Page

1. Create `src/pages/newpage.astro`:
```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---

<BaseLayout title="Page Title | Justice for Chinese PR Applicants">
    <section class="your-section">
        <div class="container">
            <h2>Your Content</h2>
        </div>
    </section>
</BaseLayout>

<style>
    /* Scoped styles for this page */
</style>
```

2. Add link to Navbar if needed (`src/components/Navbar.astro`)

### Adding a New Component

1. Create `src/components/YourComponent.astro`:
```astro
---
// Component frontmatter (props, imports)
interface Props {
    title: string;
}
const { title } = Astro.props;
---

<div class="your-component">
    <h3>{title}</h3>
</div>

<style>
    .your-component {
        /* Scoped styles */
    }
</style>
```

2. Import and use in pages:
```astro
---
import YourComponent from '../components/YourComponent.astro';
---

<YourComponent title="Hello" />
```

### Modifying Styles

- **Component-specific styles:** Edit the `<style>` block in the component
- **Global styles / CSS variables:** Edit `src/styles/global.css`
- **Shared button styles:** In `global.css`, look for `.btn`, `.btn-primary`, etc.

### Adding Images

1. Add image to `static/images/`
2. Reference as `/images/your-image.jpg` in HTML/Astro

## Development Commands

```bash
# Install dependencies
npm install

# Start dev server with hot reload (http://localhost:4321)
npm run dev

# Build for production
npm run build

# Preview production build locally
npm run preview

# Deploy to Cloudflare Workers
npm run deploy
```

## Important Notes

### Astro Specifics

- Files use `.astro` extension (similar to JSX but with frontmatter)
- Frontmatter goes between `---` fences at the top
- Use `class` not `className`
- Scoped styles only apply to the component they're in
- Use `:global(.class)` in scoped styles to target global classes

### Deployment

- Build output goes to `dist/` (configured in `astro.config.mjs`)
- Cloudflare Workers serves from `dist/` (configured in `wrangler.toml`)
- Always run `npm run build` before deploying to catch errors

### Event Information

The procession is scheduled for **March 13, 2026** at Parliament Hill, Ottawa.
Key dates/info are in:
- `Hero.astro` - hero section event line
- `event.astro` - full event details
- `Navbar.astro` - announcement bar

## Related Documentation

- `DEPLOY.md` - Deployment instructions
- `QUICK_START.md` - Quick setup guide
- `WEBSITE_SUMMARY.md` - Content overview
- [Astro Docs](https://docs.astro.build/) - Framework documentation

---
> Source: [gikoluo/chinese-pr-justice](https://github.com/gikoluo/chinese-pr-justice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
