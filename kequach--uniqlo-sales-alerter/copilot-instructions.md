## project-conventions

> Core project conventions for the Uniqlo Sales Alerter


# Project Conventions

## After every change

- Run `python -m pytest tests/ --tb=short` and fix any failures before finishing — **only when code in `src/` or `tests/` was modified**. Skip for documentation-only changes (`README.md`, `CHANGELOG.md`, `.cursor/rules/`, etc.).
- Run `python -m ruff check src/ tests/` and fix any lint errors — **same rule: only when code was modified**.
- Update `CHANGELOG.md` under the **current version** section for user-visible changes. The current version is the `version` field in `pyproject.toml` — never invent a new version number. Whenever the changelog is updated, also update the current version header's date to today's date (format: `## vX.Y.Z — YYYY-MM-DD`).
- Update `README.md` when user-facing behaviour changes: new config options, new CLI flags, new API endpoints, changed notification format, new Docker examples, or anything a user would need to know.

## Architecture

- **Country capabilities**: Use `config.capabilities` (the `CountryCapabilities` registry in `config.py`) for any country-specific logic. Never hardcode country checks like `if country == "ph"`.
- **Price display** has three states in all notification channels:
  - `has_known_discount=True` and `discount_percentage > 0` -- strikethrough with percentage
  - `has_known_discount=False` -- "Sale" label (limited countries, unknown discount)
  - `has_known_discount=True` and `discount_percentage == 0` -- just the price, no label
- **`has_known_discount`** is determined by whether the item came from the sale feed (`in_sale_feed`), not by the `promo` field.
- **Stock verification**: Controlled by `CountryCapabilities.stock_api`. Countries with `stock_api="v5"` trust the stock data — items where all sizes are OOS are dropped. Countries with `stock_api="none"` (PH, TH) skip the stock call but still fetch L2 variant data for accurate product URLs; items are never dropped.
- **Product URL style**: Controlled by `CountryCapabilities.url_style`. Countries with `url_style="display_code"` (default) use `/{priceGroup}?colorDisplayCode=XX&sizeDisplayCode=YYY`. Countries with `url_style="code"` (PH, TH) use `?colorCode=COLXX&sizeCode=SMAYYY` without a price-group path segment. `build_product_url` handles both.

## Validation for API and filtering changes

- When any code in `src/` is modified that touches **data retrieval** (`clients/uniqlo.py`, `fetch_sale_products`, `_fetch_page`, `_fetch_page_v3`, `_normalize_v3_product`), **stock verification** (`_verify_stock`, `_verify_one`, `_enrich_from_l2`, `_rebuild_from_l2`), **filtering** (`_apply_filters`, `_matching_sizes`, `CountryCapabilities`, `listing_sources`), or **URL construction** (`build_product_url`, `_build_variant_urls`), you **must** also run the e2e country sweep before finishing:

    python -m pytest tests/test_e2e_html_preview.py -m e2e -v --tb=short

- Fix any failures. These tests verify that every registered country returns real products, that PH/TH product counts are not silently truncated, and that generated product URLs use the correct format and point at resolvable products.

## Config and settings

- `server_url` is host-only (no port). The `port` field is separate. Use `config.full_server_url` to get the combined URL.
- The web UI auto-saves watched variants and ignored products immediately; other settings require "Save & Reload".

## Notifications

- All four channels (email, Telegram, console, HTML report) must stay consistent: same information, same price display logic, same action buttons.
- Email expands deals into per-variant rows (one listing per colour+size). Other channels show all sizes in one listing.
- Watched items show "Unwatch" instead of "Watch" in action buttons.
- When `server_url` is configured, include a Settings link in the footer.

---
> Source: [kequach/uniqlo-sales-alerter](https://github.com/kequach/uniqlo-sales-alerter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
