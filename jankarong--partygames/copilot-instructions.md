## partygames

> When adding new games, follow these SEO character limits:

# Party Games Project Guidelines

## SEO Standards

When adding new games, follow these SEO character limits:

### Meta Tags Character Limits
- **Title**: 50-60 characters (max 70)
- **Description**: 150-160 characters (max 320)  
- **Keywords**: 5-10 keywords only (meta keywords are largely ignored by search engines)

### Required Elements for Each Game
- [ ] Favicon using game's logo (`<link rel="icon" type="image/png" href="./[game-logo].png">`)
- [ ] Title within character limit
- [ ] Description within character limit
- [ ] Open Graph tags (og:title, og:description, og:image)
- [ ] Canonical URL
- [ ] JSON-LD structured data

## File Structure
- Each game should be in its own folder under `/games/`
- Game logo should be in the same folder as the HTML file
- Use existing CSS and JS structure for consistency

## URL Structure (IMPORTANT for SEO)

**All game URLs must NOT include `.html` extension** - This is critical for SEO:

### Rules:
- ❌ **WRONG**: `https://www.bestpartygames.net/games/sexdice/sexdice.html`
- ✅ **CORRECT**: `https://www.bestpartygames.net/games/sexdice/sexdice`

### Where to apply this rule:
1. **Canonical URLs in HTML files**
   ```html
   <link rel="canonical" href="https://www.bestpartygames.net/games/sexdice/sexdice">
   ```

2. **All internal links in HTML**
   ```html
   <a href="/games/sexdice/sexdice">Link text</a>  <!-- NO .html -->
   ```

3. **Navigation links in JS files**
   ```javascript
   { name: 'Sex Dice', url: '/games/sexdice/sexdice' }  <!-- NO .html -->
   ```

4. **Sitemap (sitemap.xml)**
   ```xml
   <loc>https://www.bestpartygames.net/games/sexdice/sexdice</loc>  <!-- NO .html -->
   ```

5. **Meta tags (og:url, JSON-LD)**
   ```html
   <meta property="og:url" content="https://www.bestpartygames.net/games/sexdice/sexdice">
   ```

### Why this matters:
- Cloudflare already handles URL rewriting (`.html` files are served without extension)
- Google uses Sitemap as the source of truth for "real" URLs
- If Sitemap has `.html`, Google indexes both versions, splitting SEO value
- This causes 307 redirects and duplicate content issues
- **Always remove `.html` to keep all SEO value on one canonical URL**

## Notes
- Focus on natural keyword integration in title and description
- Avoid keyword stuffing
- Ensure mobile-friendly descriptions (shorter is better)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jankarong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jankarong)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
