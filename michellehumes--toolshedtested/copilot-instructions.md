## toolshedtested

> This is a WordPress affiliate marketing site for power tool reviews. The theme is located in `wp-content/themes/toolshed-tested/`.

# Claude Code Instructions for ToolshedTested

## Project Overview
This is a WordPress affiliate marketing site for power tool reviews. The theme is located in `wp-content/themes/toolshed-tested/`.

## CRITICAL: Always Use Existing Components First

Before creating ANY new code, check these existing resources:

### Available Shortcodes (`inc/shortcodes.php`)
| Shortcode | Usage | Description |
|-----------|-------|-------------|
| `[product_box id="123"]` | Product reviews | Displays product with image, rating, pros/cons, CTA |
| `[comparison_table ids="1,2,3"]` | Roundup posts | Side-by-side product comparison table |
| `[star_rating rating="4.5"]` | Inline use | Displays star rating visualization |
| `[affiliate_button url="..." text="..." style="amazon"]` | CTAs | Affiliate link button (styles: amazon, cta, primary, secondary) |
| `[pros_cons pros="A\|B" cons="C\|D"]` | Product sections | Visual pros/cons list |
| `[disclosure]` | All posts | FTC affiliate disclosure notice |
| `[newsletter title="..." description="..."]` | Lead gen | Email signup form |

### Available CSS Classes
**Buttons:** `.tst-btn`, `.tst-btn-primary`, `.tst-btn-secondary`, `.tst-btn-amazon`, `.tst-btn-cta`
**Cards:** `.review-card`, `.review-box`, `.card-rating`, `.card-price`, `.card-actions`
**Grids:** `.posts-grid`, `.reviews-grid`, `.reviews-grid.small`
**Badges:** `.badge`, `.badge-best-seller`, `.badge-editors-choice`, `.badge-budget-pick`
**Tables:** `.comparison-table`, `.spec-table`
**Components:** `.pros-cons`, `.faq-section`, `.newsletter-box`, `.author-box`, `.trust-badges`

### Template Parts
- `template-parts/content/` - Post content templates
- `template-parts/review/review-card.php` - Review card component
- `template-parts/product/comparison-table.php` - Product comparison table

### Automation Scripts (`/scripts/`)
- `wp_publish.py` - Publish markdown to WordPress
- `bulk-import.py` - Batch content import
- `publish_to_hostinger.py` - Deploy to Hostinger
- `analyze_content.py` - Content quality analysis
- `check_affiliate_links.py` - Validate affiliate links
- `fix_affiliate_tags.py` - Fix Amazon tags

## Affiliate Configuration
- **Amazon Tag:** `toolshedtested-20`
- **Link Attributes:** `rel="nofollow noopener sponsored"`
- **Additional Retailers:** Home Depot, Lowe's, Acme Tools

## Color Palette
```css
--tst-primary: #2d5a27;      /* Forest Green */
--tst-primary-dark: #1e3d1a;
--tst-secondary: #f4a524;    /* Tool Orange */
--tst-accent: #e63946;       /* Alert Red */
```

## Typography
- **Headings:** Montserrat (600, 700, 800)
- **Body:** Inter (400, 500, 600, 700)

## Development Guidelines

### DO:
- Modify existing components before creating new ones
- Use existing CSS classes and variables
- Follow WordPress coding standards
- Ensure mobile responsiveness
- Add proper escaping and sanitization
- Test affiliate link tracking

### DON'T:
- Create duplicate functionality
- Use inline styles when CSS classes exist
- Skip FTC disclosure requirements
- Hardcode affiliate tags (use theme customizer)
- Create new shortcodes without checking existing ones

## Quick Commands

### Run Site Audit
```bash
claude --dangerously-skip-permissions /site-review-and-implementation
```

### Publish Content
```bash
python scripts/wp_publish.py posts/new-article.md
```

### Check Affiliate Links
```bash
python scripts/check_affiliate_links.py
```

### Analyze Content Quality
```bash
python scripts/analyze_content.py posts/
```

## Key Files
- `wp-content/themes/toolshed-tested/functions.php` - Theme functions
- `wp-content/themes/toolshed-tested/inc/shortcodes.php` - All shortcodes
- `wp-content/themes/toolshed-tested/inc/class-tst-schema.php` - Schema markup
- `wp-content/themes/toolshed-tested/inc/class-tst-affiliate.php` - Affiliate handling
- `wp-content/themes/toolshed-tested/style.css` - Main styles
- `wp-content/themes/toolshed-tested/assets/css/components.css` - Component styles

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michellehumes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
