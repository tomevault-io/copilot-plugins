## adlab

> - **ALWAYS place imports at the top of the file** (module level)

# Cursor Rules - Django/Python Best Practices
# AdLab Laboratory Management System

## 🐍 Python General Best Practices

### Imports
- **ALWAYS place imports at the top of the file** (module level)
- **NEVER use local imports inside functions** unless absolutely necessary for:
  - Avoiding circular imports (rare, document why)
  - Optional dependencies
  - Performance-critical edge cases
- **Group imports** in this order (PEP 8):
  1. Standard library imports
  2. Related third-party imports (Django, etc.)
  3. Local application imports
- **Sort imports alphabetically** within each group
- **Use absolute imports** over relative imports when possible

### Code Style (PEP 8)
- Use **4 spaces** for indentation (never tabs)
- **Maximum line length**: 88 characters (Black formatter standard)
- **Two blank lines** before top-level classes and functions
- **One blank line** between methods in a class
- Use **snake_case** for functions and variables
- Use **PascalCase** for classes
- Use **UPPER_CASE** for constants
- Add **trailing commas** in multi-line collections

### Naming Conventions
- Use **descriptive names**: `calculate_total_price()` not `calc()`
- **Avoid single-letter variables** except for: i, j, k (loops), e (exceptions), f (files)
- **Boolean variables**: prefix with `is_`, `has_`, `can_`, `should_`
- **Private methods**: prefix with single underscore `_internal_method()`

### Documentation
- **Every module** should have a module-level docstring
- **Every function/method** should have a docstring explaining:
  - What it does
  - Parameters (with types)
  - Return value (with type)
  - Exceptions raised
- Use **Google-style** or **NumPy-style** docstrings
- Keep docstrings **up-to-date** with code changes

---

## 🎯 Django Best Practices

### Models
- **Always use verbose_name** and **verbose_name_plural** for models
- **Add help_text** to fields that need clarification
- **Use choices** for fields with limited options (TextChoices, IntegerChoices)
- **Override save()** with super().save() calls, never skip the super call
- **Add __str__()** method to every model (for admin and debugging)
- **Use related_name** in ForeignKey/ManyToMany relationships
- **Add indexes** for fields used in filters/queries
- **Use db_index=True** for fields frequently used in WHERE clauses
- **Avoid nullable CharField/TextField** - use blank=True with default="" instead

### Views
- **Use class-based views** for common patterns (ListView, DetailView, etc.)
- **Keep views thin** - business logic belongs in models or services
- **Always use @login_required** or LoginRequiredMixin for protected views
- **Use get_object_or_404** instead of try/except for single object retrieval
- **Add docstrings** to views explaining their purpose
- **Use proper HTTP status codes** (200, 201, 400, 404, etc.)

### Forms
- **Always use Django forms** - never process raw POST data
- **Use ModelForm** when working with models
- **Add clean_*() methods** for field-level validation
- **Add clean()** for cross-field validation
- **Customize widgets** for better UX (add CSS classes)
- **Use help_text** for user guidance

### Templates
- **Always use {% csrf_token %}** in forms
- **Escape user input** by default (Django does this, don't use |safe unless needed)
- **Use {% static %}** tag for static files
- **Use {% url %}** tag for URLs (never hardcode URLs)
- **Keep logic out of templates** - move to views or template tags

### Admin
- **Register all models** with admin.site.register() or @admin.register()
- **Customize list_display** for better overview
- **Add list_filter** for common filters
- **Add search_fields** for searchable fields
- **Use readonly_fields** for fields that shouldn't be edited
- **Add actions** for bulk operations
- **Customize fieldsets** for better organization

### URLs
- **Use path()** over re_path() when possible
- **Always name your URLs** with name= parameter
- **Use app namespaces** with app_name
- **Group related URLs** using include()
- **Keep URL patterns RESTful** when appropriate

### Settings
- **NEVER commit secrets** to version control
- **Use environment variables** for secrets (.env file)
- **Separate settings** for dev/staging/production (if needed)
- **Use django-environ** or python-decouple for env management
- **Document all custom settings** with comments

### Security
- **Always use HTTPS** in production (set SECURE_SSL_REDIRECT=True)
- **Use CSRF protection** (enabled by default, keep it)
- **Validate and sanitize** all user input
- **Use Django's password hashers** (never store plain passwords)
- **Set secure cookie flags** (HTTPONLY, SECURE, SAMESITE)
- **Use secrets module** for token generation (not random)
- **Add audit logging** for sensitive operations
- **Implement rate limiting** for authentication endpoints
- **Use permissions and groups** for access control
- **Keep dependencies updated** (security patches)

---

## 🎨 Clean Code Practices for Views & Complex Logic

**CRITICAL**: Apply these patterns from the start. Write clean code, not code that needs refactoring.

### Early Returns (Guard Clauses) ✅

**ALWAYS use early returns to avoid deep nesting.**

**Bad Example** ❌:
```python
def login_view(request):
    if request.user.is_authenticated:
        return redirect('/')
    if request.method == "POST":
        form = LoginForm(request.POST)
        if form.is_valid():
            user = form.get_user()
            if user.is_active:
                if user.email_verified:
                    # Success logic (4 levels deep!)
                    login(request, user)
                    return redirect('/')
                else:
                    messages.error(request, "Email not verified")
            else:
                messages.error(request, "Account inactive")
    else:
        form = LoginForm()
    return render(request, 'login.html', {'form': form})
```

**Good Example** ✅:
```python
def login_view(request):
    # Early return: already authenticated
    if request.user.is_authenticated:
        return redirect('/')
    
    # Early return: GET request
    if request.method != "POST":
        return render(request, 'login.html', {'form': LoginForm()})
    
    # POST: process form
    form = LoginForm(request.POST)
    
    # Early return: form invalid
    if not form.is_valid():
        return render(request, 'login.html', {'form': form})
    
    user = form.get_user()
    
    # Early return: account inactive
    if not user.is_active:
        messages.error(request, "Account inactive")
        return render(request, 'login.html', {'form': form})
    
    # Early return: email not verified
    if not user.email_verified:
        messages.error(request, "Email not verified")
        return render(request, 'login.html', {'form': form})
    
    # Success path (zero nesting!)
    login(request, user)
    return redirect('/')
```

**Rules:**
- Check error conditions FIRST
- Return IMMEDIATELY if condition fails
- Happy path stays at the END with ZERO nesting
- Each check is ONE level of indentation max
- Result: Flat, readable, maintainable code

### Extract Method (DRY Principle) ✅

**NEVER copy-paste code. Extract to helper functions.**

**Bad Example** ❌:
```python
def register_view(request):
    # ... form processing ...
    
    # Send email (30 lines)
    token = secrets.token_urlsafe(32)
    user.verification_token = token
    user.save()
    
    html_message = render_to_string(
        'emails/verification.html',
        {'user': user, 'token': token}
    )
    plain_message = strip_tags(html_message)
    send_mail(
        subject="Verify Email",
        message=plain_message,
        from_email=settings.DEFAULT_FROM_EMAIL,
        recipient_list=[user.email],
        html_message=html_message,
    )

def resend_verification_view(request):
    # ... same 30 lines copied again! ❌
    
def admin_resend_action(request):
    # ... same 30 lines copied AGAIN! ❌
```

**Good Example** ✅:
```python
def send_verification_email(request, user):
    """
    Send email verification to user.
    
    Args:
        request: HTTP request (for building absolute URI)
        user: User object to send verification to
    
    Returns:
        bool: True if sent successfully
    """
    token = user.generate_verification_token()
    verification_url = request.build_absolute_uri(
        f"/accounts/verify-email/{token}/"
    )
    
    html_message = render_to_string(
        'emails/verification.html',
        {'user': user, 'verification_url': verification_url}
    )
    
    try:
        send_mail(
            subject="Verify Email",
            message=strip_tags(html_message),
            from_email=settings.DEFAULT_FROM_EMAIL,
            recipient_list=[user.email],
            html_message=html_message,
            fail_silently=False,
        )
        return True
    except Exception:
        return False

# Now use it everywhere:
def register_view(request):
    # ... form processing ...
    send_verification_email(request, user)

def resend_verification_view(request):
    # ... 
    send_verification_email(request, user)

def admin_resend_action(request):
    # ...
    send_verification_email(request, user)
```

**Rules:**
- If code appears MORE THAN ONCE, extract it
- Helper functions should do ONE thing
- Name helpers clearly: `send_verification_email` not `send_email`
- Place helpers at MODULE level (before views)
- Prefix with `_` if internal: `_log_failed_login()`

### Consistent View Structure ✅

**ALL views should follow the SAME pattern.**

**Standard View Structure:**
```python
def view_name(request, *args, **kwargs):
    """Brief description of what this view does."""
    
    # 1. GUARD CLAUSES (early returns for edge cases)
    if edge_case:
        return redirect_or_error()
    
    # 2. GET REQUEST: Show form
    if request.method != "POST":
        return render(request, 'template.html', {'form': Form()})
    
    # 3. POST REQUEST: Process form
    form = Form(request.POST)
    
    # 4. VALIDATION: Early return if invalid
    if not form.is_valid():
        return render(request, 'template.html', {'form': form})
    
    # 5. PROCESS: Do the work
    result = process_form(form)
    
    # 6. SUCCESS: Show message and redirect
    messages.success(request, "Success message")
    return redirect('success_url')
```

**Benefits:**
- Predictable code structure
- Easy to understand ANY view
- Faster code reviews
- Easier onboarding

### Helper Functions for Complex Logic ✅

**Break complex views into focused helper functions.**

**Bad Example** ❌:
```python
def login_view(request):
    # 150+ lines of code
    # Handles: auth, lockout, logging, session, redirect
    # Too many responsibilities!
```

**Good Example** ✅:
```python
def login_view(request):
    """Handle user login - orchestrates the flow."""
    if request.user.is_authenticated:
        return redirect('/')
    
    if request.method != "POST":
        return render(request, 'login.html', {'form': LoginForm()})
    
    form = LoginForm(request, data=request.POST)
    user = User.objects.filter(email=request.POST.get('username')).first()
    
    if user and user.is_locked_out():
        _handle_locked_account(request, user)
        return render(request, 'login.html', {'form': form})
    
    if not form.is_valid():
        _handle_failed_login(request, user)
        return render(request, 'login.html', {'form': form})
    
    return _handle_successful_login(request, form.get_user(), form)


def _handle_locked_account(request, user):
    """Handle login attempt on locked account."""
    _log_failed_login(request, user.email, user, "Account locked")
    messages.error(request, "Account is locked. Contact admin.")


def _handle_failed_login(request, user):
    """Handle failed login attempt with lockout logic."""
    if user:
        user.increment_failed_attempts()
        remaining = 5 - user.failed_login_attempts
        
        if user.is_locked_out():
            AuthAuditLog.log(
                action=AuthAuditLog.Action.ACCOUNT_LOCKED,
                email=user.email,
                user=user,
                ip_address=get_client_ip(request),
                user_agent=get_user_agent(request),
            )
            messages.error(request, "Account locked after 5 attempts.")
        elif remaining > 0:
            messages.error(request, f"Wrong password. {remaining} attempts left.")
    else:
        messages.error(request, "Invalid credentials.")


def _handle_successful_login(request, user, form):
    """Handle successful login."""
    user.reset_failed_attempts()
    user.last_login_at = timezone.now()
    user.save()
    
    login(request, user)
    
    if not form.cleaned_data.get('remember_me'):
        request.session.set_expiry(0)
    
    messages.success(request, f"Welcome, {user.get_full_name()}!")
    return redirect(request.GET.get('next', '/'))
```

**Rules:**
- Main view = ORCHESTRATOR (calls helpers)
- Helpers = WORKERS (do specific tasks)
- Each helper has ONE responsibility
- Prefix internal helpers with `_`
- Keep helpers focused (< 20 lines)

### Avoid Deep Nesting ✅

**Maximum 2 levels of indentation in views.**

**Bad Example** ❌:
```python
def view(request):
    if request.method == "POST":                    # Level 1
        if form.is_valid():                         # Level 2
            if user.is_active:                      # Level 3
                if user.has_permission:             # Level 4
                    if not user.is_locked:          # Level 5
                        # Success (5 levels deep!)
```

**Good Example** ✅:
```python
def view(request):
    if request.method != "POST":                    # Level 1
        return render(...)                          # Early return
    
    if not form.is_valid():                         # Level 1
        return render(...)                          # Early return
    
    if not user.is_active:                          # Level 1
        return error_response(...)                  # Early return
    
    if not user.has_permission:                     # Level 1
        return error_response(...)                  # Early return
    
    if user.is_locked:                              # Level 1
        return error_response(...)                  # Early return
    
    # Success (1 level only!)
    return success_response(...)
```

**Rules:**
- Maximum 2 levels of indentation
- Use early returns for validation
- Extract nested logic to helpers
- One condition per if statement

### Better Error Handling ✅

**Use .filter().first() instead of try/except for queries.**

**Bad Example** ❌:
```python
try:
    user = User.objects.get(email=email)
    if user.is_active:
        # nested logic
except User.DoesNotExist:
    user = None
    # more logic
```

**Good Example** ✅:
```python
user = User.objects.filter(email=email).first()

if not user:
    return error_response("User not found")

if not user.is_active:
    return error_response("User inactive")

# Success path
```

**Rules:**
- Use `.filter().first()` for single objects (returns None)
- Use `get_object_or_404()` in views (raises 404)
- Reserve try/except for actual exceptions
- Specific exceptions only (never bare `except:`)

### Meaningful Function Names ✅

**Function names should explain WHAT and WHY.**

**Bad Names** ❌:
- `process()` - Process what?
- `handle()` - Handle what?
- `do_stuff()` - What stuff?
- `func1()` - Meaningless
- `temp()` - Too generic

**Good Names** ✅:
- `send_verification_email()` - Clear purpose
- `_handle_failed_login()` - Explains what it handles
- `calculate_total_with_tax()` - Says what it calculates
- `is_user_allowed_to_edit()` - Boolean, clear intent
- `get_active_users_from_department()` - Describes what it gets

**Rules:**
- Use verbs for actions: `send`, `calculate`, `validate`
- Use `is_`, `has_`, `can_` for booleans
- Be specific: `send_verification_email` not `send_email`
- Length is OK if it adds clarity
- Avoid abbreviations unless obvious

### Single Responsibility Principle (SRP) ✅

**Each function should do ONE thing.**

**Bad Example** ❌:
```python
def process_order(order):
    # 1. Validate order
    # 2. Calculate total
    # 3. Process payment
    # 4. Send email
    # 5. Update inventory
    # 6. Create invoice
    # Too many responsibilities!
```

**Good Example** ✅:
```python
def process_order(order):
    """Orchestrate order processing."""
    validate_order(order)
    total = calculate_order_total(order)
    payment = process_payment(order, total)
    send_order_confirmation(order)
    update_inventory(order)
    create_invoice(order, payment)

# Each function does ONE thing:
def validate_order(order):
    """Validate order has all required fields."""
    ...

def calculate_order_total(order):
    """Calculate order total with tax and shipping."""
    ...

def process_payment(order, total):
    """Process payment for order."""
    ...
```

**Rules:**
- Function should do ONE thing well
- If you use "and" describing it, split it
- Functions should be short (< 30 lines ideal)
- Each function one level of abstraction

### Cyclomatic Complexity ✅

**Keep functions simple and testable.**

**Complexity Guidelines:**
- **CC < 5**: Low complexity (Easy to test) ✅
- **CC 5-10**: Medium complexity (Acceptable) ⚠️
- **CC > 10**: High complexity (Refactor!) ❌

**High Complexity Example** ❌:
```python
def complex_view(request):
    if condition1:
        if condition2:
            if condition3:
                if condition4:
                    if condition5:
                        # CC = 15+
```

**Low Complexity Example** ✅:
```python
def simple_view(request):
    # Early returns = lower complexity
    if not condition1:
        return error1()
    
    if not condition2:
        return error2()
    
    if not condition3:
        return error3()
    
    # CC = 3 (much better!)
    return success()
```

**How to Reduce Complexity:**
1. Use early returns (guard clauses)
2. Extract nested logic to helpers
3. Replace nested ifs with separate checks
4. Use helper functions
5. Simplify boolean expressions

### View Patterns Summary ✅

**Template for Every View:**

```python
# Module-level helper functions (if needed)
def _helper_function(args):
    """Helper does ONE specific thing."""
    pass


# Main view function
@decorator_if_needed
def view_name(request, *args, **kwargs):
    """
    Brief description of view purpose.
    
    Args:
        request: HTTP request
        *args, **kwargs: URL parameters
    
    Returns:
        HttpResponse: Rendered template or redirect
    """
    # 1. Guard clauses (early returns)
    if edge_case:
        return handle_edge_case()
    
    # 2. GET: show form
    if request.method != "POST":
        return render(request, 'template.html', {
            'form': Form()
        })
    
    # 3. POST: process
    form = Form(request.POST)
    
    # 4. Validate (early return)
    if not form.is_valid():
        return render(request, 'template.html', {
            'form': form
        })
    
    # 5. Process (use helpers)
    result = _process_form_data(form)
    
    # 6. Log if needed
    _log_action(request, result)
    
    # 7. Success message
    messages.success(request, "Operation successful")
    
    # 8. Redirect
    return redirect('success_url')
```

### Before/After Comparison

**Before Refactoring** ❌:
- 531 lines
- 4 levels of nesting
- Cyclomatic Complexity: 15
- Duplicated code in 3 places
- Hard to read and test

**After Clean Code** ✅:
- 493 lines (-7%)
- 1 level of nesting max
- Cyclomatic Complexity: 8
- DRY with helper functions
- Easy to read and test

**Result: 37/37 tests passing with cleaner code!**

---

## 🧪 Testing Best Practices

### Test Organization
- **Use TestCase classes** for database tests
- **Use SimpleTestCase** for non-database tests (faster)
- **One test file per app** (tests.py or tests/ directory)
- **Group related tests** in test classes
- **Name tests descriptively**: `test_user_cannot_login_without_email_verification`

### Test Coverage
- **Aim for >80% coverage** on business logic
- **Test happy paths** (normal flow)
- **Test edge cases** (empty inputs, max values, etc.)
- **Test error conditions** (invalid data, permissions, etc.)
- **Test authentication/authorization** flows
- **Mock external services** (email, APIs, etc.)

### Test Practices
- **Use setUp()** for common test data
- **Use factories** for complex object creation (factory_boy)
- **Use fixtures sparingly** (prefer factories)
- **Test one thing per test** (focused tests)
- **Use assertRaises** for exception testing
- **Use @patch** for mocking (unittest.mock)
- **Clean up in tearDown()** if needed

---

## 📊 Database Best Practices

### Queries
- **Use select_related()** for ForeignKey lookups (single query)
- **Use prefetch_related()** for ManyToMany/reverse FK (two queries)
- **Avoid N+1 queries** - always check query count
- **Use only()** to fetch specific fields
- **Use defer()** to exclude large fields
- **Use exists()** instead of count() for existence checks
- **Use F() expressions** for database-level operations
- **Use Q() objects** for complex queries

### Migrations
- **Always create migrations** after model changes
- **Review migrations** before applying (makemigrations output)
- **Never edit applied migrations** (create new ones)
- **Use data migrations** for complex data changes
- **Make migrations reversible** when possible
- **Test migrations** on copy of production data
- **Keep migrations small** and focused

### Performance
- **Add database indexes** for frequently queried fields
- **Use database constraints** (unique, check, etc.)
- **Denormalize carefully** only when necessary
- **Use Django's caching framework** for expensive queries
- **Monitor slow queries** (django-debug-toolbar in dev)
- **Use pagination** for large result sets

---

## 🔒 Security Checklist

### Authentication & Authorization
- ✅ Use Django's authentication system (don't roll your own)
- ✅ Implement password complexity requirements
- ✅ Add account lockout after failed attempts
- ✅ Use email verification for external users
- ✅ Implement password reset with tokens
- ✅ Add audit logging for authentication events
- ✅ Use permissions for access control
- ✅ Check permissions in views (not just templates)

### Data Protection
- ✅ Validate all user input
- ✅ Sanitize data before display
- ✅ Use parameterized queries (Django ORM does this)
- ✅ Don't expose sensitive data in URLs
- ✅ Don't log sensitive data (passwords, tokens)
- ✅ Encrypt sensitive data at rest (if required)
- ✅ Use HTTPS for all traffic (production)

### API Security (if applicable)
- ✅ Use authentication for all endpoints
- ✅ Validate Content-Type headers
- ✅ Implement rate limiting
- ✅ Use CORS properly (whitelist domains)
- ✅ Version your APIs
- ✅ Document security requirements

---

## 🎨 Frontend Best Practices (HTMX + Alpine.js)

### HTMX
- **Use hx-target** to specify where response goes
- **Use hx-swap** to control how content is swapped
- **Add hx-indicator** for loading states
- **Use hx-trigger** for custom events
- **Add proper error handling** with hx-on::after-request

### Alpine.js
- **Keep Alpine logic simple** (complex logic in backend)
- **Use x-data** for component state
- **Use x-show/x-if** for conditional rendering
- **Use x-bind** for dynamic attributes
- **Use x-on** for event handling

### Tailwind CSS
- **Use utility classes** for styling
- **Create component classes** for repeated patterns
- **Use @apply** in custom CSS (sparingly)
- **Follow mobile-first** approach (sm:, md:, lg:)
- **Use consistent spacing** (4, 8, 16, 24, etc.)

---

## 📝 Git Best Practices

### Commits

**IMPORTANT: Follow strict commit message format**

#### Commit Message Format
```
<type>[step-XX]: <brief summary (max 50 chars)>

[Optional body with details, max 72 chars per line]
[Explain WHY, not WHAT - the diff shows what changed]

[Optional footer with breaking changes, issues, etc.]
```

#### Commit Types (Required)
- **feat**: New feature or functionality
- **fix**: Bug fix
- **docs**: Documentation only changes
- **style**: Code style/formatting (no logic change)
- **refactor**: Code refactoring (no behavior change)
- **test**: Adding or updating tests
- **chore**: Maintenance, deps, config, etc.
- **perf**: Performance improvements
- **security**: Security-related changes

#### Step Tags (Required for feature work)
- **[step-01]**: Authentication & User Management
- **[step-01.1]**: Email Verification
- **[step-02]**: Veterinarian Profiles
- **[step-03]**: Protocol Submission
- **[step-04]**: Sample Reception
- **[step-05]**: Sample Processing
- **[step-06]**: Report Generation
- **[step-07]**: Work Orders
- **[step-08]**: Email Notifications
- **[step-09]**: Dashboard
- **[step-10]**: Reports & Analytics
- **[step-11]**: Data Migration
- **[step-12]**: System Administration
- **[step-13]**: Email Configuration

#### Examples

**Good Examples** ✅:
```
feat[step-01.1]: Add email verification for veterinarians

Implement secure token-based email verification:
- Cryptographically secure tokens (24h expiration)
- Separate flow for external vs internal users
- Resend functionality with rate limiting
- Comprehensive audit logging

Closes #42
```

```
fix[step-01]: Prevent login with unverified email

Block veterinarians from logging in before email verification.
Internal users (lab staff, admin) bypass this check.
```

```
refactor[step-01]: Move imports to module level

Moved all imports from function level to module level
following PEP 8 guidelines for better performance and
maintainability.
```

```
test[step-01.1]: Add 17 tests for email verification

Coverage includes:
- Token generation and expiration
- Verification flows (success, expired, invalid)
- Resend functionality
- Admin actions
```

```
docs[step-13]: Add email configuration guide

Created comprehensive guide for production email setup
including SMTP options, costs, and security practices.
```

```
chore: Update dependencies and fix security warnings

Updated Django to 5.2.1 for security patches.
Updated all dependencies to latest stable versions.
```

**Bad Examples** ❌:
```
Update code  ❌ Too vague, no type, no step
```

```
feat: implemented email verification and also fixed some bugs and updated docs  ❌ Too long (>50 chars), multiple changes
```

```
WIP  ❌ Not descriptive, no type
```

```
fixed stuff  ❌ Not professional, no type, no step
```

#### Summary Line Rules
- **Max 50 characters** for summary line
- **Use imperative mood**: "Add feature" not "Added feature"
- **No period at end** of summary line
- **Capitalize first letter** after the type
- **Be specific**: "Fix login bug" not "Fix bug"

#### Body Rules (Optional but Recommended)
- **Wrap at 72 characters** per line
- **Explain WHY**, not what (diff shows what)
- **Use bullet points** for multiple items
- **Reference issues**: "Closes #123", "Fixes #456"
- **Breaking changes**: Start line with "BREAKING CHANGE:"

#### When to Include Step Tags
- ✅ **Use step tags** for: feature work, fixes, tests related to a specific step
- ❌ **Omit step tags** for: general refactoring, documentation, chore tasks

#### Commit Frequency
- **Commit often** (small, atomic commits)
- **One logical change** per commit
- **Working code** at each commit (tests should pass)
- **Don't mix** refactoring with feature work

### Branches
- **Use feature branches** for development
- **Name branches descriptively**: `feature/email-verification`
- **Keep main/master stable** (always deployable)
- **Delete merged branches** to keep repo clean

### Pull Requests
- **Write descriptive PR titles**
- **Add PR description** explaining changes
- **Link to issues** if applicable
- **Request reviews** from team members
- **Address review comments** before merging

---

## 🚀 Deployment Best Practices

### Pre-deployment Checklist
- ✅ All tests passing
- ✅ Migrations tested
- ✅ Static files collected
- ✅ Environment variables configured
- ✅ Database backed up
- ✅ Monitoring in place
- ✅ Rollback plan ready

### Environment Configuration
- ✅ DEBUG=False in production
- ✅ ALLOWED_HOSTS configured
- ✅ SECRET_KEY is secret (not in code)
- ✅ Database credentials secure
- ✅ Static files served by CDN/nginx
- ✅ Email configured (SMTP)
- ✅ Logging configured

---

## 🐛 Debugging Best Practices

### Development
- **Use django-debug-toolbar** for query inspection
- **Use logging** (not print statements)
- **Use breakpoints** (debugger) over print debugging
- **Use Django shell** for quick testing
- **Use manage.py shell_plus** (django-extensions)

### Production
- **Never use DEBUG=True** in production
- **Use proper logging** (file or service)
- **Monitor error rates** (Sentry, etc.)
- **Keep logs searchable** (structured logging)
- **Don't log sensitive data**

---

## 📚 Documentation Best Practices

### Code Documentation
- **Document WHY, not WHAT** - code shows what, docs explain why
- **Keep README updated** with setup instructions
- **Document complex algorithms** with comments
- **Add docstrings** to all public functions/classes
- **Use type hints** for function parameters and returns

### Project Documentation
- **README.md**: Setup, requirements, quick start
- **CONTRIBUTING.md**: How to contribute
- **CHANGELOG.md**: What changed in each version
- **API documentation**: If building APIs
- **Architecture docs**: High-level system design

---

## ⚡ Performance Best Practices

### Django Performance
- **Use select_related/prefetch_related** to reduce queries
- **Cache expensive operations** (template fragments, views, querysets)
- **Use pagination** for large lists
- **Optimize database queries** (avoid N+1)
- **Use indexes** on frequently queried fields
- **Minimize middleware** (only use what's needed)

### Frontend Performance
- **Minimize HTTP requests** (bundle assets)
- **Compress static files** (gzip/brotli)
- **Use CDN** for static files
- **Lazy load images** (loading="lazy")
- **Minimize JavaScript** (only load what's needed)

---

## 🔧 Development Workflow

### Before Starting Work
1. Pull latest changes from main
2. Create feature branch
3. Install/update dependencies
4. Run migrations
5. Run tests to ensure baseline

### During Development
1. Write tests first (TDD) or alongside code
2. Commit frequently
3. Run tests before committing
4. Review your own changes (git diff)
5. Keep changes focused and atomic

### Before Committing
1. Run all tests: `python manage.py test`
2. Check code style: `flake8` or `black --check`
3. Review changes: `git diff`
4. Write clear commit message
5. Push to remote regularly

### Before Merging
1. All tests passing
2. Code reviewed
3. Documentation updated
4. No merge conflicts
5. Changelog updated (if applicable)

---

## 🎯 Code Review Checklist

### Functionality
- ✅ Does it work as intended?
- ✅ Edge cases handled?
- ✅ Error cases handled?
- ✅ Tests covering new code?

### Code Quality
- ✅ Follows Django/Python conventions?
- ✅ No code duplication (DRY)?
- ✅ Functions/methods are focused (SRP)?
- ✅ Clear variable/function names?
- ✅ No magic numbers/strings?

### Security
- ✅ User input validated?
- ✅ No SQL injection risks?
- ✅ No XSS vulnerabilities?
- ✅ Permissions checked?
- ✅ Sensitive data protected?

### Performance
- ✅ No N+1 queries?
- ✅ Proper indexing?
- ✅ Caching where needed?
- ✅ Pagination for large lists?

### Documentation
- ✅ Code commented where needed?
- ✅ Docstrings present?
- ✅ README updated?
- ✅ CHANGELOG updated?

---

## 🚨 Common Anti-Patterns to Avoid

### Django Anti-Patterns
- ❌ Business logic in views (move to models/services)
- ❌ Fat models (break into services if too large)
- ❌ N+1 queries (use select_related/prefetch_related)
- ❌ Hardcoded URLs (use {% url %} and reverse())
- ❌ Mutable default arguments (use None, then assign in function)
- ❌ Circular imports (restructure code)
- ❌ Not using Django forms (validation, security)

### Python Anti-Patterns
- ❌ Bare except: clauses (be specific: except ValueError:)
- ❌ Using mutable defaults (def func(x=[]):)
- ❌ Not using context managers (with open(...):)
- ❌ Importing * (from module import *)
- ❌ Using eval() or exec() (major security risk)
- ❌ Concatenating strings in loops (use join())

---

## 📋 Project-Specific Rules (AdLab)

### Language and Translation
- **Model names**: ALWAYS use English (e.g., `Protocol`, `CytologySample`, `Veterinarian`)
- **Field names**: ALWAYS use English (e.g., `submission_date`, `animal_identification`)
- **Translations (verbose_name, help_text, labels)**: ALWAYS use Spanish for user-facing text
- **HTML templates**: ALWAYS use Spanish for all user-facing text
- **Error messages**: ALWAYS use Spanish
- **Success messages**: ALWAYS use Spanish
- **Admin interface**: Spanish translations via verbose_name
- **Code comments**: Can be in English (internal documentation)

**Example**:
```python
# CORRECT ✅
class Protocol(models.Model):  # English model name
    submission_date = models.DateField(  # English field name
        verbose_name="Fecha de remisión",  # Spanish translation
        help_text="Fecha en que se envía la muestra"  # Spanish help text
    )

# INCORRECT ❌
class Protocolo(models.Model):  # Spanish model name
    fecha_remision = models.DateField(  # Spanish field name
        verbose_name="Submission Date",  # English translation
    )
```

### Authentication
- External users (veterinarians) MUST verify email
- Internal users (lab staff, admins) bypass email verification
- All auth events MUST be logged in AuthAuditLog
- Account lockout after 5 failed attempts

### Email
- Use console backend in development
- Configure SMTP before production (Step 13)
- All emails must have HTML and plain text versions
- Token expiration: 24 hours for verification, 1 hour for password reset
- Email content MUST be in Spanish

### Models
- All models MUST have __str__() method
- Use Role-based access control (RBAC)
- Add audit fields where needed (created_at, updated_at, created_by)
- All verbose_name and help_text MUST be in Spanish

### Testing
- Aim for >80% test coverage
- Mock email sending in tests
- Test both authenticated and unauthenticated flows
- Test all user roles (veterinarian, lab staff, admin)

### Deployment
- Always backup database before migrations
- Test migrations on staging first
- Keep Step 13 (Email Config) for production setup
- Monitor email delivery rates

### Server Changes Workflow
- **NEVER make changes directly on the server** that aren't in the codebase
- **ALWAYS follow this workflow**:
  1. Make changes locally
  2. Test changes locally
  3. Commit changes with proper commit message
  4. Push to remote repository
  5. Pull changes on server: `git pull`
  6. Rebuild containers if needed: `docker compose build && docker compose up -d`
- **This ensures**:
  - All changes are version controlled
  - Changes can be reviewed and rolled back
  - Server and local codebase stay in sync
  - No "works on my machine" issues

---

**Remember**: These rules exist to maintain code quality, security, and consistency. 
When in doubt, follow Django's official documentation and "The Zen of Python" (import this).

**Last Updated**: October 2025
**Next Review**: When adding new patterns or technologies

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fmoreyra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
