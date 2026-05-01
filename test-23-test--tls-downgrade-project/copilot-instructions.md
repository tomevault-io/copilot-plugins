## tls-downgrade-project

> Security rules for the Flask dashboard. Enforces CSP compliance, authentication, and input validation.


# Dashboard Security Rules

## Content Security Policy

The CSP header is: `script-src 'self' https://cdn.jsdelivr.net`

This means:
- NO inline `onclick`, `onchange`, `onsubmit`, or any `on*` attributes in HTML.
- NO `<script>` blocks in HTML templates. All JS goes in `static/js/dashboard.js`.
- All event handlers use `addEventListener()` inside the `DOMContentLoaded` block in `dashboard.js`.
- Interactive elements use `data-*` attributes for parameters (e.g., `data-scan-type="server"`).

### After Any HTML Edit

Search for inline handlers: `rg 'on(click|change|submit|load|error)=' src/dashboard/templates/`

If any are found, move them to `dashboard.js` as `addEventListener` calls.

## Authentication

- Every route except `/login` requires `@_login_required` decorator.
- Password is bcrypt-hashed in `.env` as `ADMIN_PASSWORD_HASH`.
- Never display the password or hash on any frontend page.
- Session expires after `SESSION_LIFETIME_MINUTES` (default 30).

## Input Validation

Every user-supplied parameter to `/api/scan` must go through a validator:

| Parameter | Validator | Location |
|-----------|-----------|----------|
| host | `_validate_host()` | Rejects shell metacharacters, non-hostname strings |
| port | `_validate_port()` | Enforces 1-65535 integer |
| subnet | `_validate_subnet()` | Validates via `ipaddress.ip_network()` |
| label | `_validate_label()` | Regex whitelist, 100-char max |
| duration | `_validate_duration()` | Bounded numeric, max 600s |
| timeout | `_validate_timeout()` | Bounded numeric, max 60s |
| ports_csv | `_validate_ports_csv()` | Max 20 ports, each validated |

When adding a new scan type, add a corresponding validator for every new parameter.

## File Access

- `_safe_json_read()` canonicalizes paths and rejects traversal outside `sample_results/`.
- PDF `send_file()` validates resolved path stays within project root.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/test-23-test) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
