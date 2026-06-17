## sargeant-lee

> This project is a Full Site Editing (FSE) WordPress Theme with WooCommerce integration.

# AI Zippy — FSE Theme + WooCommerce

This project is a Full Site Editing (FSE) WordPress Theme with WooCommerce integration.

## Build Commands (run from project root)
```bash
npm run dev      # Vite watch + wp-scripts watch + BrowserSync
npm run build    # Vite production + wp-scripts production
```

> **Note:** User typically runs `npm run dev` in their terminal. Do NOT run build commands unless explicitly asked.

## Project Structure
```
ai_zippy/
├── package.json & vite.config.js     # Build config (root level)
├── bs.config.js                      # BrowserSync (reads PROJECT_HOST from .env)
├── docker-compose.yml                # WordPress + MySQL
└── src/wp-content/themes/
    ├── ai-zippy/                     # Parent theme
    │   ├── theme.json                # Design tokens (single source of truth)
    │   ├── functions.php             # Constants + loader only (16 lines)
    │   ├── templates/                # FSE page templates
    │   ├── parts/                    # Reusable template parts (header, footer)
    │   ├── woocommerce/              # WC template overrides
    │   │   └── checkout/
    │   │       └── form-checkout.php # Custom card-based checkout layout
    │   ├── src/
    │   │   ├── js/
    │   │   │   ├── theme.js          # Entry: header, shop-view-toggle, add-to-cart
    │   │   │   ├── modules/          # Vanilla JS modules (header, cart-api, add-to-cart)
    │   │   │   ├── shop-filter/      # React app: product filtering
    │   │   │   ├── cart/             # React app: cart page
    │   │   │   └── checkout/         # React app: checkout page
    │   │   ├── scss/
    │   │   │   ├── style.scss        # Main entry (imports all partials)
    │   │   │   ├── wc-checkout-entry.scss  # WC default checkout (conditional)
    │   │   │   ├── _variables.scss   # Colors, breakpoints, mixins from theme.json
    │   │   │   ├── _base.scss        # Box-sizing, img, links
    │   │   │   ├── _header.scss
    │   │   │   ├── _shop.scss
    │   │   │   ├── _cart.scss
    │   │   │   ├── _mini-cart.scss
    │   │   │   ├── _add-to-cart.scss # AJAX add-to-cart + toast notifications
    │   │   │   ├── _checkout.scss    # WC checkout block styles
    │   │   │   ├── _wc-checkout.scss # Classic WC checkout styles (conditional)
    │   │   │   ├── _wc-notices.scss  # Toast-style WC notices
    │   │   │   └── _footer.scss
    │   │   └── blocks/               # Custom Gutenberg blocks (wp-scripts)
    │   │       ├── hero-section/
    │   │       └── product-showcase/
    │   ├── assets/
    │   │   ├── dist/                 # Vite build output
    │   │   └── blocks/               # wp-scripts build output
    │   └── inc/                      # PSR-4 PHP classes (AiZippy\ namespace)
    │       ├── loader.php            # Autoloader + bootstrap all modules
    │       ├── setup/
    │       │   └── dynamic-url.php   # Auto-detect URL for tunnel/local dev
    │       ├── Core/
    │       │   ├── ViteAssets.php     # Manifest reader + asset enqueue + WC nonce
    │       │   ├── ThemeSetup.php     # Theme supports, blocks, block categories
    │       │   ├── Cache.php          # Centralized cache keys
    │       │   └── RateLimiter.php    # IP-based rate limiting
    │       ├── Api/
    │       │   └── ProductFilterApi.php  # REST: /ai-zippy/v1/products, /filter-options
    │       ├── Hooks/
    │       │   └── CacheInvalidation.php # Clear cache on product/category changes
    │       ├── Shop/
    │       │   └── ShopAssets.php     # Shop filter React app enqueue
    │       ├── Cart/
    │       │   └── CartAssets.php     # Cart React app enqueue
    │       └── Checkout/
    │           ├── CheckoutAssets.php     # Conditional enqueue (React or WC default)
    │           ├── CheckoutSettings.php   # WC Settings: checkout template selector
    │           ├── CheckoutShortcode.php  # [ai_zippy_checkout] shortcode
    │           └── CheckoutValidation.php # Server-side phone/email validation
    └── ai-zippy-child/               # Child theme (per-client customizations)
```

## Architecture Decisions

### PHP — PSR-4 Classes
- All PHP logic lives in `inc/` under the `AiZippy\` namespace
- `functions.php` only defines constants and requires `loader.php`
- `loader.php` handles autoloading and bootstraps all modules via `::register()`
- Never put logic directly in `functions.php`

### Dual Build System
- **Vite** (`vite.config.js`): Theme JS/SCSS + React apps → `assets/dist/`
- **@wordpress/scripts**: Gutenberg blocks → `assets/blocks/`
- Vite entries: `theme`, `style`, `shop-filter`, `cart`, `checkout`, `wc-checkout`
- Manifest keys use full paths from root: `src/wp-content/themes/ai-zippy/src/js/theme.js`

### CSS-Only Vite Entries
- `ViteAssets::enqueue()` handles both JS entries and CSS-only entries
- CSS-only entries (like `wc-checkout-entry.scss`) output `.css` files directly in the manifest
- Entry file and partial must have different names to avoid Sass ambiguity (e.g., `wc-checkout-entry.scss` imports `_wc-checkout.scss`)

### SCSS Variables & Mixins
- Colors: `$c-primary`, `$c-accent`, `$c-border`, etc. mapped from `theme.json` CSS custom properties
- Breakpoints: `$bp-sm` (480), `$bp-md` (768), `$bp-lg` (1024), `$bp-xl` (1200), `$bp-xxl` (1400)
- Mixins: `@include from(md)`, `@include until(lg)`, `@include transition(...)`, `@include truncate(2)`, `@include card-hover`

### WC Store API Nonce
- Provided globally via `ViteAssets::enqueueTheme()` as `wcBlocksMiddlewareConfig`
- Used by `cart-api.js`, `add-to-cart.js`, checkout API, cart API
- Do NOT add duplicate nonces in page-specific asset classes

### Checkout — Dual Template
- Admin selects template in **WooCommerce > Settings > Advanced > Checkout template**
- `react` → React checkout app (step-by-step: Contact → Billing → Payment)
- `woocommerce` → Classic WC checkout with custom `form-checkout.php` override
- `[ai_zippy_checkout]` shortcode in `page-checkout.html` renders the selected template
- `CheckoutAssets` conditionally loads React JS or WC checkout CSS (never both)
- Phone input uses `react-international-phone` library (preferred countries: SG, MY, VN)

### React Apps
- **Shop Filter** (`/shop`): Product filtering, pagination, grid/list toggle
- **Cart** (`/cart`): Full cart management via WC Store API
- **Checkout** (`/checkout`): Step-by-step with numbered sections, phone with country code
- All use WC Store API for client-side operations (no page reloads)
- Cart steps component (`CartSteps.jsx`) shared between cart and checkout

### Gutenberg Blocks
- Server-side rendered (`save.js` returns null, `render.php` for output)
- Block category: "AI Zippy" (slug: `ai-zippy`)
- `hero-section`: Two-column hero with CTA
- `product-showcase`: Grid/slider with Swiper.js (CDN, dynamically loaded)

## Coding Rules
- All PHP in PSR-4 classes — no procedural functions outside `functions.php`
- No jQuery — use vanilla JS or React
- Use theme.json design tokens — never hardcode colors
- SCSS uses `_variables.scss` mixins for responsive breakpoints
- WC template overrides go in `woocommerce/` directory
- CSS class prefixes: `az-` (theme), `zc-` (cart app), `zk-` (checkout app), `sf__` (shop filter)

---
> Source: [FCS-WP/sargeant_lee](https://github.com/FCS-WP/sargeant_lee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
