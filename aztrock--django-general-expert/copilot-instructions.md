## django-general-expert

> Expert Django guidance covering clean code, DRF, testing, security, and performance. Focus on code organization, early return patterns, avoiding bool trap, decoupling business logic from views, and production-ready patterns. For Django 4.2+, DRF, and TestCase. Use when building Django APIs, optimizing queries, implementing authentication, writing tests, or refactoring Django code.


# Django General Expert

Expert guidance for Django development with emphasis on clean code, maintainable architecture, and production-ready patterns.

## Objective

Deliver production-ready code that maintains consistency with the project's architecture, meets quality expectations, integrates with the team's workflow, and adheres to security, testing, and documentation standards so that code reviews are efficient and the codebase remains maintainable and scalable.

## Overview

This skill provides comprehensive Django guidance for advanced developers, focusing on:

- **Clean Code Patterns**: Early return, avoiding bool trap, meaningful naming
- **Architecture**: Service layer, decoupled business logic, modular design
- **Django REST Framework**: Serializers, viewsets, permissions, performance
- **Testing**: TestCase-based testing strategies (no pytest)
- **Performance**: Query optimization, caching, database patterns
- **Security**: OWASP Top 10, authentication, authorization

**Target Audience**: Advanced Django developers working on production applications.

**Django Version**: 4.2 LTS and later

## When to Use

Invoke this skill when:

**Code Quality & Organization:**
- "Refactor this view to be cleaner"
- "This code has too many nested ifs"
- "How to organize this Django app?"
- "Avoid bool trap in this code"
- "Make this more readable"

**Models & ORM:**
- "Design models for..."
- "Fix N+1 query problem"
- "Optimize this queryset"
- "Create migrations for..."

**Views & APIs:**
- "Create an API endpoint for..."
- "Build a view that handles..."
- "Decouple logic from this view"
- "Implement permissions for..."

**Testing:**
- "Write tests for this feature"
- "Test this DRF endpoint"
- "Mock external service in tests"

**Security:**
- "Secure this endpoint"
- "Implement authentication"
- "Review for security issues"

**Performance:**
- "This view is slow"
- "Optimize database queries"
- "Add caching to..."

## Instructions

### Step 1: Classify the Request

Identify the task category, then **use the Read tool** to load the corresponding reference file:

| Task Type | Action |
|-----------|--------|
| Clean code, organization, early return | Read `references/code-organization.md` |
| Code quality standards | Read `references/code-quality.md` |
| Models, ORM | Read `references/models-and-orm.md` |
| Views, URLs | Read `references/views-and-urls.md` |
| DRF (serializers, viewsets, errors) | Read `references/drf-guidelines.md` |
| Filters (django-filters) | Read `references/filters.md` |
| Testing | Read `references/testing-strategies.md` |
| Security | Read `references/security-checklist.md` |
| Performance | Read `references/performance-optimization.md` |
| Advanced patterns (services, repositories) | Read `references/advanced-patterns.md` |
| Constants and choices | Read `references/constants.md` |
| Celery tasks | Read `references/celery-patterns.md` |
| Django signals | Read `references/signals-guide.md` |
| Django Admin | Read `references/admin-guide.md` |
| Forms & ModelForms | Read `references/forms-modelforms.md` |
| Authentication | Read `references/authentication.md` |
| Middleware | Read `references/middleware.md` |
| Database migrations | Read `references/migrations.md` |
| Logging | Read `references/logging.md` |
| Static & media files | Read `references/static-media-files.md` |
| Project structure | Read `references/project-structure.md` |
| Third-party packages | Read `references/third-party-packages.md` |

For multi-category requests, read all relevant references.

### Step 2: Read Reference File(s)

**CRITICAL**: Always use the Read tool to load the relevant reference file(s) before implementing.

Example:
```
Read references/code-organization.md
```

Do NOT proceed without reading the appropriate documentation.

### Step 3: Implement

Apply patterns from references. Before presenting solutions, verify:

**Code Quality Checklist:**
- [ ] Uses early return pattern (no nested ifs)
- [ ] Avoids bool trap (uses enums/choices)
- [ ] Business logic in services, NOT in views
- [ ] Meaningful names for variables/functions
- [ ] No docstrings that just repeat function names
- [ ] Constants in separate file (not hardcoded)
- [ ] No imports inside functions/classes
- [ ] Follows Boy Scout Rule, DRY, KISS, YAGNI

**Django/DRF Checklist:**
- [ ] No N+1 queries (select_related/prefetch_related)
- [ ] Serializers explicitly list fields (no `__all__`)
- [ ] Permissions in settings, NOT in every view
- [ ] Status choices in constants.py
- [ ] UUID for public IDs, id for internal
- [ ] Database indexes on search fields
- [ ] Filters only when needed (else in settings)

**Security Checklist:**
- [ ] User input validated/sanitized
- [ ] Always use ORM (if RawSQL needed, sanitize)
- [ ] Minimum permissions
- [ ] No secrets in code

**Testing Checklist:**
- [ ] Unit tests for services
- [ ] FactoryBoy for test data
- [ ] Mock external services (requests.get/post)
- [ ] DON'T mock the service itself

### Step 4: Validate

For significant implementations:
- Suggest or write tests
- Verify security best practices
- Check for performance issues
- Ensure no circular imports

## Standards & Conventions

### 1. Early Return Pattern

**ALWAYS prefer early return over nested conditionals.**

```python
# ❌ BAD: Nested ifs
def process_order(order):
    if order is not None:
        if order.is_paid:
            if order.items.exists():
                return process_items(order.items)
    return None

# ✅ GOOD: Early return
def process_order(order):
    if not order:
        return None
    if not order.is_paid:
        return None
    if not order.items.exists():
        return None
    
    return process_items(order.items)
```

**Reference**: `references/code-organization.md`

---

### 2. Services

**Encapsulate business logic in service classes.**

```python
# constants.py
class OrderStatus(models.TextChoices):
    PENDING = 'pending', 'Pending'
    PAID = 'paid', 'Paid'
    SHIPPED = 'shipped', 'Shipped'


# services.py
from dataclasses import dataclass
from django.db import transaction
import logging

logger = logging.getLogger(__name__)


@dataclass
class OrderData:
    customer_id: int
    items: list[dict]
    shipping_address: str


class OrderService:
    @staticmethod
    @transaction.atomic
    def create_order(data: OrderData) -> Order:
        logger.info(f'Creating order for customer {data.customer_id}')
        
        order = Order.objects.create(
            customer_id=data.customer_id,
            shipping_address=data.shipping_address,
            status=OrderStatus.PENDING
        )
        
        order_items = [
            OrderItem(order=order, **item) 
            for item in data.items
        ]
        OrderItem.objects.bulk_create(order_items)
        
        logger.info(f'Order {order.id} created successfully')
        return order

# ❌ BAD: Logic in view
class OrderViewSet(viewsets.ModelViewSet):
    def create(self, request):
        order = Order.objects.create(...)  # Logic in view!
        calculate_totals(order)  # Business logic in view!
```

**Rules:**
- Use dataclasses and typing
- Follow SOLID principles
- Separate read/write operations
- Use transactions for complex operations
- Log meaningful information
- Handle exceptions properly
- **NO imports inside functions**

**Reference**: `references/advanced-patterns.md`

---

### 3. View DRF

**Use ViewSets or APIView; avoid `@api_view`.**

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
    'EXCEPTION_HANDLER': 'drf_standardized_errors.handler.exception_handler',
}


# views.py
class OrderViewSet(viewsets.ModelViewSet):
    serializer_class = OrderSerializer
    
    def get_queryset(self):
        return Order.objects.filter(customer=self.request.user)
    
    def perform_create(self, serializer):
        order = OrderService.create_order(
            data=OrderData(**serializer.validated_data),
            customer=self.request.user
        )
        serializer.instance = order


# ❌ BAD: @api_view decorator
@api_view(['POST'])
def create_order(request):
    # Hard to test, hard to reuse
    ...


# ❌ BAD: Redundant permission_classes
class OrderViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]  # Already in settings!
    ...


# ❌ BAD: Redundant filter_backends
class OrderViewSet(viewsets.ModelViewSet):
    filter_backends = [DjangoFilterBackend]  # Already in settings!
    ...
```

**Rules:**
- Use ViewSets or APIView
- Keep views lightweight
- Orchestrate services
- Override methods like `get_queryset()`
- Use mixins for reuse
- Permissions in settings, NOT in views
- Filters in settings, NOT in views

**Reference**: `references/drf-guidelines.md`

---

### 4. Serializer

**Use `read_only_fields` and `extra_kwargs`.**

```python
# ✅ GOOD: Explicit serializer
class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model = Order
        fields = ['id', 'customer', 'items', 'total', 'status']
        read_only_fields = ['id', 'total', 'status']
        extra_kwargs = {
            'customer': {'write_only': True},
            'items': {'help_text': 'List of item IDs'}
        }


# ❌ BAD: __all__ exposes sensitive fields
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'  # Exposes password!
```

**Reference**: `references/drf-guidelines.md`

---

### 5. Models

**UUID for public IDs, id for internal use. Status choices in constants.py.**

```python
# constants.py
class OrderStatus(models.TextChoices):
    PENDING = 'pending', 'Pending'
    PAID = 'paid', 'Paid'
    SHIPPED = 'shipped', 'Shipped'


# models.py
import uuid
from .constants import OrderStatus


class Order(models.Model):
    id = models.BigAutoField(primary_key=True)  # Internal use
    public_id = models.UUIDField(
        default=uuid.uuid4,
        editable=False,
        unique=True,
        db_index=True
    )
    
    status = models.CharField(
        max_length=20,
        choices=OrderStatus.choices,
        default=OrderStatus.PENDING
    )
    
    class Meta:
        db_table = 'orders'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['status', 'created_at']),
            models.Index(fields=['public_id']),
        ]
    
    def __str__(self):
        return f'Order {self.public_id}'


# ❌ BAD: Nested choices (should be in constants.py)
class Order(models.Model):
    class Status(models.TextChoices):  # Should be in constants.py!
        PENDING = 'pending', 'Pending'
```

**Rules:**
- UUID for public identifiers
- UUID for public, id for internal (security)
- Define `__str__` and `class Meta`
- Avoid complex logic (use services)
- Status choices in constants.py (avoid circular imports)
- Add indexes for search fields

**Reference**: `references/models-and-orm.md`, `references/constants.md`

---

### 6. Constants

**All constants in separate file.**

```python
# constants.py
class OrderStatus(models.TextChoices):
    PENDING = 'pending', 'Pending'
    PAID = 'paid', 'Paid'
    SHIPPED = 'shipped', 'Shipped'
    CANCELLED = 'cancelled', 'Cancelled'


MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT = 30
TAX_RATE = Decimal('0.21')


# services.py
from .constants import OrderStatus, MAX_RETRY_ATTEMPTS


class OrderService:
    @staticmethod
    def process_payment(order_id: int):
        for attempt in range(MAX_RETRY_ATTEMPTS):
            ...


# ❌ BAD: Magic numbers and inline choices
class Order(models.Model):
    class Status(models.TextChoices):  # Should be in constants.py!
        PENDING = 'pending', 'Pending'
    
    # Magic number!
    def calculate_fee(self):
        return self.total * 0.21  # What is 0.21?
```

**Reference**: `references/constants.md`

---

### 7. Filters

**Use django-filters with custom FilterSets.**

```python
# apps/orders/api/v1/order_list/filters.py
import django_filters


class OrderFilter(django_filters.FilterSet):
    status = django_filters.CharFilter(field_name='status')
    min_total = django_filters.NumberFilter(field_name='total', lookup_expr='gte')
    max_total = django_filters.NumberFilter(field_name='total', lookup_expr='lte')
    created_after = django_filters.DateTimeFilter(
        field_name='created_at', 
        lookup_expr='gte'
    )
    
    class Meta:
        model = Order
        fields = ['status', 'customer']


# views.py
class OrderViewSet(viewsets.ModelViewSet):
    filterset_class = OrderFilter


# ❌ BAD: filter_backends in view when it's in settings
class OrderViewSet(viewsets.ModelViewSet):
    filter_backends = [DjangoFilterBackend]  # Already in settings!
    ...
```

**Rules:**
- Use django-filters in DRF
- Define custom FilterSets in `apps/{name}/api/{version}/{name_function}/filters/`
- No docstrings in filters
- filter_backends in settings, not in views

**Reference**: `references/filters.md`

---

### 8. Business Logic

**Implement rules in services, use transactions.**

```python
# services.py
import logging
from django.db import transaction

logger = logging.getLogger(__name__)


class OrderService:
    @staticmethod
    @transaction.atomic
    def process_payment(order_id: int, payment_token: str) -> dict:
        logger.info(f'Processing payment for order {order_id}')
        
        try:
            order = Order.objects.select_for_update().get(pk=order_id)
            
            if order.status != OrderStatus.PENDING:
                logger.warning(f'Order {order_id} not pending')
                raise ValidationError('Order not pending')
            
            result = PaymentGateway.charge(order.total, payment_token)
            
            order.status = OrderStatus.PAID
            order.save()
            
            logger.info(f'Payment successful for order {order_id}')
            return {'success': True, 'order_id': order_id}
        
        except PaymentError as e:
            logger.error(f'Payment failed for order {order_id}: {e}')
            raise


# ❌ BAD: Logic in view, no transaction
def process_payment(request, order_id):
    order = Order.objects.get(pk=order_id)
    charge_card(order.total)  # No transaction!
    order.status = 'paid'
    order.save()
```

**Rules:**
- Implement rules in services
- Use transactions for complex operations
- Log meaningful information
- Handle exceptions properly
- Avoid domain logic in APIs

**Reference**: `references/advanced-patterns.md`

---

### 9. Code Quality

**Follow Boy Scout Rule, DRY, KISS, YAGNI, SOLID.**

```python
# constants.py
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT = 30
TAX_RATE = Decimal('0.21')


# services.py
from .constants import MAX_RETRY_ATTEMPTS, DEFAULT_TIMEOUT


def calculate_order_total(items: list[OrderItem]) -> Decimal:
    """Calculate total for order items."""
    subtotal = sum(item.price * item.quantity for item in items)
    tax = subtotal * TAX_RATE
    return subtotal + tax


# ❌ BAD: Magic numbers, long function, poor names
def process(o):
    x = 0
    for i in o.stuff:
        x += i.p * i.q
    if x > 100:
        x = x * 1.1  # What is 1.1?
    # ... 100 more lines ...
```

**Rules:**
- Follow Boy Scout Rule, DRY, KISS, YAGNI, SOLID
- Keep functions short (50-70 LoC)
- Use descriptive names (no comments needed)
- Avoid magic numbers (use constants)
- Remove dead code
- Prefer clarity over tricks

**Reference**: `references/code-quality.md`

---

### 10. Security

```python
# ✅ GOOD: ORM, parameterized, minimum permissions
class IsOrderOwner(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.customer == request.user


# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}


# ❌ BAD: Raw SQL with string formatting
def get_orders(request):
    user_id = request.GET.get('user_id')
    # SQL INJECTION VULNERABILITY!
    cursor.execute(f'SELECT * FROM orders WHERE user_id = {user_id}')
```

**Rules:**
- Follow OWASP Top 10
- Never commit secrets
- Sanitize inputs
- Always use ORM (if RawSQL needed, sanitize)
- Minimum permissions

**Reference**: `references/security-checklist.md`

---

### 11. Error Handling DRF

**Use DRF exceptions with drf-standardized-errors.**

```python
# settings.py
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'drf_standardized_errors.handler.exception_handler',
}


# views.py
from rest_framework.exceptions import ValidationError, NotFound


class OrderViewSet(viewsets.ModelViewSet):
    def perform_update(self, serializer):
        order = self.get_object()
        
        if order.status == OrderStatus.SHIPPED:
            raise ValidationError({
                'status': 'Cannot update shipped orders'
            })
        
        serializer.save()


# ❌ BAD: Generic exceptions
def update_order(request, pk):
    order = Order.objects.get(pk=pk)  # Raises generic DoesNotExist
    order.save()
```

**Reference**: `references/drf-guidelines.md`

---

### 12. Testing

```python
from django.test import TestCase
from unittest.mock import patch
from myapp.services import OrderService
from myapp.factories import UserFactory, ProductFactory


class OrderServiceTest(TestCase):
    def setUp(self):
        self.customer = UserFactory()
        self.product = ProductFactory(price=100)
    
    def test_create_order_success(self):
        data = OrderData(
            customer_id=self.customer.id,
            items=[{'product_id': self.product.id, 'quantity': 2}],
            shipping_address='123 Test St'
        )
        
        order = OrderService.create_order(data)
        
        self.assertEqual(order.total, 200)
        self.assertEqual(order.status, OrderStatus.PENDING)
    
    @patch('requests.post')
    def test_process_payment_success(self, mock_post):
        mock_post.return_value.json.return_value = {'success': True}
        
        order = OrderFactory(customer=self.customer)
        
        result = OrderService.process_payment(order.id, 'token123')
        
        self.assertTrue(result['success'])


# ❌ BAD: Mocking the service itself
@patch('myapp.services.OrderService.create_order')
def test_bad_mock(self, mock_create):
    # Don't mock the service you're testing!
    ...
```

**Rules:**
- Unit tests for services
- Use FactoryBoy for test data
- Mock external services (requests.get/post)
- **DON'T mock the service itself**

**Reference**: `references/testing-strategies.md`

---

### 13. Documentation

```python
# ✅ GOOD: Type hints, comments explain WHY
from typing import Optional


def process_refund(order_id: int, amount: Optional[Decimal] = None) -> Refund:
    # Use full amount if not specified (most common case)
    refund_amount = amount or order.total
    
    # Validate refund doesn't exceed order total
    if refund_amount > order.total:
        raise ValidationError('Refund exceeds order total')
    
    return RefundService.create(order, refund_amount)


# ❌ BAD: Useless docstring
def process_refund(order_id, amount):
    """Process refund for an order."""  # OBVIOUS from function name!
    ...


# ❌ BAD: No type hints, comments explain WHAT
def process_refund(order_id, amount):
    # Get the order  (obvious from code)
    order = Order.objects.get(pk=order_id)
    # Create refund  (obvious from code)
    return Refund.objects.create(...)
```

**Rules:**
- Use type hints
- Update documentation with changes
- Comments explain WHY, not WHAT
- Avoid redundant docstrings

**Reference**: `references/code-quality.md`

---

### 14. Imports

```python
# ✅ GOOD: All imports at top
from django.db import models
from .constants import OrderStatus
from .services import OrderService


class Order(models.Model):
    status = models.CharField(
        max_length=20,
        choices=OrderStatus.choices,
        default=OrderStatus.PENDING
    )


# ❌ BAD: Import inside function
class Order(models.Model):
    def calculate_total(self):
        from .services import OrderService  # WRONG!
        return OrderService.calculate(self)


# ❌ BAD: Import inside class
class Order(models.Model):
    from .constants import OrderStatus  # WRONG!
    
    status = models.CharField(...)
```

**Rule: ALL imports at module level, NEVER inside functions/classes.**

---

### 15. Signals

**Use for critical processes, but avoid complex chains.**

```python
# ✅ GOOD: Simple signal for notification
from django.db.models.signals import post_save
from django.dispatch import receiver


@receiver(post_save, sender=Order)
def send_order_notification(sender, instance, created, **kwargs):
    if created:
        NotificationService.send_confirmation(instance)


# ✅ GOOD: Signal triggers Celery task
@receiver(post_save, sender=Order)
def process_order_async(sender, instance, created, **kwargs):
    if created:
        process_order_task.delay(instance.id)


# ❌ BAD: Signal modifies multiple models
@receiver(post_save, sender=Order)
def update_related_models(sender, instance, **kwargs):
    # Hard to debug!
    instance.customer.orders_count += 1
    instance.customer.save()
    
    instance.product.sales_count += 1
    instance.product.save()
    
    Inventory.objects.update(...)
```

**Rules:**
- Signals for critical processes
- Signals can call Celery tasks
- **AVOID** signals that modify multiple models (hard to debug)
- Consider service layer instead

**Reference**: `references/signals-guide.md`

---

### 16. Celery Tasks

**Validate before creating, verify state before actions.**

```python
# tasks.py
from celery import shared_task
import logging

logger = logging.getLogger(__name__)


@shared_task(bind=True, max_retries=3)
def process_order_task(order_id: int):
    logger.info(f'Processing order {order_id}')
    
    # Validate order exists
    try:
        order = Order.objects.get(pk=order_id)
    except Order.DoesNotExist:
        logger.warning(f'Order {order_id} not found')
        return
    
    # Verify state
    if order.status != OrderStatus.PENDING:
        logger.info(f'Order {order_id} already processed')
        return
    
    # Process...
    ...


@shared_task(bind=True, max_retries=3)
def send_order_email_task(order_id: int):
    logger.info(f'Sending email for order {order_id}')
    
    # Verify order exists
    try:
        order = Order.objects.get(pk=order_id)
    except Order.DoesNotExist:
        logger.warning(f'Order {order_id} not found')
        return
    
    # Verify email not already sent
    if order.confirmation_email_sent:
        logger.info(f'Email already sent for order {order_id}')
        return
    
    # Send email...
    ...


# ❌ BAD: No validation
@shared_task
def process_order_task(order_id: int):
    order = Order.objects.get(pk=order_id)  # May not exist!
    
    # No state check
    order.status = OrderStatus.PROCESSED
    order.save()
```

**Rules:**
- Validate if entity exists before creating
- Verify state before actions (e.g., check if email already sent)
- Use proper logging
- Handle errors with retries

**Reference**: `references/celery-patterns.md`

---

### 17. Naming Conventions

```python
# ✅ GOOD: Follow conventions
class OrderViewSet(viewsets.ModelViewSet):      # PascalCase + ViewSet
    ...

class OrderSerializer(serializers.ModelSerializer):  # PascalCase + Serializer
    ...

class OrderFilter(django_filters.FilterSet):    # PascalCase + Filter
    ...

class StatusTransition:                         # PascalCase for service
    ...

def get_queryset(self):                         # snake_case for methods
    ...

user_profile = request.user.profile             # snake_case for variables

MAX_RETRY_ATTEMPTS = 3                          # UPPER_CASE for constants
DEFAULT_TIMEOUT = 30
```

**Quick Reference:**
- Classes: PascalCase (e.g., `UserSerializer`)
- Methods/functions: snake_case (e.g., `get_queryset`)
- Variables: snake_case (e.g., `user_profile`)
- Constants: UPPER_CASE (e.g., `MAX_RETRY_ATTEMPTS`)
- Serializers: Suffix `Serializer`
- ViewSets: Suffix `ViewSet`
- Services: Descriptive (e.g., `StatusTransition`)
- FilterSets: Suffix `Filter`

**Reference**: Quick reference above, detailed in `references/code-quality.md`

---

## Bundled Resources

**references/** - Comprehensive documentation loaded as needed

> **IMPORTANT**: Use the Read tool to load these files when needed.
> Example: `Read references/code-organization.md`

- **`code-organization.md`** (973 lines)
  - Early return pattern
  - Bool trap avoidance
  - Naming conventions
  - File organization
  - Import patterns

- **`code-quality.md`** (554 lines)
  - Boy Scout Rule, DRY, KISS, YAGNI, SOLID
  - Function length guidelines
  - Magic numbers avoidance
  - Dead code removal
  - Documentation guidelines

- **`models-and-orm.md`** (685 lines)
  - Model design patterns
  - UUID vs id for security
  - Relationships (FK, M2M, O2O)
  - Custom managers
  - Migrations
  - N+1 query prevention
  - Database indexes

- **`views-and-urls.md`** (599 lines)
  - FBV vs CBV decision guide
  - Service layer integration
  - URL patterns
  - Middleware
  - Error handling

- **`drf-guidelines.md`** (383 lines)
  - Serializer patterns
  - Viewsets vs APIViews
  - Permissions (in settings)
  - Pagination & filtering
  - drf-standardized-errors
  - API versioning

- **`filters.md`** (528 lines)
  - django-filters integration
  - Custom FilterSets
  - Filter organization
  - Performance

- **`testing-strategies.md`** (639 lines)
  - TestCase patterns
  - APITestCase for DRF
  - FactoryBoy for test data
  - Mocking external services
  - Test organization

- **`security-checklist.md`** (640 lines)
  - OWASP Top 10
  - Authentication
  - Authorization
  - Input validation
  - HTTPS/SSL

- **`performance-optimization.md`** (698 lines)
  - Query optimization
  - Caching (Redis)
  - Database indexing
  - Profiling
  - Bulk operations

- **`advanced-patterns.md`** (480 lines)
  - Service layer (fuera del flujo básico de DRF/Django)
  - QuerySets y Managers
  - Fat Model Problem
  - Fat View Problem
  - Flujo correcto DRF y Django

- **`constants.md`** (463 lines)
  - Why separate constants
  - TextChoices organization
  - Avoiding circular imports
  - Examples

- **`celery-patterns.md`** (649 lines)
  - Task definition
  - Validation before creation
  - State verification
  - Error handling
  - Retry patterns

- **`signals-guide.md`** (554 lines)
  - When to use signals
  - When to avoid signals
  - Debugging difficulties
  - Examples

- **`admin-guide.md`** (290 lines)
  - Django Admin customization
  - List display options
  - Inline models
  - Custom admin views

- **`forms-modelforms.md`** (403 lines)
  - Form validation
  - ModelForm patterns
  - Form handling in views
  - Widget customization

- **`authentication.md`** (393 lines)
  - Custom user model
  - Login/logout
  - JWT authentication
  - Permission classes

- **`middleware.md`** (256 lines)
  - Custom middleware
  - Request/response processing
  - Authentication middleware

- **`migrations.md`** (287 lines)
  - Creating migrations
  - Data migrations
  - Best practices
  - Troubleshooting

- **`logging.md`** (290 lines)
  - Logging configuration
  - Logging in services
  - Celery task logging

- **`static-media-files.md`** (254 lines)
  - Static files configuration
  - Media file handling
  - Image processing

- **`project-structure.md`** (260 lines)
  - Project organization
  - Settings structure
  - Application patterns

- **`third-party-packages.md`** (197 lines)
  - Recommended packages
  - Package selection criteria

## Quick Reference

### Common Commands

```bash
# Run tests
python manage.py test

# Run specific test
python manage.py test myapp.tests.test_views.PostViewTest

# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Shell with models
python manage.py shell

# Check for issues
python manage.py check

# Collect static
python manage.py collectstatic
```

### Model Field Quick Reference

```python
# Strings
name = models.CharField(max_length=100)
description = models.TextField()

# Numbers
count = models.IntegerField()
price = models.DecimalField(max_digits=10, decimal_places=2)
rating = models.FloatField()

# Boolean
is_active = models.BooleanField(default=True)

# Dates
created_at = models.DateTimeField(auto_now_add=True)
updated_at = models.DateTimeField(auto_now=True)
birth_date = models.DateField()

# UUID (for public IDs)
public_id = models.UUIDField(default=uuid.uuid4, editable=False)

# Relationships
author = models.ForeignKey(User, on_delete=models.CASCADE)
tags = models.ManyToManyField(Tag)
profile = models.OneToOneField(Profile, on_delete=models.CASCADE)

# Choices (from constants.py)
status = models.CharField(max_length=20, choices=Status.choices, default=Status.DRAFT)
```

### QuerySet Quick Reference

```python
# Basic queries
Post.objects.all()
Post.objects.filter(status='published')
Post.objects.get(pk=1)
Post.objects.get_or_create(title='Test')

# Relationships
Post.objects.select_related('author')  # FK
Post.objects.prefetch_related('tags')  # M2M

# Optimization
Post.objects.only('title', 'author')
Post.objects.defer('content')
Post.objects.annotate(comment_count=Count('comments'))

# Bulk
Post.objects.bulk_create([post1, post2])
Post.objects.update(status='published')
Post.objects.filter(pk__in=[1, 2, 3]).delete()
```

## Version Notes

**Django 4.2 LTS** (April 2023 - April 2026)
- Async interfaces
- Coveralls in templates
- Improved query performance

**Django 5.0** (December 2023)
- Generated fields
- Asynchronous handlers
- Form rendering simplification

**Django 5.2** (Expected April 2025)
- Next LTS release

Always check the [Django release notes](https://docs.djangoproject.com/en/stable/releases/) for deprecations when upgrading.

---
> Source: [aztrock/django-general-expert](https://github.com/aztrock/django-general-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
