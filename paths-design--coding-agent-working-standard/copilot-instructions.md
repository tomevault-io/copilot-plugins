## coding-agent-working-standard

> Language-agnostic engineering standards and universal code quality metrics


# Language-Agnostic Engineering Standards

## Core Principle

**Engineering standards must apply universally across all programming languages.** Focus on architectural patterns, quality metrics, and maintainability principles that transcend specific language syntax.

## Universal Code Quality Metrics

### Complexity Standards

**Cyclomatic Complexity:**

- **Maximum per function**: 10
- **Maximum per class/module**: 20
- **Critical functions**: 5 (auth, billing, data processing)

**Cognitive Complexity:**

- **Maximum per function**: 15
- **Maximum per class/module**: 30
- **UI components**: 20 (higher due to rendering logic)

**Nesting Depth:**

- **Maximum nesting levels**: 4
- **Preferred**: 2-3 levels with guard clauses
- **Exception**: Complex algorithms with documented justification

### Maintainability Standards

**Function Size:**

- **Maximum lines per function**: 50
- **Preferred**: 20-30 lines
- **Exception**: Complex algorithms with documentation

**File Size:**

- **Maximum lines per file**: 1000
- **Preferred**: 200-500 lines
- **Exception**: Generated code or data files

**Parameter Count:**

- **Maximum parameters per function**: 5
- **Preferred**: 2-3 parameters
- **Exception**: Configuration objects or builder patterns

**Class/Module Size:**

- **Maximum methods per class**: 10
- **Maximum fields per class**: 15
- **Preferred**: 5-7 methods, 8-10 fields

## Universal Architectural Patterns

### Dependency Injection

**All languages must support:**

- Constructor injection for dependencies
- Interface/contract-based design
- Testable dependency substitution
- Clear dependency graphs

**Implementation Examples:**

```typescript
// TypeScript/JavaScript
class OrderService {
  constructor(
    private userRepo: UserRepository,
    private paymentService: PaymentService,
    private notificationService: NotificationService
  ) {}
}
```

```rust
// Rust
struct OrderService {
    user_repo: Box<dyn UserRepository>,
    payment_service: Box<dyn PaymentService>,
    notification_service: Box<dyn NotificationService>,
}
```

```python
# Python
class OrderService:
    def __init__(self, user_repo: UserRepository, payment_service: PaymentService):
        self.user_repo = user_repo
        self.payment_service = payment_service
```

### Error Handling Patterns

**Universal Error Handling Requirements:**

- Explicit error types (no generic exceptions)
- Error context preservation
- Graceful degradation capabilities
- Comprehensive error logging

**Implementation Examples:**

```typescript
// TypeScript - Result pattern
type Result<T, E> = { success: true; data: T } | { success: false; error: E };

async function processOrder(order: Order): Promise<Result<OrderResult, OrderError>> {
  try {
    const result = await validateOrder(order);
    return { success: true, data: result };
  } catch (error) {
    return { success: false, error: new OrderError(error.message) };
  }
}
```

```rust
// Rust - Result type
fn process_order(order: Order) -> Result<OrderResult, OrderError> {
    let validated = validate_order(&order)?;
    let result = calculate_total(&validated)?;
    Ok(result)
}
```

```python
# Python - Custom exceptions
class OrderError(Exception):
    def __init__(self, message: str, context: dict = None):
        super().__init__(message)
        self.context = context or {}

def process_order(order: Order) -> OrderResult:
    try:
        validated = validate_order(order)
        return calculate_total(validated)
    except ValidationError as e:
        raise OrderError(f"Order validation failed: {e}", {"order_id": order.id})
```

### Configuration Management

**Universal Configuration Standards:**

- Environment-based configuration
- Type-safe configuration objects
- Validation of configuration values
- Secrets management integration

**Implementation Examples:**

```typescript
// TypeScript
interface Config {
  database: {
    host: string;
    port: number;
    ssl: boolean;
  };
  api: {
    timeout: number;
    retries: number;
  };
}

const config: Config = {
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT || '5432'),
    ssl: process.env.DB_SSL === 'true',
  },
  api: {
    timeout: parseInt(process.env.API_TIMEOUT || '5000'),
    retries: parseInt(process.env.API_RETRIES || '3'),
  },
};
```

```rust
// Rust
#[derive(Debug, Deserialize)]
struct Config {
    database: DatabaseConfig,
    api: ApiConfig,
}

#[derive(Debug, Deserialize)]
struct DatabaseConfig {
    host: String,
    port: u16,
    ssl: bool,
}

impl Config {
    fn from_env() -> Result<Self, ConfigError> {
        let host = env::var("DB_HOST").unwrap_or_else(|_| "localhost".to_string());
        let port = env::var("DB_PORT")
            .unwrap_or_else(|_| "5432".to_string())
            .parse()
            .map_err(|_| ConfigError::InvalidPort)?;

        Ok(Config {
            database: DatabaseConfig {
                host,
                port,
                ssl: env::var("DB_SSL").unwrap_or_else(|_| "false".to_string()) == "true",
            },
            api: ApiConfig::from_env()?,
        })
    }
}
```

## Universal Testing Standards

### Test Structure Requirements

**All languages must follow:**

- Given-When-Then structure
- Descriptive test names
- Independent test execution
- Proper setup/teardown

**Implementation Examples:**

```typescript
// TypeScript/JavaScript
describe('OrderService', () => {
  describe('when processing a valid order', () => {
    it('should return order confirmation', async () => {
      // Given: Valid order data
      const order = createValidOrder();

      // When: Processing the order
      const result = await orderService.processOrder(order);

      // Then: Should return confirmation
      expect(result.success).toBe(true);
      expect(result.data.orderId).toBeDefined();
    });
  });
});
```

```rust
// Rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_process_valid_order() {
        // Given: Valid order data
        let order = create_valid_order();

        // When: Processing the order
        let result = order_service.process_order(order).await;

        // Then: Should return confirmation
        assert!(result.is_ok());
        let order_result = result.unwrap();
        assert!(order_result.order_id.is_some());
    }
}
```

```python
# Python
class TestOrderService:
    def test_process_valid_order_returns_confirmation(self):
        # Given: Valid order data
        order = self.create_valid_order()

        # When: Processing the order
        result = self.order_service.process_order(order)

        # Then: Should return confirmation
        assert result.success is True
        assert result.data.order_id is not None
```

### Test Data Management

**Universal Requirements:**

- Realistic test data factories
- Deterministic test execution
- Proper cleanup procedures
- Isolated test environments

## Universal Documentation Standards

### API Documentation

**All languages must provide:**

- Function/method signatures with types
- Parameter descriptions and constraints
- Return value specifications
- Error conditions and exceptions
- Usage examples

**Implementation Examples:**

````typescript
/**
 * Processes a customer order and returns confirmation details.
 *
 * @param order - The order to process with validated items and payment info
 * @param options - Optional processing options for customization
 * @returns Promise resolving to order confirmation or rejection details
 * @throws ValidationError when order data is invalid
 * @throws PaymentError when payment processing fails
 * @throws InventoryError when items are unavailable
 *
 * @example
 * ```typescript
 * const order = { items: [...], payment: {...} };
 * const result = await processOrder(order);
 * console.log(`Order ${result.orderId} confirmed`);
 * ```
 */
async function processOrder(order: Order, options?: ProcessingOptions): Promise<OrderResult> {
  // Implementation
}
````

````rust
/// Processes a customer order and returns confirmation details.
///
/// # Arguments
/// * `order` - The order to process with validated items and payment info
/// * `options` - Optional processing options for customization
///
/// # Returns
/// Returns `Ok(OrderResult)` with confirmation details on success
///
/// # Errors
/// * `ValidationError` - When order data is invalid
/// * `PaymentError` - When payment processing fails
/// * `InventoryError` - When items are unavailable
///
/// # Examples
/// ```rust
/// let order = Order { items: vec![...], payment: PaymentInfo {...} };
/// let result = process_order(order).await?;
/// println!("Order {} confirmed", result.order_id);
/// ```
pub async fn process_order(
    order: Order,
    options: Option<ProcessingOptions>
) -> Result<OrderResult, OrderError> {
    // Implementation
}
````

```python
def process_order(order: Order, options: Optional[ProcessingOptions] = None) -> OrderResult:
    """Process a customer order and return confirmation details.

    Args:
        order: The order to process with validated items and payment info
        options: Optional processing options for customization

    Returns:
        OrderResult with confirmation details

    Raises:
        ValidationError: When order data is invalid
        PaymentError: When payment processing fails
        InventoryError: When items are unavailable

    Example:
        >>> order = Order(items=[...], payment=PaymentInfo(...))
        >>> result = process_order(order)
        >>> print(f"Order {result.order_id} confirmed")
    """
    # Implementation
```

## Universal Security Standards

### Input Validation

**All languages must implement:**

- Schema-based validation
- Type checking at boundaries
- Sanitization of user input
- Length and format constraints

### Authentication & Authorization

**Universal Requirements:**

- Token-based authentication
- Role-based access control
- Session management
- Audit logging

### Data Protection

**Universal Standards:**

- Encryption at rest and in transit
- Secure key management
- Data anonymization capabilities
- Privacy compliance measures

## Universal Performance Standards

### Response Time SLAs

**Universal Performance Requirements:**

- API endpoints: P95 < 250ms
- Database queries: P95 < 100ms
- File operations: P95 < 500ms
- Complex computations: P95 < 2000ms

### Resource Management

**Universal Resource Standards:**

- Memory usage monitoring
- CPU utilization limits
- Connection pool management
- Garbage collection optimization

## Universal Monitoring Standards

### Logging Requirements

**All applications must provide:**

- Structured logging (JSON format)
- Log levels (ERROR, WARN, INFO, DEBUG)
- Request correlation IDs
- Performance metrics

### Metrics Collection

**Universal Metrics:**

- Request/response times
- Error rates and types
- Resource utilization
- Business KPIs

### Health Checks

**Universal Health Endpoints:**

- Application health status
- Dependency health checks
- Resource availability
- Business logic validation

## CAWS Integration

### Quality Gate Integration

```bash
# Run complexity analysis
caws quality-gates --check-complexity

# Check architectural patterns
caws quality-gates --check-architecture

# Validate universal standards
caws quality-gates --check-universal-standards
```

### Metrics Tracking

```bash
# Track complexity metrics
caws metrics track --metric="cyclomatic_complexity" --value=8

# Track maintainability metrics
caws metrics track --metric="function_size" --value=25

# Track architectural compliance
caws metrics track --metric="dependency_injection" --value=1
```

### Progress Tracking

```bash
# Update progress with complexity compliance
caws progress update --criterion-id="COMPLEXITY-001" --status="completed"

# Track architectural pattern implementation
caws progress update --criterion-id="ARCHITECTURE-001" --status="in_progress"
```

## Enforcement Mechanisms

### Static Analysis Integration

**Universal Static Analysis:**

- Complexity analysis
- Dependency analysis
- Security vulnerability scanning
- Code quality metrics

### CI/CD Integration

**Universal CI/CD Requirements:**

- Automated testing
- Quality gate enforcement
- Security scanning
- Performance regression testing

### CAWS Quality Gates

```yaml
# In .caws/working-spec.yaml
quality_gates:
  complexity:
    max_cyclomatic: 10
    max_cognitive: 15
    max_nesting: 4

  maintainability:
    max_function_lines: 50
    max_file_lines: 1000
    max_parameters: 5

  architecture:
    dependency_injection: required
    error_handling: required
    configuration_management: required
```

## Quality Metrics Dashboard

### Universal Quality Score

**Calculation Formula:**

```
Quality Score = (
  Complexity Score * 0.2 +
  Test Coverage Score * 0.3 +
  Security Score * 0.2 +
  Performance Score * 0.15 +
  Documentation Score * 0.15
) * 100
```

### Trend Analysis

**Track over time:**

- Quality score trends
- Complexity growth
- Test coverage changes
- Security vulnerability rates
- Performance degradation

### CAWS Integration

```bash
# Generate quality report
caws quality-report --include-complexity --include-architecture

# Track quality trends
caws metrics trends --metric="quality_score" --period="30d"

# Export quality metrics
caws metrics export --format=json --include-universal-standards
```

This rule ensures consistent engineering standards across all programming languages while respecting language-specific idioms and capabilities, fully integrated with CAWS quality gates and metrics tracking.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Paths-Design)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Paths-Design)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
