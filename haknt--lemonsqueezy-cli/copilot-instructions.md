## lemonsqueezy-cli

> A CLI tool for managing LemonSqueezy stores, products, customers, orders, subscriptions, and more. Built for AI coding agents and humans.

# LemonSqueezy CLI

A CLI tool for managing LemonSqueezy stores, products, customers, orders, subscriptions, and more. Built for AI coding agents and humans.

## Setup

```bash
export LEMONSQUEEZY_API_KEY=your_key_here
export LEMONSQUEEZY_STORE_ID=your_store_id  # optional
```

## Usage

```
lemonsqueezy <resource> <action> [id] [flags]
```

Always use `--json` for structured output when processing results programmatically.

## Resources

| Resource | Actions |
|---|---|
| auth | whoami |
| store | get, list |
| products | get, list |
| variants | get, list |
| customers | get, list, create, update, archive |
| orders | get, list, invoice, refund |
| subscriptions | get, list, update, cancel |
| checkouts | get, list, create |
| webhooks | get, list, create, update, delete |
| discounts | get, list, create, delete, redemptions |
| licenses | get, list, update, activate, deactivate, validate, instances |
| usage | get, list, create |

## Examples

```bash
# List all products
lemonsqueezy products list --json

# Get a specific subscription
lemonsqueezy subscriptions get 12345 --json

# List active subscriptions
lemonsqueezy subscriptions list --status active --json

# Create a checkout link
lemonsqueezy checkouts create --variant-id 12345 --email user@example.com --json

# Cancel a subscription
lemonsqueezy subscriptions cancel 12345 --json

# List orders with pagination
lemonsqueezy orders list --limit 25 --page 2 --json

# Create a discount
lemonsqueezy discounts create --name "Launch" --code LAUNCH20 --amount 20 --amount-type percent --json

# Validate a license key
lemonsqueezy licenses validate --key "ABCD-1234-EFGH-5678" --json

# Create a webhook
lemonsqueezy webhooks create --url https://example.com/hook --events subscription_created,order_created --secret mysecret --json
```

## Output Modes

- Default: human-readable table with color
- `--json`: structured JSON (use this for parsing)
- `--plain`: tab-separated values, no headers

## Error Format (--json)

Errors go to stderr:
```json
{"error": "message", "code": "ERROR_CODE"}
```

Exit codes: 0 = success, 1 = API/runtime error, 2 = usage error

---
> Source: [Haknt/lemonsqueezy-cli](https://github.com/Haknt/lemonsqueezy-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
