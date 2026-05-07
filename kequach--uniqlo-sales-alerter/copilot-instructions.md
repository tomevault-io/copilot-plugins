## testing

> Testing conventions


# Testing Conventions

## Fixtures and helpers

- Use `sample_deal(**overrides)` from `conftest.py` to build `SaleItem` objects. Never construct `SaleItem(...)` manually in tests.
- Use `_make_email_cfg(**overrides)` for `EmailChannelConfig` in notification tests.
- Use `noop_verify()` and `noop_watched_fetch()` from conftest to isolate filter/check tests from stock verification.
- Use the module-level `_mock_v3_empty()` in `test_uniqlo_client.py` for v3 endpoint mocks.

## Country configs in client tests

- Tests for `fetch_sale_products` must use a country whose `CountryCapabilities.listing_sources` include the endpoints being tested:
  - v5 discount only: `de/de`
  - v5 discount + limitedOffer: `id/en`
  - v3 + v5 mix: `th/en`
  - sale_paths: `sg/en` (with `sale_paths` configured)

## E2E country representatives

- The e2e sweep tests one representative per distinct `CountryCapabilities` combination (listing_sources × stock_api × url_style). Countries within the same group share identical pipeline code paths.
  - `de/de` — v5_disc only, stock_api=v5, display_code URLs (covers uk, fr, es, it, be, nl, dk, se, au, in, kr)
  - `id/en` — v5_disc + v5_ltd, stock_api=v5, display_code URLs (covers vn, my, us, ca, jp)
  - `ph/en` — v3_disc + v3_ltd, stock_api=none, code URLs
  - `th/en` — v5_ltd + v3_disc + v3_ltd, stock_api=none, code URLs
  - `sg/en` — v5_disc + sale_paths, stock_api=v5, display_code URLs
- When adding a new country, check if its capabilities match an existing group. If not, add a new representative to `_REPRESENTATIVE_COUNTRIES` in `test_e2e_html_preview.py`.

## Parametrize over copy-paste

- When 3+ tests follow the same pattern with different inputs, use `@pytest.mark.parametrize`.
- Example: SMTP error tests are parametrized, not duplicated.

---
> Source: [kequach/uniqlo-sales-alerter](https://github.com/kequach/uniqlo-sales-alerter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
