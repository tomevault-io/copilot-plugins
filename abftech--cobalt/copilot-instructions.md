## cobalt

> Cobalt is a Django web application for the **Australian Bridge Federation (ABF)** to administer the card game of contract bridge. It is deployed at [myabf.com.au](https://www.myabf.com.au).

# Cobalt — Claude Code Guide

## What This Project Is

Cobalt is a Django web application for the **Australian Bridge Federation (ABF)** to administer the card game of contract bridge. It is deployed at [myabf.com.au](https://www.myabf.com.au).

It handles: club and organisational management, member payments, event entry, masterpoints, results/scoring, discussion forums, push notifications, and Xero accounting integration.

---

## Tech Stack

| Concern | Technology |
|---|---|
| Framework | Django 5.2 |
| Database | PostgreSQL via AWS RDS (psycopg2) |
| Frontend | Bootstrap 4 + **HTMX** |
| Rich text editor | Summernote |
| Payments | Stripe |
| Email delivery | AWS SES via django-ses + django-post-office (queuing) |
| Push notifications | Firebase/FCM (fcm-django) |
| File storage | AWS S3 (boto3) |
| PDF generation | ReportLab |
| Excel export | XlsxWriter |
| Monitoring | New Relic |
| WSGI server | Gunicorn (5 workers, 8 threads) |
| REST API | Django Ninja |
| 2FA | Django OTP |
| Background jobs | **Cron** (Django management commands — no Celery) |
| Deployment | AWS Elastic Beanstalk (no Docker) |

---

## Coding Conventions

### Views: function-based by default

Prefer **function-based views** (FBVs). Class-based views (CBVs) are a last resort and should only be used when Django's generic CBVs provide significant boilerplate reduction that cannot be achieved otherwise.

### HTMX is the primary interactivity mechanism

We use HTMX extensively for partial page updates instead of a full SPA framework. Most interactive UI flows use HTMX posts to views that return HTML fragments. Templates are often split into a "full page" template and an `_htmx.html` fragment variant.

#### Confirmation dialogs — use SweetAlert2, not `confirm()`

Never use the browser's native `confirm()` for destructive actions. Use SweetAlert2 instead — use the **cobalt-htmx-confirm** skill to add a confirmation dialog.

#### Suppressing the "leave page?" prompt

`cobalt-core.js` registers a `beforeunload` handler that warns users about unsaved form changes. Pages that manage their own state (e.g. HTMX-driven admin tools) should suppress it by adding `<span id="ignore_cobalt_save" hidden></span>` anywhere in the template.

### Models stay thin

Keep models as short as possible. They should define fields, `__str__`, simple properties, and `Meta`. **Business logic belongs in views** (or in `*_core.py` helper modules alongside the app's views), not in model methods.

### Register all new models in admin.py

Every new model must be added to the app's `admin.py` file so it appears in the Django admin. Add the model to the import line and call `admin.site.register(MyModel)` for it.

### No Celery — use cron + management commands

Background/scheduled work is handled by Django management commands run by cron on the EC2 instances. See examples in `notifications/management/commands/` and `organisations/management/commands/`.

### Batch job pattern

Every management command **must** use `BatchStatus` + `CobaltLock` — use the **cobalt-batch-command** skill to scaffold a new command. Reference implementation: `organisations/management/commands/auto_pay_batch.py`.

### Configuration via environment variables

There is a single `cobalt/settings.py`. All environment-specific configuration is supplied via environment variables (managed by Elastic Beanstalk on AWS, and by a local `.env` or shell exports in development). Never hardcode secrets or environment-specific values.

---

## Project Structure

```
cobalt/                  # Django project root (settings, urls, wsgi)
accounts/                # Custom User model and authentication
api/                     # REST API (Django Ninja)
calendar_app/            # Calendar
club_sessions/           # Club session management
dashboard/               # User dashboard
events/                  # Event entry and management
forums/                  # Discussion forums
logs/                    # Application event logging
masterpoints/            # ABF masterpoints tracking
notifications/           # Email and push notification delivery
organisations/           # Clubs and state/national organisations
payments/                # Stripe payment processing and Bridge Credits
rbac/                    # Role-based access control
results/                 # Results and scoring
support/                 # Help desk and support tools
tests/                   # Test infrastructure (custom runner, test data)
utils/                   # Shared utilities, middleware, AWS helpers
xero/                    # Xero accounting integration
```

Each app typically has:
- `models.py` — data models (kept thin)
- `views/` — sub-module with FBVs split by feature area, plus `*_core.py` for shared business logic
- `urls.py` — URL routing
- `templates/<app>/` — HTML templates
- `management/commands/` — cron-invoked management commands
- `tests/unit/` and `tests/integration/` — test classes

---

## AWS Infrastructure

- **Elastic Beanstalk** — application hosting (no Docker; EB-native Python platform)
- **RDS (PostgreSQL)** — primary database (`RDS_*` environment variables)
- **SES** — transactional email delivery (`AWS_SES_*` settings)
- **SNS** — used for push notification infrastructure
- **S3** — media file storage

The deployment hostname is exposed as `COBALT_HOSTNAME`. Management commands that could be destructive check this value and raise `SuspiciousOperation` if accidentally run against production (`myabf.com.au` / `www.myabf.com.au`).

---

## Access Control (RBAC)

The `rbac` app provides fine-grained role-based access control. Roles follow the pattern:

```
<app>.<resource>.<object_id>.<action>
# e.g. orgs.members.42.edit
```

Views use decorator helpers like `@check_club_menu_access(check_members=True)` to gate access. Always check RBAC rather than relying on session state alone.

---

## Coding Style and Pre-commit Hooks

Code is formatted and validated automatically via **git pre-commit hooks**. Do not skip them (`--no-verify`). The hooks run:

- **black** — Python code formatting (Python 3.13)
- **flake8** — Python linting
- **djhtml** — Django template indentation
- **debug-statements** — prevents accidental committed `breakpoint()` / `pdb` calls

Run `pre-commit install` after cloning. To run hooks manually: `pre-commit run --all-files`.

### Python version

The minimum supported Python version is **3.13**. Write code that targets 3.13+ features freely — do not add compatibility shims for older versions.

### String formatting

Always use **f-strings** for string interpolation. Do not use `%` formatting or `str.format()`. Do not concatenate strings with `+` when an f-string can be used instead.

### List comprehensions

Only use list comprehensions (and dict/set comprehensions) if the expression fits on a **single line**. If it would need to wrap, use an explicit `for` loop instead.

---

## Testing

Tests use a **custom framework** (no pytest) built on top of Django's test client and Selenium.

Run tests via management commands:
```bash
# Unit tests
python manage.py run_tests_unit --app <appname>

# Integration / Selenium tests
python manage.py run_tests_integration --app <appname> --browser chrome --headless

# Smoke tests
python manage.py run_tests_smoke --script <script_name>
```

Test data is seeded with `python manage.py add_test_data`. Pre-populated test users have numeric system numbers (100–115) and password `F1shcake`.

Each app has tests in `<app>/tests/unit/` (unit) and `<app>/tests/integration/` (Selenium/client). See `tests/test_manager.py` for the test framework core.

Use the **cobalt-unit-test** skill to scaffold new test files. See `xero/tests/unit/unit_test_xero_api.py` for the reference pattern on mocking/live toggle for external API tests.

---

## Key Domain Concepts

- **System Number** — the ABF membership number, used as the primary identifier for people throughout the app (not Django's `pk`). Stored on the custom `User` model as `system_number`.
- **Bridge Credits** — an internal virtual currency used for payments within the platform.
- **Membership States** — `CURRENT`, `FUTURE`, `DUE`, `LAPSED`, `ENDED`, `RESIGNED`, `TERMINATED`, `DECEASED`.
- **Auto Pay** — automatic payment of membership fees from Bridge Credits on a scheduled date.
- **Unregistered Users** — people in the system (e.g. contacts imported from clubs) who have not yet created an account. They have a sentinel email `noemail@notset.com`.
- **GLOBAL_ORG** — `"ABF"` — the top-level organisation.
- **does_not_renew** — a membership type flag meaning the membership never expires (lifetime/life member).

---

## Environments

| Name | Hostname |
|---|---|
| Development | `127.0.0.1:8000` |
| Test | test.myabf.com.au |
| UAT | uat.myabf.com.au |
| Production | www.myabf.com.au |

`DISABLE_PLAYPEN=OFF` on non-production environments prevents real emails from being sent. Set `DISABLE_PLAYPEN=ON` only on production.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abftech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abftech)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
