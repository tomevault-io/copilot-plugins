## dticalendar

> A web-based event/activity calendar for the Department of Trade and Industry (DTI) Caraga regional office. It manages events organized by office → division → unit hierarchy, with geographic tagging (region → province → LGU → barangay), and role-based access control.

# DTI Caraga Calendar of Events System

A web-based event/activity calendar for the Department of Trade and Industry (DTI) Caraga regional office. It manages events organized by office → division → unit hierarchy, with geographic tagging (region → province → LGU → barangay), and role-based access control.

---

## Tech Stack

### Backend
| Component | Version |
|-----------|---------|
| Python | 3.12 |
| Django | 5.2 (installed), settings.py comment says 4.2.4 — ignore the comment |
| PostgreSQL | Default port 5432, db name: `db_appcal` |
| psycopg2-binary | 2.9.11 |
| django-bootstrap4 | 26.1 |

### Frontend (all via local static files, no npm/build step)
| Component | Version |
|-----------|---------|
| Vue.js 3 | CDN global build (`vue.global.js`) |
| Bootstrap | 5.3.2 |
| jQuery | 3.7.0 |
| jQuery UI | Included (datepicker) |
| jQuery DataTables | Included (server-side processing) |

### Key Python packages (no requirements.txt — use `pip freeze`)
- `asgiref` 3.8.1
- `sqlparse` 0.5.3
- `psycopg2-binary` 2.9.11
- `django-bootstrap4` 26.1

---

## Dev Commands

```bash
# Run dev server
python manage.py runserver

# Run on local network (accessible from other machines)
python manage.py runserver 0.0.0.0:8000

# Apply migrations
python manage.py migrate

# Create new migrations after model changes
python manage.py makemigrations

# Create superuser
python manage.py createsuperuser

# Django admin
# http://localhost:8000/admin/

# Dump/load data
python manage.py dumpdata
python manage.py loaddata

# No test suite configured — no test runner commands currently in use
```

---

## Directory Structure

```
dticalendar/
├── dticalendar/          # Django project config
│   ├── settings.py       # All config: DB, email, apps, static/media paths
│   ├── urls.py           # Root URL dispatcher
│   └── wsgi.py           # WSGI entry point
│
├── users/                # Auth, registration, profile, user management
├── events/               # Core Event model and CRUD (most complex app)
├── calendars/            # Calendar grouping for events
├── divisions/            # Division management (belongs to Office)
├── units/                # Unit management (belongs to Division)
├── offices/              # Office management (top-level org unit)
├── employees/            # Employee records (minimal, mostly unused)
├── orgoutcomes/          # Organizational Outcomes (linked to PAPs)
├── paps/                 # Priority Action Plans (linked to OrgOutcomes)
├── regions/              # Region reference data (read-only)
├── provinces/            # Province reference data
├── lgus/                 # Local Government Units
├── barangays/            # Barangay reference data
│
├── static/               # All static assets (JS, CSS — no build step)
│   ├── js/
│   │   ├── custom-vue.js          # Main Vue app: form modals, CRUD, dropdowns
│   │   ├── custom-events-vue.js   # Events display Vue app: DataTables, pills
│   │   ├── vue.global.js          # Vue 3 runtime
│   │   ├── bootstrap5.3.2.bundle.min.js
│   │   ├── jquery-3.7.0.js
│   │   ├── jquery.dataTables.min.js
│   │   └── [jszip, pdfmake, popper, etc.]
│   └── css/
│
├── templates/            # Global templates directory
│   ├── users/
│   │   ├── master.html   # Base template — all JS/CSS loaded here
│   │   ├── profile.html  # Main dashboard (logged-in home page)
│   │   ├── register.html
│   │   ├── login.html
│   │   └── manage_users.html
│   └── registration/     # Django password reset templates
│
├── events/templates/events/
│   ├── event_display.html        # All-events DataTable view
│   ├── events_display_div.html   # Events filtered by division
│   └── events_display_unit.html  # Events filtered by unit
│
└── media/                # User-uploaded file attachments (FileField uploads)
```

---

## Architecture Patterns

### Request/Response Pattern
All CRUD operations are done via AJAX — no traditional form POSTs that redirect. The pattern is:

```python
# views.py
@login_required
@csrf_exempt
def save_something_ajax(request):
    if request.method == 'POST':
        # validate & save
        return JsonResponse({'message': 'True'})
    return JsonResponse({'message': 'False'})
```

Frontend checks `response.message === 'True'` and reloads/updates accordingly.

### Permission Levels
Three tiers, checked inline in each view (no Django permissions framework):

1. **Superuser** — full access to everything
2. **Office Admin** (`UserProfile.is_office_admin = True`) — can manage Division, Unit, OrgOutcome, PAP for their assigned office
3. **Regular User** — can only create/edit events via the profile page

Each app that needs it defines its own `_can_manage(user, office_id)` or `_is_authorized(user)` helper at the top of `views.py`.

### DataTables Server-Side Processing
All list views (Division, Unit, Office, OrgOutcome, PAP, Users) use jQuery DataTables with server-side processing. Views handle `draw`, `start`, `length`, `search[value]`, `order[0][column]`, `order[0][dir]` query params and return `{draw, recordsTotal, recordsFiltered, data}`.

Sorting is case-insensitive via `.annotate(lower_field=Lower('field')).order_by('lower_field')`.

### Vue.js Integration
Vue 3 is used as mounted components (not SPA). Two apps:
- `custom-vue.js` — form management, modal open/close, cascading dropdowns
- `custom-events-vue.js` — events display, pills rendering, date filtering

**Critical:** Vue delimiters are set to `['{[', ']}']` to avoid clashing with Django template `{{ }}` syntax. Always use `{[ variable ]}` in Vue templates embedded inside Django templates.

### Denormalized Fields
Several models store human-readable names alongside FK references (e.g., `Event.division_name`, `Event.unit_name`, `Event.org_outcome`, `Event.paps`). These are populated at save time. This is intentional for display performance — be aware that updating the source record does NOT cascade to these denormalized fields.

### UserProfile Extension
Django's `User` is extended via a `UserProfile` OneToOne model in `users/models.py`:
- `fk_office` — which office this user belongs to
- `is_office_admin` — elevated permissions within their office

Always access via `user.profile.fk_office` or wrap in try/except since new users may not have a profile yet.

---

## URL Structure

| Prefix | App |
|--------|-----|
| `/users/` | Auth, profile, user management |
| `/events/` | Event CRUD and display |
| `/offices/` | Office management |
| `/divisions/` | Division management |
| `/units/` | Unit management |
| `/orgoutcomes/` | Org Outcomes |
| `/paps/` | PAPs |
| `/provinces/` | Province reference API |
| `/lgus/` | LGU reference API |
| `/barangays/` | Barangay reference API |
| `/admin/` | Django admin (Office model only) |

API endpoints follow the pattern `/app/api/get-modelList/` for JSON list fetches and `/app/save-model-ajax/` for CRUD.

---

## Database

- **Engine:** PostgreSQL
- **DB name:** `db_appcal`
- **User:** `postgres`
- **Host:** `localhost:5432`
- Credentials are hardcoded in `settings.py` (dev environment — do not commit production credentials)
- `DEFAULT_AUTO_FIELD = BigAutoField` (all PKs are BigInt)
- `Office` uses a custom `db_table = 'tbl_offices'`

---

## Email (Password Reset)

- Gmail SMTP via `dtiictunit@gmail.com`
- Configured in `settings.py` (credentials hardcoded)
- Password reset flow uses Django's built-in views at `/users/password-reset/`

---

## Static & Media Files

- **Static:** Served directly from `/static/` during development (`STATICFILES_DIRS`). No `STATIC_ROOT` set — `collectstatic` is not configured for production.
- **Media:** Uploaded files go to `/media/media/` (double `media/` due to `upload_to='media/'` with `MEDIA_ROOT` already pointing to `media/`).

---

## Gotchas

- **No `requirements.txt`** — use `pip freeze` to see installed packages. If setting up fresh, install: `Django`, `psycopg2-binary`, `django-bootstrap4`.
- **Django version mismatch:** `settings.py` was generated with a 4.2.4 comment but the running version is **5.2**. The code works on 5.2.
- **Double `media/` path:** `MEDIA_ROOT = .../media` + `upload_to='media/'` = files land at `.../media/media/filename`. Don't change one without the other.
- **`@csrf_exempt` on AJAX views:** Most save/delete AJAX endpoints bypass CSRF. This is the established pattern — don't add CSRF tokens to these calls.
- **Vue delimiter conflict:** Django templates and Vue both use `{{ }}`. This project uses `{[ ]}` for Vue. Never use standard Vue `{{ }}` syntax in `.html` files.
- **Denormalized name fields:** When creating or updating a Division, Unit, OrgOutcome, etc., also update the `_name` denormalized field on related Event records if needed (currently this is NOT done automatically).
- **No test suite:** There are no tests. Adding new features should include manual testing at minimum.
- **`employees` app is minimal:** The Employee model exists and has a form but is not used by Event creation — events link to User (Django auth user), not Employee.
- **`regions` app has no views:** Region data is pre-seeded and only used as a FK parent for Provinces. No API endpoint exists for regions.
- **Duplicate URL includes in root `urls.py`:** `calendars.urls` and `divisions.urls` are included twice at different prefixes. Be careful when adding new URL names to avoid routing ambiguity.
- **`is_office_admin` check requires `user.profile`:** This will raise `RelatedObjectDoesNotExist` if a user has no `UserProfile`. All `_can_manage` / `_is_authorized` helpers wrap this in try/except — follow that pattern.

---

## Coding Standards (observed in codebase)

- **Views:** All function-based views (FBV). No class-based views used.
- **AJAX responses:** Always `JsonResponse({'message': 'True'})` or `{'message': 'False'}` — string booleans, not actual booleans.
- **Decorators:** `@login_required` for auth, `@csrf_exempt` for AJAX endpoints, `@require_GET` for read-only endpoints.
- **Forms:** `ModelForm` used where forms exist (`EmployeeForm`, `UserRegisterForm`). Most CRUD goes through raw `request.POST` in AJAX views.
- **Model timestamps:** Newer models use `auto_now_add` / `auto_now` on `DateTimeField`. Older models use `DateField(default=date.today)` manually.
- **Imports:** Standard Django imports. No third-party ORM tools.
- **Template inheritance:** All pages extend `users/templates/users/master.html` which loads all JS/CSS.
- **No type hints** used anywhere in the codebase.
- **Variable naming:** `snake_case` throughout Python. JS uses `camelCase`.

---

## Self-Improvement Loop

> Rules added after real mistakes during development. Review at the start of every session. Every entry = a mistake that must never repeat.

### CSS

- **DataTables `className` in `columnDefs` applies to BOTH `thead th` AND `tbody td`.** Never write a bare utility class (e.g. `.bold-column { color: X }`) and then assign it via `columnDefs`. The color will override the `thead th` style. Always scope utility classes to `tbody td` — e.g. `.dataTables_wrapper table.dataTable tbody td.bold-column { ... }`.

- **DataTables sort icons are hidden by `opacity: .125`, not by `color`.** Setting `color` on sort icon pseudo-elements does nothing visible. Always override `opacity: 1` alongside `color` to make them visible. Cover all three states: `sorting`, `sorting_asc`, `sorting_desc` (both `:before` and `:after`).

- **`overflow: hidden` on any ancestor blocks `position: sticky`.** It creates a scroll container, trapping the sticky element inside it. To get sticky `thead` headers, never put `overflow: hidden` on `.dataTables_wrapper` or the `table` itself. Put borders directly on the `table` element and avoid `overflow: hidden` on ancestors entirely.

- **CSS `position: sticky` on `thead th` is unreliable with `border-collapse: collapse`.** Do not use CSS sticky for DataTables frozen headers. Use DataTables' native `scrollY` + `scrollX` + `scrollCollapse` options instead — they create a `.dataTables_scrollHead` div that reliably pins the header. Also style `.dataTables_scrollHead` and `.dataTables_scrollHead table.dataTable thead th` to match the custom header colors, since DataTables clones the thead into that div.

- **When adding a column to a DataTable, always update `<thead>` and `<tfoot>` in the HTML template to match.** DataTables requires `<th>` count to exactly match the `columns` array. A mismatch causes a silent error — the loading spinner never hides and data never renders.

- **Never put `<script>` tags inside Vue's mount element.** Vue 3 strips `<script>` and `<style>` tags inside its template. Constants like `IS_SUPERUSER` defined this way will be undefined at runtime. Use a `{% block page_scripts %}` block in master.html placed *before* the Vue mount div so scripts render outside Vue's template.

### Dependencies

- **`requirements.txt` must be kept in sync with `events/views.py` imports.** When a new library is added to views (e.g. `openpyxl`, `reportlab`), add it to `requirements.txt` immediately or the server won't start. Check `events/views.py` imports whenever pulling a PR that touches that file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/j-curato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
