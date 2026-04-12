## property-administration

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Django 4.2.4 property management web application backed by PostgreSQL. Single Django app (`mysite`) with Tailwind CSS frontend, Twilio for SMS/conversations, DocuSeal for contracts, Telegram bots for notifications, and Open Router integration.

## Commands

### Development

```bash
# Start Django dev server
python ./start --no-sync

```

### Cron / Process Management

```bash
pm2 start pm2.config.js    # start cron.js (runs management commands daily at 8am)
pm2 logs telegram-notifications
```


## Architecture

### Single-App Django Structure

All code lives in the `mysite/` Django app. No separate apps — models, views, forms, middleware, and management commands are all in `mysite/`.

```
mysite/
  models.py                      # All Django models (single large file)
  forms.py                       # All forms with CustomFieldMixin
  urls.py                        # All URL routes
  views/                         # Views split by feature area
    generic_view.py              # CRUD pattern shared by most pages
    utils.py                     # MODEL_MAP, parse_query, handle_post_request
    chat.py, messaging.py        # Twilio conversations
    payments_report.py, payment_sync.py, payment_sync_v2.py
    apartments_view.py, apartments_report.py
    booking_report.py, booking_availability.py
    ...
  management/commands/           # Django management commands (~30 commands)
  base_models.py                 # BaseModelWithTracking abstract model
  unified_logger.py              # Single logging system (DB + Telegram + file)
  decorators.py                  # @user_has_role(*roles) decorator
  request_context.py             # Thread-local current user for model tracking
  request_context_middleware.py  # Sets request context per request
  docuseal_contract_managment.py # DocuSeal API integration
  telegram_logger.py             # Telegram notification helpers
templates/                       # Django HTML templates
static/
  input.css                      # Tailwind source
  output.css                     # Compiled Tailwind (do not edit directly)
cron.js                          # Node.js cron scheduler (runs via PM2)
pm2.config.js                    # PM2 process config for cron.js
```

### CRUD Pattern

Most entity pages follow this pattern:
1. View function in `views/generic_view.py` calls `generic_view(request, model_key, FormClass, template)`
2. `handle_post_request()` in `views/utils.py` dispatches `add`/`edit`/`delete` POST actions
3. `MODEL_MAP` in `views/utils.py` maps URL model keys to model classes
4. `parse_query()` handles the `?q=field.related.id=value` filter syntax used in cross-page links

### User Tracking Pattern

Every model save automatically records `created_by` and `last_updated_by` (stored as display strings, not FKs). The mechanism:
- `RequestContextMiddleware` stores the current request in thread-local `_request_context`
- `apply_user_tracking()` in `request_context.py` reads the thread-local to set tracking fields
- Each model's `save()` calls `apply_user_tracking(self, updated_by)` before `super().save()`
- New models should inherit from `BaseModelWithTracking` in `base_models.py`

### Logging

Use `unified_logger.py` — do NOT use `error_logger.py` or `telegram_logger.py` directly (they are legacy).

```python
from mysite.unified_logger import log_error, log_info, log_warning, logger

log_error(e, "Context description", severity='high', source='web')
log_info("Event description", category='payment', details={'amount': 100})
log_warning("Warning message", category='sms')
```

Errors are stored in the `ErrorLog` DB model and sent to Telegram for `high`/`critical` severity. Duplicate errors within one hour are deduplicated and only the occurrence count is incremented.

### Access Control

Views are protected with `@user_has_role(*roles)` from `decorators.py`. Roles: `Admin`, `Manager`, `Cleaner`, `Tenant`, `Owner`. Admin-only deletions are enforced in `handle_post_request()`.

### Key Models

- `User` — custom auth model (`AUTH_USER_MODEL = 'mysite.User'`), login via email, roles determine access
- `Apartment` — core entity; has managers (M2M), owner (FK to User)
- `ApartmentPrice` — price history with `effective_date`; `current_price` property returns latest
- `Booking` — links Apartment + Tenant; statuses: Confirmed, Waiting Contract, Waiting Payment, Blocked, Pending, Problem Booking
- `Payment` — links to Booking or Apartment directly; has PaymentMethod and PaymenType (note: intentional typo in model name `PaymenType`)
- `Cleaning` — cleaning schedule linked to Booking
- `Notification` — scheduled SMS/email notifications
- `TwilioConversation` / `TwilioMessage` — mirrors Twilio Conversations data in DB
- `ChatMessageTemplate` — reusable message templates for the chat interface
- `ErrorLog` / `SystemLog` — unified logging DB storage
- `CalendarNote` — notes displayed on booking availability calendar (global or per-apartment)
- `HandymanCalendar`, `Parking`, `ParkingBooking` — auxiliary scheduling models

### External Integrations

| Service | Env vars | Usage |
|---------|----------|-------|
| PostgreSQL | `PGDATABASE`, `PGUSER`, `PGPASSWORD`, `PGHOST`, `PGPORT` | Primary database |
| Twilio | `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` | SMS, Conversations (chat) |
| DocuSeal | `DOCUSEAL_API_KEY`, `DOCUSEAL_TEMPLATE_ID` | Lease contracts |
| Telegram | `TELEGRAM_TOKEN`, `TELEGRAM_CHAT_ID`, `TELEGRAM_ERROR_CHAT_ID`, group chat IDs | Notifications & error alerts |
| Open Router | `OPENROUTER_API_KEY` | GPT integration |
| Google APIs | `GOOGLE_OAUTH2_TOKEN_FILE`, `MANAGER_GMAIL` | Google Docs/email |


### Phone Numbers

All phone numbers are stored and validated in E.164 format (`+12025551234`). The canonical validation function is `validate_and_format_phone()` in `models.py`. US numbers without country code are assumed to be `+1`. Invalid phones raise `ValueError` on model save.

### Frontend

- **CSS**: Tailwind v3 compiled from `static/input.css` → `static/output.css`. Edit `input.css` or use Tailwind classes in templates; never edit `output.css` directly.
- **UI Components**: Flowbite (Tailwind component library)
- **Templates**: Standard Django templates in `templates/`. The `slippers` package provides component-style template tags.
- **Static serving**: Whitenoise in production; Django dev server in development.
- **CSRF**: CSRF middleware is disabled (commented out in `settings.py`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andyvauliln)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/andyvauliln)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
