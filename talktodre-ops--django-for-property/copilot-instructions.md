## django-for-property

> Heimly is a property listing platform with verify-by-default model. We verify owner identity (KYC), property documents, and all related documents before listings go live. Target market: Nigeria/Africa.

# Heimly Property Listing Platform - Cursor AI Rules

## Project Context
Heimly is a property listing platform with verify-by-default model. We verify owner identity (KYC), property documents, and all related documents before listings go live. Target market: Nigeria/Africa.

## Core Philosophy: Django-First, External-Last

**CRITICAL RULE**: Use Django's built-in batteries before suggesting external services. any installation that is needed, output the instructions and the command.
DO NOT INSTALL ANYTHING YOURSELF 

### What Django Provides (Use These First):
- ✅ Built-in authentication system (NOT Supabase Auth)
- ✅ Built-in admin interface (NOT custom admin panels)
- ✅ Built-in forms system (NOT raw HTML forms)
- ✅ Built-in template engine
- ✅ Built-in messages framework
- ✅ Built-in file upload handling (FileField/ImageField)
- ✅ Built-in email backend (console for dev, SMTP for prod)

### External Services (ONLY When Forced):
- Supabase Postgres (only when deploying - Railway doesn't persist SQLite)
- Supabase Storage (only when deploying - Railway doesn't persist files)
- SMTP provider (Gmail/other - only when need real emails)

### What We DON'T Need (MVP):
❌ Supabase Auth, Realtime, Edge Functions
❌ Redis, Celery, Task queues
❌ React/Vue - Django templates are fine
❌ Docker, Kubernetes
❌ GraphQL, JWT tokens, OAuth
❌ AI/ML services for face verification

## Engineering Principles (MANDATORY)

### 1. O(1) Performance Mindset
- **Database operations MUST be O(1) or O(log n) at worst**
- Always add `db_index=True` to frequently queried fields: `owner_id`, `status`, `city`, `id_number`
- Use `select_related()` and `prefetch_related()` to eliminate N+1 queries
- Example: Dashboard views must load all data in ≤3 queries using prefetch
- Core operations (save draft, toggle status, fetch dashboard) must be constant time

### 2. World-Class Code Quality
- **Type hints**: Use them for all function signatures, especially service methods
- **Docstrings**: Every class, function, and complex method must have clear docstrings
- **Linting**: Code must pass flake8/black/isort checks
- **Tests**: Critical verification flows MUST have tests (even basic smoke tests)
- **Meaningful names**: `submit_for_review()` not `do_stuff()`
- Code quality standards apply from day one, even in local development

### 3. Deterministic Processes
- Use explicit state machines/services, NOT ad-hoc conditionals
- Status transitions must go through service methods (e.g., `VerificationService.submit_listing()`)
- All state changes must be auditable (use AuditEntry model)
- Avoid magic strings - use model enums/choices
- Example: `listing.status = ListingStatus.VERIFIED` not `listing.status = 'verified'`

## Code Style & Structure

### Models
- Use `Meta.indexes` for composite indexes on commonly filtered fields
- Use `Meta.constraints` for unique constraints (e.g., one primary photo per listing)
- Always include `created_at` and `updated_at` timestamps
- Use `db_index=True` on foreign keys used in filters
- Use TextChoices for status fields (e.g., `class ListingStatus(models.TextChoices)`)

### Views
- Prefer class-based views (CreateView, UpdateView, DetailView) over function views
- Use `@login_required` or `LoginRequiredMixin`
- Use `get_queryset()` to filter by user ownership
- Always use `select_related()` and `prefetch_related()` in querysets
- Return structured errors, not generic messages

### Forms
- Use `ModelForm` when possible - Django generates 90% of form code
- Custom validation in `clean_<field>()` methods
- Use `forms.Textarea`, `forms.CheckboxSelectMultiple` for UX
- File uploads: validate file type/size in form, not just model

### Templates
- Use template inheritance (`{% extends 'base.html' %}`)
- Use Django's `{% csrf_token %}` (never skip CSRF)
- Use Django messages framework (`{% if messages %}`)
- Keep templates DRY - extract repeated blocks

### Services
- Create service classes for complex business logic (e.g., `VerificationService`)
- Services handle state transitions, validation, notifications
- Keep views thin - delegate to services
- Services must be testable in isolation

## Verification Flow Requirements

### Owner KYC (MVP - Simplified)
- Government ID (NIN, Passport, Driver's License) - front photo only
- Email verification OR Phone verification (at least one)
- Full name matching ID
- NO selfie/face verification (too complex for MVP)
- NO bank statements (too stiff for MVP)
- NO handwritten codes (too much friction)

### Property Verification
- Required documents: C of O, Deed, Utility Bill, Tax Receipt (upload what you have)
- Photos: Minimum 3-5 photos (one primary)
- Location: Address required, GPS optional but preferred
- Manual review by staff via Django admin

### Listing Status Flow
1. `draft` - User creating, can save incomplete
2. `pending_identity` - Draft saved, owner KYC incomplete
3. `pending_documents` - Identity OK, documents incomplete
4. `in_review` - All requirements met, under staff review
5. `verified` - Approved, goes live with full visibility
6. `rejected` - Rejected (with reason)
7. `archived` - User/admin archived

## File Storage

### Development (SQLite)
- SQLite stores ONLY metadata (file paths)
- Actual files go to `MEDIA_ROOT = BASE_DIR / 'media'`
- Django's `FileField`/`ImageField` handle this automatically
- No cloud storage needed until deployment

### Production
- Switch to Supabase Storage (S3-compatible)
- Change `DEFAULT_FILE_STORAGE` setting only
- Database paths remain valid - same relative paths work

## Testing Requirements

### Must Test:
- Listing creation/update flow
- Document upload validation
- Status transition validation (can't skip states)
- Verification submission prerequisites
- Owner profile completion checks

### Test Types:
- Unit tests for service methods
- Integration tests for view flows
- Admin action tests for reviewer workflows

## Error Handling

- Use Django messages framework for user-facing errors
- Return structured error dictionaries for API/JSON responses
- Log errors to console (Django logging) - no external logging service needed
- Always validate before state transitions
- Show helpful error messages (e.g., "Please upload at least one photo")

## Security (MVP Basics)

- Django's CSRF protection (always use `{% csrf_token %}`)
- Django's XSS protection (automatic in templates)
- `@login_required` on all authenticated views
- File upload validation (type, size)
- SQL injection protection (Django ORM handles this)

## Performance Checklist

Before committing code, verify:
- [ ] No N+1 queries (use prefetch/select_related)
- [ ] Critical fields have `db_index=True`
- [ ] Composite indexes on filtered fields
- [ ] Querysets are filtered efficiently
- [ ] File uploads validate before save
- [ ] Views complete in reasonable time (<500ms for simple operations)

## Documentation Standards

- Every model must have docstring explaining its purpose
- Complex business logic must have inline comments
- Service methods must document parameters, return values, side effects
- Views should document authentication requirements
- README should have setup instructions (even for SQLite)

## Code Review Checklist

When reviewing or writing code:
1. Does it use Django's built-in features first?
2. Are database queries optimized (O(1) or O(log n))?
3. Are there type hints and docstrings?
4. Is error handling clear and helpful?
5. Are state transitions explicit and auditable?
6. Does it follow the verification flow requirements?
7. Is it testable?

## Reminders

- We're building an MVP - focus on core features, not perfection
- Django does 95% of the work - leverage it
- External services are last resort, not first choice
- Code quality matters from day one, even locally
- Performance matters - O(1) mindset always
- Verification must be strict but UX must be smooth

## When in Doubt

1. Check if Django has a built-in solution
2. Check plan.md for architecture decisions
3. Keep it simple, testable, and maintainable
4. Follow the O(1) performance principle
5. Write code that would pass a senior Django developer's review

---
> Source: [Talktodre-ops/django-for-property](https://github.com/Talktodre-ops/django-for-property) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
