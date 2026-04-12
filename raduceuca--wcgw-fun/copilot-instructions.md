## wcgw-fun

> The creative studio landing page for [wcgw.fun](https://wcgw.fun).

# wcgw-fun

The creative studio landing page for [wcgw.fun](https://wcgw.fun).

## Quick Reference

| Aspect | Value |
|--------|-------|
| **Type** | Static HTML/CSS site |
| **Deploy** | Cloudflare Pages (auto-deploy on push to main) |
| **Domain** | wcgw.fun |
| **Analytics** | Microsoft Clarity (disabled on localhost) |

---

## Project Structure

```
site/
├── index.html       # Main landing page (all content + inline JS)
├── styles.css       # Stylesheet with CSS animations
├── favicon.svg      # Animated favicon
├── logo.svg         # Brand logo
├── og-image.png     # Open Graph social card
└── og-template.html # OG image template reference
```

No build step. The `site/` directory is deployed directly to Cloudflare Pages.

---

## Design Direction

- **Font**: JetBrains Mono (monospace)
- **Aesthetic**: Warning-label / incident-report energy. Clinical, not playful.
- **Layout**: Centered, max-width ~540px, generous whitespace
- **Accent color**: Phosphor green, used sparingly
- **Modes**: Light (default) and dark, toggled by clicking the logo

See `wcgw-fun-brief.md` for the full design brief.

---

## Notes

- No JavaScript framework, no bundler, no dependencies needed for the site
- The site has inline JS for: theme toggle, animated favicon, Microsoft Clarity events
- Clarity is conditionally loaded (skipped on localhost/127.0.0.1)
- Scaffolded with [claudecraft](https://claudecraft.dev) by [@raduceuca](https://x.com/raduceuca)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raduceuca)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/raduceuca)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
