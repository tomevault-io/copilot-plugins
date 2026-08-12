## ecommerce-ux-standards

> Baymard-grade e-commerce UX standards for Best Bottles — filtering, product lists, PDP, search, navigation, cart, accounts, mobile, and B2B wholesale patterns. Sourced from 200,000+ hours of usability testing across 327 benchmarked sites.


# E-Commerce UX Gold Standard (Baymard Institute)

These standards govern all catalog, product, search, navigation, and checkout UI work on Best Bottles. They are distilled from Baymard Institute research (200,000+ hours of usability testing, 327 top-grossing sites benchmarked, 700+ UX guidelines). Violations cause measurable abandonment — treat these as hard requirements, not suggestions.

Sources: [baymard.com/blog](https://baymard.com/blog), [Product List](https://baymard.com/blog/collections/product-list), [Product Page](https://baymard.com/blog/collections/product-page), [Cart & Checkout](https://baymard.com/blog/collections/cart-and-checkout), [Homepage & Category](https://baymard.com/blog/collections/homepage-and-category), [Accounts & Self-Service](https://baymard.com/blog/collections/accounts-and-self-service), [Popular](https://baymard.com/blog/popular), [B2B Research](https://baymard.com/research/business-to-business)

---

## 1. Faceted Filtering & Product Lists

### Filter Logic
- **One facet = one product attribute.** Never inflate a facet by pulling in products from other attribute dimensions (e.g., Atomizer family must only show family=Atomizer products).
- **OR logic within a group, AND logic across groups.** 15% of sites incorrectly force mutually exclusive selection within filter groups — always use multi-select checkboxes.
- **Show live product counts** next to every option — `Blue (47)`. Update dynamically. Hide zero-count options entirely.
- **Display applied filters in an overview chip bar** (32% don't use best practices for this). Each chip must have a one-click remove action.

### Filter Layout
- **Order facets by usage frequency.** Most-used (price, applicator type, capacity) at top. Re-evaluate quarterly.
- **Promote key filters** as horizontal pill toggles above the grid (All Bottles | Roll-On | Spray). 61% of sites fail to promote important filters.
- **5 essential filter types** must be present for product listings. 57% of sites don't offer all five. Ensure the core filter types users expect are available.
- **Filters for all displayed list-item info.** If capacity, neck thread, color appear on the card, matching filters must exist. 38% of sites don't align filters with displayed attributes.
- **Always explain industry-specific filters.** 62% don't. For Best Bottles: explain what "neck thread size" means with a tooltip or inline hint.
- **Desktop: persistent sidebar.** Never hide filters behind a "Filter" button on desktop — the space is available.
- **Mobile: full-screen filter modal.** Do not use inline accordions that push content down. Collapse to a bottom sheet or modal.
- **Avoid horizontal filtering toolbars** as sole filter mechanism — they hide depth and truncate on smaller viewports.

### Product List Items
- **Combine variations into one list item.** Show one card per product group (e.g. "Cylinder 9ml Clear – 6 variants"), not separate cards per SKU variant. 12% of sites don't — this clutters lists and hurts product finding.
- **5 required attributes per card:** thumbnail, product title, price, variant count, one differentiating spec (capacity/color/applicator for Best Bottles).
- **Display "Price Per Unit"** for multiquantity items. When showing case pricing, always also show per-unit price. 86% of sites don't.
- **Consistent attribute display** across every card in a grid. Never show specs on some cards but not others. (64% get at least one of the two key list-item design principles wrong.)
- **At least 3 product thumbnails** visible per list item for visually-driven products. Let users preview alternate images without opening the PDP.
- **Color swatches** in list items for visually-driven products — 57% of mobile sites don't show all swatches.
- **Highlight items already in the user's cart** with a subtle indicator on the list card. 96% of sites don't — this reduces duplicate adds and improves orientation.
- **Avoid Quick Views for spec-driven products.** Best Bottles is spec-driven (neck thread, capacity, material). Send users to the full PDP; don't use Quick View modals. 21% of spec-driven sites still use Quick Views inappropriately.
- **Synchronized hover effects and unified hit-areas.** The entire card should be clickable with a consistent hover state. 76% of sites don't synchronize hover effects across the card.
- **Scannability over density.** Bold price, muted secondary text, clear thumbnail. Users scan, they don't read line-by-line.

### Sorting & Pagination
- **Sort options:** relevance (default for search), price low→high, price high→low, newest, best-selling, name A→Z. 64% of sites don't offer all four essential sorts (price, rating, best-selling, newest).
- **Relevance sort must be diversity-based** — mix product families and types in results rather than clustering the same family together. 24% of sites don't diversify relevance sort.
- **"Load More" button** or pagination — never auto-load infinite scroll below the initial viewport. 52% of sites get default product count wrong.
- **Show total result count** prominently at top of results.
- **Return users to the same position** in the product list when they press Back from a PDP. 13% of sites don't, forcing re-scroll.

---

## 2. Product Detail Pages (PDP)

### Layout & Content Organization
- **Never use horizontal tabs.** 27% of users overlook tabbed content even when actively looking for it. 28% of sites still use this layout. Use vertical scroll with anchored section headings instead.
- **Structure descriptions by "Highlights"** — bullet-point key features above the fold. 78% of sites don't do this, losing engagement.
- **10% of sites have insufficient product descriptions** — every PDP must have a substantive description explaining material, use case, and differentiators.

### Required PDP Sections
1. Hero image gallery with thumbnail strip
2. Title + SKU + family breadcrumb
3. Price (per-unit AND case pricing for B2B)
4. Variant selector (use buttons, not dropdowns — 28% of desktop sites still use dropdowns for size/variant)
5. Specifications table (neck thread size, capacity, dimensions, material, weight)
6. Compatible fitments/closures section
7. Clear CTA: add-to-cart or request-quote

### Image Gallery
- **Thumbnail strip** for additional images — 76% of mobile sites don't provide thumbnails, yet 50-80% of users overlook truncated galleries (30% of sites get this wrong).
- **At least one "in scale" image** showing the product relative to a hand or common object. 28% get this wrong.
- **Descriptive text or graphics** on some images to call out features. 52% don't do this.
- **Support pinch-to-zoom on mobile.** 25% of sites don't have sufficient image resolution or zoom.
- **Sync data across variations** — when user selects a different variant, update all images, specs, price, and availability. 28% don't synchronize.

### Cross-Sells & Recommendations
- **Show both alternative AND supplementary products.** Only 42% get both right. For Best Bottles: show compatible closures (supplementary) AND sibling bottles in same family (alternative).
- **Cross-sell relevance:** 52% of sites show cart cross-sells that appear unrelated, eroding confidence. Every recommendation must be contextually relevant to the viewed product.

### Shipping & Trust
- **Show estimated shipping costs on PDP.** 43% of sites don't — this is a top-5 cart abandonment reason.
- **"Free Shipping" threshold must appear on PDP**, not just in a site-wide banner. 32% get this wrong.
- **Show stock status** and estimated delivery date.

---

## 3. Search & Autocomplete

### Autocomplete Design (only 19% get all 9 patterns right)
- **Guide query formulation**, not just typing speed. Suggest product families, categories, and common terms.
- **Copy the active suggestion to the search field** on keyboard navigation so users can edit before submitting. 58% fail this.
- **Handle misspellings.** "atomiser" → "atomizer", "roll on" → "roll-on". 69% of sites don't offer suggestions for close misspellings.
- **8 common search query types** exist (exact product, product type, symptom/use-case, feature, compatibility, brand, non-product, slang) — 41% of sites have issues handling at least one.

### Search Results
- **Use the same faceted filtering as the catalog.** Never show a stripped-down results page.
- **Show "No results" with recovery paths:** corrected spelling, related categories, popular products. Generic "No results found" = site exit.
- **Preserve search context** — when user refines or goes back, maintain their query and applied filters.

---

## 4. Homepage & Category Navigation

### Navigation Architecture
- **Full scope visibility.** Homepage and mega-menu must surface the full product taxonomy. 75% of sites overcategorize, leading to abandonment.
- **Product categories as top-level nav items** on mobile. 33% don't.
- **"View All" option** at each level of the mobile product catalog. Only 24% get this right.
- **Sub-sub-category links** are vital for large catalogs. 52% get this wrong.
- **Highlight current scope** in the main navigation. 66% of sites don't.
- **Show sibling categories** for easy scope adjustment. 47% get this wrong.

### Breadcrumbs
- **Every interior page needs breadcrumbs.** Format: Home > Category > Family > Product.
- **Sites need 2 types of breadcrumbs** (hierarchy-based AND history-based). 68% get this wrong.
- **Mobile: provide full category paths.** 36% of sites don't, and others make breadcrumbs hard to find.

### Homepage
- **Can users infer the breadth of your catalog?** Featured products should link to their categories. 43% get this wrong.
- **Carousel: 10 UX requirements** if you use one. 46% of homepage carousels have performance issues. Consider simpler alternatives.
- **Full scope for links on mobile homepages.** 58% don't provide it, causing tunnel vision.

### Footer & Utility Navigation
- **Footer links divided into distinct semantic sections.** 13% don't.
- **Direct links to Return Policy and Shipping Info in footer.** 20% don't.
- **Hover delay of 300-500ms for dropdown menus.** 60% don't, causing accidental triggers.

---

## 5. Cart & Checkout

### Form Design
- **Explain every required field.** "Phone: for delivery updates." 39% don't explain, causing 14% abandonment.
- **Mark BOTH required AND optional fields explicitly.** Only 14% of sites do this.
- **Auto-format credit card fields** with spaces matching the physical card. 80% don't.
- **Expiration date format** must match the physical card. 72% don't.
- **Avoid "Apply" buttons** for most fields (use inline application). 22% still have them.
- **Inline form validation.** 31% don't have it. 4% get it wrong. Errors must be adaptive (98% don't adapt).
- **Single-column form layout.** 16% use multicolumn layouts that confuse reading order.
- **Never require CAPTCHAs** — 8% failure rate (29% if case-sensitive).

### Checkout Flow
- **Guest checkout must be the most prominent option.** 47% don't make it prominent enough.
- **Defer account creation to post-purchase confirmation page.** 42% don't — this is a top abandonment driver.
- **Autodetect city/state from postal code.** 28% of mobile sites don't.
- **Provide fully automatic address lookup.** 55% don't.
- **Use "Shipping Address" as "Billing Address" by default.** 16% of mobile sites have implementation issues.
- **Use "Delivery Date" not "Shipping Speed."** 41% don't.
- **Collapse completed accordion checkout steps into summaries.** Keeps context while reducing visual weight.
- **Checkout must be completely linear.** Never branch or require backtracking.

### Cart Page
- **Persist cart across sessions.** B2B buyers research over days; losing a cart is a deal-breaker.
- **Show order summary at every step.** Line items, quantities, unit price, subtotal, and estimated shipping.
- **Use buttons or buttons+input for quantity updates.** 61% get this wrong (using only +/- or only a dropdown).
- **Cross-sells in cart must be relevant.** 52% show unrelated recommendations.
- **Retain sensitive field data after validation errors.** 34% wipe credit card fields.

---

## 6. Accounts & Self-Service

- **Icon-based account dashboard.** 81% don't offer one. Primary actions (orders, drafts, reorder, tracking) should be one click from dashboard.
- **Distinguish primary from secondary paths** in "My Account" dropdown. 71% don't.
- **Order tracking integrated within the site.** 56% redirect to carrier sites. Keep it internal.
- **Returns interface:** 54% have substantial UX issues. Provide clear self-service returns with status tracking.
- **"Fake editing" flow for stored payment methods.** When users want to update a stored card, show an editable form pre-filled with safe data (last 4 digits) rather than delete-and-re-add. 84% don't do this.
- **Allow newsletter frequency choice.** 80% don't — just on/off is not enough.

---

## 7. Mobile-Specific Standards

- **78% of mobile sites and 58% of desktop have mediocre-to-poor product list UX.** Mobile is the priority surface.
- **Never use subpages within the PDP** (e.g., clicking "Specs" opens a new page). 26% do this. Keep everything in a scrollable single-page layout.
- **Avoid inline scroll areas.** 26% get this wrong. Content should flow naturally in the page scroll.
- **Touch targets:** minimum 44x44px. Accidental taps are a top mobile frustration — implement 300ms touch delay or confirmation for destructive actions.
- **Deemphasize or avoid "Install App" ads.** They interrupt the purchase flow.
- **Hit areas in visual elements must clearly indicate where they lead.** 33% don't make this clear.

---

## 8. B2B / Wholesale Specifics

- **Tiered pricing display.** Show per-unit price at 1pc, 12pc, and case quantity thresholds.
- **Bulk quantity selector** with direct numeric input (not just +/- buttons). Support case-quantity shortcuts.
- **Product tables (line-item view).** B2B buyers prefer scannable tables. Always offer a visual grid / line-item table toggle.
- **Spec sheets and technical data** surfaced in list view — neck thread size, capacity, material visible without opening PDP.
- **Comparison feature is mandatory** for spec-driven products. 67% of B2B buyers use comparison tools. Remove identical attributes to highlight differences, group specs by category, and persist column headers during scroll.
- **Reorder and draft functionality.** Quick reorder from previous purchases. Save-as-draft for in-progress orders.
- **Product data synchronized across variations** — price, availability, specs, images must all update when toggling variants.

---

## Quick Reference: Anti-Patterns to Reject

| Anti-Pattern | Failure Rate | Impact |
|---|---|---|
| Inflating facet counts across attribute dimensions | — | Users lose trust when results don't match the label |
| Horizontal tabs hiding specs/reviews on PDP | 28% of sites | 27% of users never find the content |
| Radio buttons where checkboxes belong in filters | 15% of sites | Forces repeated filtering |
| Hiding filters behind a button on desktop | — | Desktop has the space; use persistent sidebar |
| Auto-loading infinite scroll without "Load More" | 52% get count wrong | Users can't reach footer; disorienting on mobile |
| Generic "No results found" with no recovery path | — | Dead end = site exit |
| Requiring account creation before checkout | 42% of sites | Top-5 abandonment driver |
| Not explaining why phone is required at checkout | 39% of sites | 14% will abandon |
| Truncating product image gallery | 30% of sites | 50-80% of users overlook remaining images |
| Overcategorizing the product catalog | 75% of sites | Analysis paralysis → abandonment |
| Not syncing data across product variations | 28% of sites | Users see stale price/availability |
| No "in scale" product image | 28% of sites | Users can't judge size, reducing purchase confidence |
| Dropdown menus without hover delay | 60% of sites | Accidental triggers, navigation frustration |
| Insufficient product descriptions | 10% of sites | Users leave to research elsewhere, may not return |
| Showing each variant as a separate list item | 12% of sites | Clutters results, hinders product finding |
| Quick Views on spec-driven products | 21% of sites | Users can't evaluate specs in a modal; send to PDP |
| Not showing per-unit price for multiquantity | 86% of sites | B2B buyers can't compare value across quantities |
| No cart indicator on list items | 96% of sites | Users re-add items or lose track of what's selected |
| Non-diverse relevance sort | 24% of sites | Results cluster by family; users miss product breadth |

---
> Source: [asalastudio/best-bottles-website](https://github.com/asalastudio/best-bottles-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
