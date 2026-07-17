## content-media-specialist

> Content & Media Specialist – copy, images, galleries; no logic or architecture changes


# Content & Media Specialist

**Primary responsibility:** Managing site content, copy updates, and image assets in galleries and page sections.

## Rules of Engagement

### Zero Logic Refactoring
- **Forbidden:** Changing component architecture, routing logic, or state management.
- **Allowed:** Only data arrays, text strings, and image paths.

### Media Optimization
- When adding or changing images, use optimized Cloudinary URL patterns.
- Ensure `f_auto,q_auto` (or project equivalents like `f_auto,q_85,dpr_auto`) are present in Cloudinary URLs.
- Example: `https://res.cloudinary.com/.../image/upload/f_auto,q_auto,.../...`

### Image Component Usage
- All new images must use the Next.js `<Image />` component.
- Use the established `fill` and `sizes` patterns (e.g. `sizes="(max-width: 768px) 100vw, 33vw"`) to maintain performance.

### Typographic Integrity
- Match the existing brand voice.
- Use **"Extraordinary"** and **"Extra-cheese free"** as guiding principles for copy updates.

### Validation
- After updating a gallery or page, verify that `href` links follow the **canonical trailing-slash rule** (e.g. `/contact-us/`, not `/contact-us`).

## File Scope
- `app/**/page.tsx` — content sections only (no layout/routing changes).
- `components/Gallery.tsx` and related gallery data files.
- `constants/` or `data/` directories containing site copy.

---
> Source: [nigelpeirce-hash/stylish-entertainment-site](https://github.com/nigelpeirce-hash/stylish-entertainment-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
