## skywork

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SkyWork Borivali is a static website for a premium co-working space in Mumbai. This is a traditional multi-page single-file HTML site using jQuery and Bootstrap 5, with no build process or modern framework.

## Technology Stack

- **Frontend**: Vanilla HTML5, CSS3, JavaScript (ES5 syntax)
- **Libraries**:
  - jQuery 3.x (DOM manipulation, AJAX)
  - Bootstrap 5 (layout, components, responsive grid)
  - Swiper.js (carousel for gallery)
  - Magnific Popup (image lightbox)
  - Filterizr (portfolio filtering, currently unused)
  - Line Awesome (icon font)
- **No build tools**: Direct file serving, no transpilation or bundling

## Development Workflow

### Running Locally
Simply open `index.html` in a browser. For a development server:
```bash
python3 -m http.server 8000
# or
npx serve
```

### File Structure
```
skywork/
├── index.html              # Single-page application (all sections in one file)
├── css/
│   ├── style.css          # Custom styles (editable)
│   └── *.min.css          # Third-party libraries (DO NOT edit)
├── js/
│   ├── main.js            # Custom JavaScript (editable)
│   └── *.min.js           # Third-party libraries (DO NOT edit)
├── img/                   # Images for workspace, pricing, gallery
├── reviews.json           # Customer reviews data
└── fonts/                 # Line Awesome icon fonts
```

## Key Architectural Patterns

### Single-Page Scroll Architecture
All sections (home, about, space, gallery, reviews, contact) are in `index.html` with anchor-based navigation (`#home`, `#about`, etc.). The navbar uses smooth scrolling via jQuery to navigate between sections (see `js/main.js:11-26`).

### Anti-Spam Contact Protection
Phone number is obfuscated using simple character rotation in `js/main.js:165-174`. The `decryptPhone()` function displays the contact when users click "Call Us Now" buttons. Similarly, email uses ROT13 encoding (`js/main.js:148-162`).

### Dynamic Reviews System
Reviews are loaded from `reviews.json` via AJAX (`js/main.js:75-126`) and displayed in three horizontally scrolling rows with infinite scroll animation. Reviews are duplicated in the DOM to create seamless looping. Each row scrolls in alternating directions (left-right-left) defined by `data-direction` attributes.

### Responsive Navbar Behavior
- Sticky navbar appears after 50px scroll (`js/main.js:61-72`)
- Mobile hamburger menu auto-closes after link click (`js/main.js:56-58`)
- Mobile-only floating call button positioned bottom-right (hidden on desktop via Bootstrap `d-md-none`)

## Common Development Tasks

### Updating Pricing
Edit the pricing cards in `index.html` around lines 100-156. Update the timestamp on line 105 when prices change.

### Adding/Removing Reviews
Edit `reviews.json`. The reviews array is automatically split into three rows (0-3, 4-7, 8+) by `js/main.js:79-81`. Each review requires: `reviewer_name`, `rating`, `review_date`, `reviewer_stats`, `review_text`, `customer_type`.

### Modifying Gallery Images
Update the Bootstrap carousel in `index.html:173-199`. Each slide needs a `carousel-item` div with an `<img>` tag. Update carousel indicators count to match number of slides.

### Changing Contact Information
- Address: `index.html:303`
- Hours: `index.html:310-311`
- Phone: Update the decoded number in `js/main.js:172` (NOT the encoded version on line 166)
- Google Maps: Replace iframe src on `index.html:292`

### Styling Changes
Edit `css/style.css`. Key sections include:
- Navbar styles (`.navbar`)
- Section-specific styles (`.intro`, `.about`, `.space`, `.gallery`, `.reviews`, `.contact`)
- Review cards (`.review-card`, `.reviews-row`, `.reviews-track`)
- Mobile-specific styles in media queries

## Important Notes

- **jQuery Usage**: This project uses jQuery 3.x. All DOM manipulation should use jQuery syntax (`$()`) to maintain consistency
- **ES5 JavaScript**: Uses IIFE pattern, `var` declarations, and function declarations (no arrow functions, const/let, or ES6+ features)
- **Minified Libraries**: Never edit files ending in `.min.js` or `.min.css` - these are third-party dependencies
- **Responsive Images**: All images in `img/` are high-resolution JPGs/PNGs. Consider optimizing large files before adding new images
- **Bootstrap Grid**: Layout uses Bootstrap 5 grid system (`container`, `row`, `col-*` classes)
- **Icon Usage**: Line Awesome icons via `<i class="la la-icon-name"></i>` (similar to Font Awesome syntax)

## Deployment

This is a static site that can be deployed to:
- GitHub Pages
- Netlify
- Vercel
- Any static file hosting

No build step required - upload all files maintaining directory structure.

## Domain Configuration
The `CNAME` file contains the custom domain configuration for deployment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/webbywonder) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
