## brand-standards

> Single source of truth for brand, links, images, and Luxury Dark aesthetic


# Brand Standards – Stylish Entertainment

This rule is the **single source of truth** for links, images, and visual style. Never suggest patterns that break these standards.

## Internal links – trailing slash (canonical)

- **All internal links MUST use trailing slashes.**
- Correct: `href="/artists/djs/"`, `href="/contact-us/"`, `href="/parties/party-lighting/"`
- Incorrect: `href="/artists/djs"`, `href="/contact-us"`
- Applies to: `<Link href="...">`, `<a href="...">`, `router.push(...)`, redirect `destination` values in `next.config.js` (destinations already use trailing slash).
- Rationale: Site uses `trailingSlash: true`; consistent URLs avoid duplicate content and redirect chains.

## Images – Next.js Image and Cloudinary

- **All images MUST use the Next.js `<Image />` component** from `next/image`.
- **Cloudinary URLs MUST use optimization parameters:** `f_auto,q_auto` or the project pattern `f_auto,q_85,dpr_auto` (e.g. in `image/upload/f_auto,q_85,dpr_auto/...`).
- **Use `placeholder="blur"`** with `blurDataURL` when possible (or the default blur placeholder for remote images) to avoid layout shift and improve perceived performance.
- Prefer `sizes` and `fill` or explicit `width`/`height` so images are responsive and LCP-friendly.
- Example pattern:
  ```tsx
  <Image
    src="https://res.cloudinary.com/.../image/upload/f_auto,q_85,dpr_auto/..."
    alt="..."
    width={1200}
    height={800}
    placeholder="blur"
    blurDataURL="..." // or omit for remote blur
    sizes="(max-width: 768px) 100vw, 50vw"
  />
  ```
- Raw `<img>` is only acceptable when dynamic `src` or layout makes `<Image />` impractical; still use Cloudinary `f_auto,q_auto` (or `f_auto,q_85,dpr_auto`) in the URL.

## Luxury Dark aesthetic

- **Backgrounds:** Prefer `bg-gray-900`, `bg-gray-800`, or `bg-black` for main sections. Use `bg-gray-900/50`, `bg-gray-800/60` etc. for overlays.
- **Accent and text:** Use `text-champagne-gold` (or `champagne-gold`) for highlights, CTAs, and key headings. Pair with `text-white` or `text-gray-300` for body copy.
- **Overlays and cards:** Use `backdrop-blur-sm` or `backdrop-blur-md` for glass-style overlays; combine with borders such as `border-champagne-gold/20` or `border-champagne-gold/30`.
- **Borders:** Prefer `border-champagne-gold/20` to `border-champagne-gold/30` for subtle luxury; use stronger gold for hover/active states.
- **Shadows / glow:** Optional soft gold glow: `shadow-[0_0_24px_rgba(212,175,55,0.12)]` or `hover:shadow-[0_0_20px_rgba(212,175,55,0.3)]`.
- Avoid bright white backgrounds or flat, non-luxury palettes unless explicitly for contrast (e.g. legal text).

## Summary

| Area           | Rule |
|----------------|------|
| Internal links | Always trailing slash: `/path/page/` |
| Images         | `<Image />`, Cloudinary `f_auto,q_auto` (or `f_auto,q_85,dpr_auto`), `placeholder="blur"` |
| Aesthetic      | Dark bases (`bg-gray-900`, `bg-gray-800`), `text-champagne-gold`, `backdrop-blur` for overlays |

---
> Source: [nigelpeirce-hash/stylish-entertainment-site](https://github.com/nigelpeirce-hash/stylish-entertainment-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
