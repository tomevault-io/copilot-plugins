## ndas

> Guidance for Claude Code (claude.ai/code) working in this repository.

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

**Last Updated:** 2025-12-25

<!-- OPENSPEC:START -->
## OpenSpec Instructions

For planning, proposals, or architecture changes, see `openspec/AGENTS.md` for:
- Change proposal workflow
- Spec format and conventions
- Validation commands

<!-- OPENSPEC:END -->

## Project Overview

**NDAS** - Django medical system for patient records, video-based neurodevelopmental assessments, and evaluation workflows.

**Stack:** Django 4.2.16 | PostgreSQL/SQLite | AdminLTE 3.2 | Bootstrap 4.6 | HTMX | Video.js

**Apps:** `patients/` (root URL), `users/`, `video/`, `reports/`, `problemlist/`

## Quick Commands

```bash
# Environment (Windows)
venv\Scripts\activate && pip install -r requirements.txt

# Database
python manage.py makemigrations [app] && python manage.py migrate

# Run & Test
python manage.py runserver
python manage.py test [app_name]
```

## Architecture Patterns

### Model Pattern (MANDATORY)

```python
from ndas.custom_codes.Custom_abstract_class import TimeStampedModel, UserTrackingMixin

class MyModel(TimeStampedModel, UserTrackingMixin):
    # Auto-provides: created_at, updated_at, added_by, last_edit_by
    # User tracking auto-populated by UserActivityMiddleware
    pass
```

### Custom Codes Directory (`ndas/custom_codes/`)

| File | Purpose |
|------|---------|
| `Custom_abstract_class.py` | Base models (TimeStampedModel, UserTrackingMixin) |
| `choice.py` | All TextChoices for dropdowns |
| `validators.py` | Field validators, `sanitize_text_input()`, `sanitize_filename()` |
| `sanitization.py` | HTML sanitization with bleach (`sanitize_html()`, `sanitize_plain_text()`) |
| `custom_methods.py` | Utilities: `getCountZeroIfNone()`, `calculate_age_string()`, `extract_video_metadata()` |
| `ndas_enums.py` | Enumerations (e.g., `PtStatus`) |
| `delete_helpers.py` | Entity deletion utilities with permission/business rule checks |
| `security_middleware.py` | Custom security headers and CSP middleware |
| `error_handlers.py` | View error handling decorators: `@handle_view_errors()`, `@log_and_suppress()` |

### View Pattern

```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_GET, require_http_methods
from django_ratelimit.decorators import ratelimit

@login_required(login_url="user-login")
@require_http_methods(["GET", "POST"])
@ratelimit(key='user_or_ip', rate='10/m')
def my_view(request, pk):
    obj = get_object_or_404(MyModel, id=pk)  # ALWAYS use get_object_or_404
    related = Related.objects.filter(parent=obj).select_related('added_by')
    return render(request, "app/template.html", {"obj": obj, "related": related})
```

### Template Pattern

```django
{% extends 'src/base.html' %}
{% load static %}
{% block title %}Section - Action{% endblock %}
{% block main_content %}
<div class="container-fluid">{% csrf_token %}<!-- Content --></div>
{% endblock %}
```

- Extend `'src/base.html'` (authenticated) or `'src/basic_plane.html'` (public)
- Naming: `manager.html` (list), `add.html` (create), `edit.html` (update), `view.html` (detail)
- **DO NOT** change CSS framework (AdminLTE 3.2 + Bootstrap 4.6 + Font Awesome 6.4)

## Patient Model Fields (CRITICAL)

Common field name errors to avoid:

```python
# Identifiers
patient.bht              # NOT bht_number
patient.nnc_no           # NOT nnc_number
patient.baby_name        # NOT patient_name or name
patient.dob_tob          # NOT date_of_birth or dob

# Birth Data
patient.pog_wks          # NOT gestational_age_weeks
patient.pog_days         # NOT gestational_age_days
patient.birth_weight     # NOT birth_weight_g
patient.hc               # NOT head_circumference

# APGAR
patient.apgar_1          # NOT apgar_1_min
patient.apgar_5          # NOT apgar_5_min
```

## Security Architecture

### Middleware Stack (Order Critical)

1. SecurityMiddleware
2. WhiteNoiseMiddleware
3. CSPMiddleware
4. AdditionalSecurityHeadersMiddleware (custom)
5. SessionMiddleware
6. CommonMiddleware
7. CsrfViewMiddleware
8. AuthenticationMiddleware
9. UserActivityMiddleware (custom - auto-tracks user changes)
10. MessageMiddleware
11. XFrameOptionsMiddleware
12. UserAgentMiddleware
13. SubscriptionCheckMiddleware (custom)
14. SecurityHeadersValidationMiddleware (production only)

### Security Features

- **Rate Limiting**: 24 CRUD operations protected (10/min create/edit, 5/min delete)
- **Input Sanitization**: `sanitize_text_input()` removes XSS, preserves medical notation
- **File Validation**: MIME type checking via python-magic, size limits enforced
- **HTTP Methods**: `@require_GET`, `@require_http_methods(["GET", "POST"])`
- **Session**: 1 hour timeout, browser-close expiry

### File Upload Limits

| Type | Max Size | Extensions |
|------|----------|------------|
| Video | 2GB | mp4, mov, avi, mkv, webm |
| Image | 10MB | jpg, jpeg, png, gif, bmp, webp |
| Document | 100MB | doc, docx, txt, rtf, odt, pdf |
| Profile Picture | 5MB | (images) |

## Medical Domain

**Identifiers:** BHT, NNC, PTC, PC, PIN, Disk No.
**Assessments:** GPA, HINE, CDIC, Developmental

**Validation Ranges:**
- Birth weight: 300g-8000g (basic), POG-specific (enhanced)
- APGAR scores: 0-10
- Gestational age: 20-44 weeks + 0-6 days

## Key Rules

**Always:**
- Inherit from `TimeStampedModel, UserTrackingMixin`
- Add choices to `choice.py`, validators to `validators.py`
- Use `db_index=True` for searchable fields
- Use `get_object_or_404()` not `.objects.get()`
- Use `select_related()`/`prefetch_related()` for related objects
- Include CSRF tokens, validate file uploads

**Never:**
- Add choices inline in models
- Change CSS framework or Bootstrap version
- Reorder security middleware
- Store secrets in code (use `.env`)

## Reports Module

**PDF** (`reports/utils/pdf_generator.py`): `BasePDFGenerator`, `PatientPDFGenerator`, assessment-specific generators
**Excel** (`reports/utils/excel_generator.py`): `ExcelReportGenerator` with anonymization, filtering, multi-sheet support

## Delete System

```python
from ndas.custom_codes.delete_helpers import (
    has_delete_permission, validate_can_delete,
    get_entity_warning_items, get_entity_detail_items
)
```

**Rules:** Superusers delete any; Staff delete own records; Videos blocked if in assessments.

**Template:** `{% load delete_modal_tags %}` + `{% include 'src/partials/delete_confirmation_modal.html' %}`

## Quick Reference

```python
# Core imports
from ndas.custom_codes.Custom_abstract_class import TimeStampedModel, UserTrackingMixin
from ndas.custom_codes.custom_methods import getCountZeroIfNone, calculate_age_string
from ndas.custom_codes.validators import sanitize_text_input, sanitize_filename, validate_birth_weight
from ndas.custom_codes.sanitization import sanitize_html, sanitize_plain_text
from ndas.custom_codes.ndas_enums import PtStatus
from ndas.custom_codes.delete_helpers import has_delete_permission, validate_can_delete
from ndas.custom_codes.error_handlers import handle_view_errors

# Django essentials
from django.shortcuts import render, redirect, get_object_or_404
from django_ratelimit.decorators import ratelimit
from django.views.decorators.http import require_GET, require_http_methods

# Settings access
from django.conf import settings
settings.FILE_UPLOAD_LIMITS['VIDEO_MAX_SIZE']  # 2GB
settings.ALLOWED_FILE_EXTENSIONS['VIDEO']      # ['.mp4', '.mov', ...]
```

## Environment Variables

```bash
SECRET_KEY=your-secret-key
DEBUG=True/False
ALLOWED_HOSTS=localhost,127.0.0.1

# Database (optional - defaults to SQLite)
DB_ENGINE=django.db.backends.postgresql
DB_NAME=ndas
DB_USER=user
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432

# Cache (optional - defaults to LocMem)
REDIS_URL=redis://localhost:6379/0

# Security (production)
SECURE_SSL_REDIRECT=True
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rasikakulasinghe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rasikakulasinghe)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
