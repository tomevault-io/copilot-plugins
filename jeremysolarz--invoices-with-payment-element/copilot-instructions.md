## invoices-with-payment-element

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Flask-based Stripe payment integration demonstrating Custom Checkout (Payment Element) with automatic invoice creation. The application uses Stripe's `ui_mode='custom'` for embedded checkout experiences.

## Architecture

### Backend (server.py)
- **Flask server** serving static files from `public/` directory
- **Three main endpoints**:
  - `/config` - Returns Stripe publishable key
  - `/create-checkout-session` - Creates a Stripe Checkout Session with:
    - Custom UI mode (`ui_mode='custom'`)
    - Automatic invoice creation enabled with metadata
    - Billing and shipping address collection
    - Automatic tax calculation enabled
    - Returns `clientSecret` for frontend initialization
  - `/session-status` - Retrieves session and payment intent status after payment

### Frontend Flow
1. **checkout.js** - Initializes the payment flow:
   - Fetches publishable key from `/config`
   - Creates Checkout Session via `/create-checkout-session`
   - Uses `stripe.initCheckout()` for Custom Checkout
   - Mounts three Elements: Payment, Billing Address, Shipping Address
   - On submit, calls `actions.confirm()` which redirects to `return_url`

2. **complete.js** - Handles payment completion:
   - Extracts `session_id` from URL query params
   - Calls `/session-status` to retrieve payment status
   - Displays success/failure UI with payment details

### Key Stripe Integration Details
- Uses **Stripe API version**: `2025-11-17.clover` (configurable via env var)
- **Invoice creation** is automatic on successful payment with custom metadata (`order_id`)
- Shipping restricted to Germany (`allowed_countries: ['DE']`)
- Hardcoded customer email: `customer@example.com` (should be customized for production)

## Development Commands

### Run with Docker (Recommended)
```bash
docker-compose up --build
```

### Run locally
```bash
pip3 install -r requirements.txt
python3 server.py
```

### Access the application
```
http://localhost:4242/checkout.html
```

## Environment Configuration

Required variables in `.env`:
- `STRIPE_SECRET_KEY` - Your Stripe secret key (sk_test_* or sk_live_*)
- `STRIPE_PUBLISHABLE_KEY` - Your Stripe publishable key (pk_test_* or pk_live_*)
- `PRICE` - The Stripe Price ID for the product being sold
- `DOMAIN` - Application domain (default: http://localhost:4242)
- `STRIPE_API_VERSION` - Optional, defaults to '2025-11-17.clover'

## Important Implementation Notes

### When modifying checkout flow:
- The Checkout Session uses `ui_mode='custom'` - this requires using `stripe.initCheckout()` on the frontend, NOT `stripe.confirmPayment()`
- The `return_url` must include `{CHECKOUT_SESSION_ID}` placeholder for Stripe to inject the session ID
- Address collection settings in `server.py:36-37` must match the Elements mounted in `checkout.js`

### When modifying invoice generation:
- Invoice settings are in `create-checkout-session` endpoint under `invoice_creation` parameter
- `invoice_data` accepts `description`, `metadata`, `account_tax_ids`, etc.
- Invoice is created automatically on successful payment - no webhook needed for basic flow

### When debugging payment issues:
- Check browser console for Stripe.js errors
- Backend prints Stripe secret key on startup (line 17) - verify it loads correctly
- Use `/session-status` endpoint to inspect session state after payment
- Payment Intent ID is available in complete page for Stripe Dashboard lookup

### File structure conventions:
- All frontend files live in `public/` directory
- Flask serves static files directly (configured at `server.py:19-21`)
- No build step required - pure HTML/CSS/JS

## Testing

This is a sample application without formal tests. For manual testing:
1. Use [Stripe test cards](https://stripe.com/docs/testing#cards)
2. Test card: `4242 4242 4242 4242` (any future date, any CVC)
3. Verify invoice creation in Stripe Dashboard after successful payment
4. Check payment appears under Payments and Invoices sections

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jeremysolarz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jeremysolarz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
