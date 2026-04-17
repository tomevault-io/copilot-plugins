## copacracks-backend

> - Use Java records for Value Objects


# Cursor Rules - Domain Objects Pattern
# Padrões estabelecidos para Value Objects e Domain Entities

## Value Objects (Records)

### ALWAYS for Value Objects:
- Use Java records for Value Objects
- Always validate in canonical constructor using compact syntax: `public RecordName { validateMethod(field); }`
- Never create redundant factory methods like `of()` - constructor already validates
- Only create factory methods when they ADD VALUE (transformation, multiple creation paths)
- Always make validation methods private and static
- Always add utility validation method: `public static boolean isValidX(Type value)` for pre-validation
- Override isValid() to return true (if instance exists, it's valid)
- Never store transformed data in constructor - do it in factory methods
- Always handle null values gracefully in business methods

### Example Value Object Pattern:
```java
public record ValueObject(String value) {
    
    // Canonical constructor with validation
    public ValueObject {
        validateValue(value);
        // Optional normalization: value = value.trim();
    }
    
    // Factory method ONLY when adding value
    public static ValueObject fromSpecialFormat(String input) {
        String processed = processInput(input);
        return new ValueObject(processed);
    }
    
    // Private validation
    private static void validateValue(String value) {
        if (value == null || value.trim().isEmpty()) {
            throw new ValidationException("Value cannot be null or empty");
        }
        // Additional validations...
    }
    
    // Utility validation without creating instance
    public static boolean isValidValue(String value) {
        try {
            validateValue(value);
            return true;
        } catch (ValidationException e) {
            return false;
        }
    }
    
    // Always return true for existing instances
    public boolean isValid() {
        return true;
    }
}
```

## Domain Entities

### ALWAYS for Domain Entities:
- Use regular classes for rich domain models with behavior
- Make ID field final and private
- Use Value Objects for complex fields (Username, Email, Password, etc.)
- Prefer immutable design with "with" pattern over setters
- Implement equals/hashCode based on ID when available, all fields when ID is null
- Add business methods that express domain operations
- Never use Records for entities with behavior
- Constructor should accept both raw values (with validation) and Value Objects (pre-validated)

### Example Entity Pattern:
```java
public final class DomainEntity {
    private final Long id;
    private final ValueObject1 field1;
    private final ValueObject2 field2;
    
    // Constructor for new entity
    public DomainEntity(String field1, String field2) {
        this(null, field1, field2);
    }
    
    // Constructor for existing entity
    public DomainEntity(Long id, String field1, String field2) {
        this.id = id;
        this.field1 = new ValueObject1(field1);  // Validates automatically
        this.field2 = new ValueObject2(field2);  // Validates automatically
    }
    
    // Constructor with pre-validated Value Objects
    public DomainEntity(Long id, ValueObject1 field1, ValueObject2 field2) {
        this.id = id;
        this.field1 = field1;
        this.field2 = field2;
    }
    
    // Immutable updates - return new instance
    public DomainEntity withField1(String newField1) {
        return new DomainEntity(this.id, newField1, this.field2.value());
    }
    
    // Business methods
    public boolean isSomeBusinessCondition() {
        return field1.someMethod() && field2.someOtherMethod();
    }
    
    // Standard getters returning primitive values
    public String getField1() {
        return field1.value();
    }
    
    // equals/hashCode
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        DomainEntity entity = (DomainEntity) obj;
        
        if (id != null && entity.id != null) {
            return Objects.equals(id, entity.id);
        }
        
        return Objects.equals(field1, entity.field1) && 
               Objects.equals(field2, entity.field2);
    }
    
    @Override
    public int hashCode() {
        if (id != null) {
            return Objects.hash(id);
        }
        return Objects.hash(field1, field2);
    }
}
```

## General Rules

### NEVER:
- Create factory methods that are redundant with constructors
- Use Records for entities with business behavior
- Make entities mutable with setters
- Create Value Objects without validation
- Use localStorage or sessionStorage in artifacts
- Create validation methods that can be bypassed

### ALWAYS:
- Validate in Value Object constructors (canonical for records)
- Use private constructors + factory methods ONLY when constructor should be hidden
- Prefer composition with Value Objects over primitive obsession
- Make entities immutable when possible
- Handle null values gracefully
- Use meaningful names for factory methods (fromPlainText, fromHash, etc.)
- Add utility validation methods for pre-validation without instance creation

### Exception Handling:
- Always throw specific domain exceptions (UserValidationException, etc.)
- Never catch and ignore validation exceptions
- Provide clear error messages for validation failures

### Documentation:
- Always document factory methods explaining when to use them
- Document validation rules in Value Objects
- Explain business rules in Entity methods

### Performance:
- Validation should be fast and fail-fast
- Avoid expensive operations in constructors
- Cache validation results when appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OutbackAnon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
