## order-return-management

> - Use descriptive names that indicate the scenario being tested

# Java Unit Testing Principles

## Test Method Naming

### ✅ **Good Test Names:**
- Use descriptive names that indicate the scenario being tested
- Follow the pattern: `should[ExpectedBehavior]When[Condition]` or `should[ExpectedBehavior]`
- **Never start with "test"** - this is redundant
- **No underscores** in test method names
- Be concise but meaningful

**Examples:**
```java
@Test
public void shouldReturnWarehouseNameWhenFound() { }
@Test
public void shouldCacheResultsWhenEnabled() { }
@Test
public void shouldHandleNullResponse() { }
@Test
public void shouldRejectInvalidParameters() { }
```

### ❌ **Avoid:**
```java
@Test
public void testGetWarehouseName() { }  // Don't start with "test"
@Test
public void should_return_warehouse_name() { }  // No underscores
@Test
public void test() { }  // Too generic
```

## Test Structure

### **GIVEN/WHEN/THEN Pattern:**
Every test method should have clear sections with comments:

```java
@Test
public void shouldCacheResultsWhenEnabled() {
  // GIVEN
  DwxApiClient client = new DwxApiClient(dwxApi, 1000, 10, true);
  VirtualWarehouse warehouse = createWarehouse(WAREHOUSE_ID_1, WAREHOUSE_NAME_1);
  when(dwxApi.getVirtualWarehouse()).thenReturn(Collections.singletonList(warehouse));

  // WHEN
  Optional<String> firstResult = client.getVirtualWarehouseName(WAREHOUSE_ID_1);
  Optional<String> secondResult = client.getVirtualWarehouseName(WAREHOUSE_ID_1);

  // THEN
  assertThat(firstResult).hasValue(WAREHOUSE_NAME_1);
  assertThat(secondResult).hasValue(WAREHOUSE_NAME_1);
  verify(dwxApi).getVirtualWarehouse(); // Called only once due to caching
  verifyNoMoreInteractions(dwxApi);
}
```

## Test Data and Constants

### **Naming Conventions:**
- Use descriptive constant names
- Avoid underscores in constant names
- Make constants `private static final`

**Examples:**
```java
private static final String WAREHOUSE_ID_1 = "dwx-123";
private static final String WAREHOUSE_NAME_1 = "impala-warehouse";
private static final String NONEXISTENT_ID = "non-existent-id";
```

### **Helper Methods:**
- Use descriptive method names
- Keep helper methods simple and focused

```java
private VirtualWarehouse createWarehouse(String id, String name) {
  VirtualWarehouse warehouse = new VirtualWarehouse();
  warehouse.virtualWarehouseId = id;
  warehouse.virtualWarehouseName = name;
  return warehouse;
}
```

## Test Coverage Principles

### **Critical Scenarios to Cover:**
1. **Constructor validation** - Invalid parameters, null checks
2. **Happy path scenarios** - Normal operation
3. **Edge cases** - Null values, empty collections, boundary conditions
4. **Error scenarios** - Exceptions, API failures
5. **Configuration variations** - Different settings, enabled/disabled features
6. **Concurrent access** - Thread safety when applicable
7. **Resource management** - Cache behavior, expiration, size limits

### **Example Test Categories:**
```java
// Constructor and validation
@Test public void shouldRejectZeroMaxCacheSize() { }
@Test public void shouldRejectNullDwxApi() { }

// Core functionality
@Test public void shouldCacheResultsWhenEnabled() { }
@Test public void shouldBypassCacheWhenDisabled() { }

// Edge cases
@Test public void shouldHandleNullApiResponse() { }
@Test public void shouldHandleEmptyApiResponse() { }
@Test public void shouldHandleApiException() { }

// Cache mechanics
@Test public void shouldExpireCacheEntries() { }
@Test public void shouldRespectCacheSizeLimit() { }
@Test public void shouldHandleConcurrentAccess() { }
```

## Maintainability Guidelines

### **Reduce Coupling:**
- Focus on **behavior**, not implementation details
- Avoid testing internal implementation specifics
- Use descriptive variable names that explain intent
- Minimize dependencies on specific implementation choices

### **Readability:**
- Keep test methods focused on a single scenario
- Use clear, descriptive variable names
- Add explanatory comments for complex verification steps
- Structure tests for easy understanding

### **Best Practices:**
- **One assertion per concept** - Group related assertions
- **Descriptive verification comments** - Explain what you're verifying
- **Consistent structure** - All tests follow the same pattern
- **Meaningful test data** - Use realistic, descriptive test values

## Mock Usage Guidelines

### **Mock Setup:**
```java
// GIVEN
@Mock private DwxApi dwxApi;
VirtualWarehouse warehouse = createWarehouse(WAREHOUSE_ID_1, WAREHOUSE_NAME_1);
when(dwxApi.getVirtualWarehouse()).thenReturn(Collections.singletonList(warehouse));
```

### **Verification:**
```java
// THEN
verify(dwxApi).getVirtualWarehouse(); // Called only once due to caching
verifyNoMoreInteractions(dwxApi); // No additional calls
```

## Assertion Guidelines

### **Use AssertJ for Readable Assertions:**
```java
// THEN
assertThat(result).hasValue(WAREHOUSE_NAME_1);
assertThat(result).isEmpty();
assertThat(client).isNotNull();
assertThatThrownBy(() -> new DwxApiClient(null, 1000, 10, true))
    .isInstanceOf(NullPointerException.class);
```

### **Exception Testing:**
```java
@Test
public void shouldRejectZeroMaxCacheSize() {
  // GIVEN
  int invalidMaxCacheSize = 0;
  long cacheExpiryMinutes = 10;
  boolean cacheEnabled = true;

  // WHEN & THEN
  assertThatThrownBy(() -> new DwxApiClient(dwxApi, invalidMaxCacheSize, cacheExpiryMinutes, cacheEnabled))
      .isInstanceOf(IllegalArgumentException.class)
      .hasMessage("Max cache size must be positive");
}
```

## File Organization

### **Test Class Structure:**
```java
public class DwxApiClientTest {
  // Constants
  private static final String WAREHOUSE_ID_1 = "dwx-123";
  
  // Mocks
  @Mock private DwxApi dwxApi;
  
  // Setup/Teardown
  @Before public void setup() { }
  @After public void tearDown() { }
  
  // Test methods (grouped by functionality)
  // Constructor tests
  // Core functionality tests
  // Edge case tests
  // Error scenario tests
  
  // Helper methods
  private VirtualWarehouse createWarehouse(String id, String name) { }
}
```

## Key Principles Summary

1. **Test names should indicate meaning** - Use `should[Behavior]` pattern
2. **No "test" prefix** - Redundant and not descriptive
3. **No underscores** - Use camelCase for test method names
4. **Clear GIVEN/WHEN/THEN structure** - Always comment these sections
5. **Readability is most important** - Tests serve as documentation
6. **Reduce coupling** - Focus on behavior, not implementation details
7. **Cover all critical scenarios** - Happy path, edge cases, errors, configuration variations

Remember: **Readability and maintainability are the most important aspects** of unit tests. Tests should serve as living documentation of your code's behavior.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kvin101) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
