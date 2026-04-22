## housef3

> Backend Pydantic model patterns and conventions


# Backend Models Conventions

## Pydantic Model Architecture

### Model Structure Pattern
All domain models follow this standard structure:
```python
class ModelName(BaseModel):
    """Main domain model with full business logic"""
    # Core fields with proper types and aliases
    # Validators and business methods
    # DynamoDB serialization methods

class ModelNameCreate(BaseModel):
    """DTO for creating new instances - required fields only"""
    # Subset of fields needed for creation
    # Creation-specific validation

class ModelNameUpdate(BaseModel):
    """DTO for updates - all fields optional"""
    # All fields optional for partial updates
    # Update-specific validation
```

### Field Conventions

#### Field Naming & Aliases
- Use snake_case for Python field names
- Use camelCase aliases for API/JSON serialization
- Always provide aliases for external-facing fields:
```python
account_id: uuid.UUID = Field(default_factory=uuid.uuid4, alias="accountId")
user_id: str = Field(alias="userId")
created_at: int = Field(default_factory=timestamp_ms, alias="createdAt")
```

#### Required vs Optional Fields
- Use explicit `Optional[Type]` for nullable fields
- Set sensible defaults with `Field(default=value)`
- Use `Field(default_factory=func)` for dynamic defaults
- Required fields have no default value

#### Field Validation
- Use `field_validator` for single field validation
- Use `model_validator` for cross-field validation
- Validate at creation time, not just at runtime:
```python
@field_validator('created_at', 'updated_at')
@classmethod
def check_positive_timestamp(cls, v: int) -> int:
    if v < 0:
        raise ValueError("Timestamp must be positive")
    return v

@model_validator(mode='after')
def check_currency_if_balance_exists(self) -> Self:
    if self.balance is not None and self.currency is None:
        raise ValueError("Currency required when balance is set")
    return self
```

## Enum Handling

### String Enum Pattern
All enums inherit from `str` for JSON serialization:
```python
class Status(str, enum.Enum):
    ACTIVE = "active"
    INACTIVE = "inactive"
    PENDING = "pending"
```

### Enum Validation
Validate enums properly in field validators:
```python
@field_validator('currency', mode='after')
@classmethod
def validate_currency(cls, v, info: ValidationInfo) -> Optional[Currency]:
    if v is None:
        return None
    if isinstance(v, Currency):
        return v
    # Check database deserialization context
    if info.context and info.context.get('from_database'):
        return v
    raise ValueError(f"Currency must be Currency enum, got {type(v).__name__}")
```

### Enum Conversion Utilities
Provide helper functions for API input conversion:
```python
def convert_currency_input(currency_input: Any) -> Optional[Currency]:
    """Convert API input to Currency enum with proper error handling"""
    if currency_input is None:
        return None
    if isinstance(currency_input, Currency):
        return currency_input
    if isinstance(currency_input, str):
        try:
            return Currency(currency_input)
        except ValueError:
            valid_options = ', '.join([c.value for c in Currency])
            raise ValueError(f"Invalid currency: '{currency_input}'. Valid: {valid_options}")
    raise ValueError(f"Currency must be string or enum, got {type(currency_input).__name__}")
```

## DynamoDB Integration

### Serialization Pattern
Every model must implement DynamoDB serialization:
```python
def to_dynamodb_item(self) -> Dict[str, Any]:
    """Serialize to DynamoDB item format"""
    item = self.model_dump(mode='python', by_alias=True, exclude_none=True)
    
    # Convert UUIDs to strings
    if 'accountId' in item and isinstance(item['accountId'], uuid.UUID):
        item['accountId'] = str(item['accountId'])
    
    # Convert Decimals to strings for DynamoDB
    if 'balance' in item and isinstance(item['balance'], Decimal):
        item['balance'] = str(item['balance'])
    
    # Convert enums to string values
    if 'currency' in item and item.get('currency') is not None:
        item['currency'] = item['currency'].value if hasattr(item['currency'], 'value') else str(item['currency'])
    
    return item
```

### Deserialization Pattern
Use `model_construct()` to preserve enum objects:
```python
@classmethod
def from_dynamodb_item(cls, data: Dict[str, Any]) -> Self:
    """Deserialize from DynamoDB item"""
    # Copy to avoid modifying original
    converted_data = data.copy()
    
    # Convert Decimal fields
    if 'balance' in converted_data and converted_data['balance'] is not None:
        try:
            converted_data['balance'] = Decimal(str(converted_data['balance']))
        except decimal.InvalidOperation:
            raise ValueError(f"Invalid decimal: {converted_data['balance']}")
    
    # Convert string enums to enum objects
    if 'currency' in converted_data and isinstance(converted_data.get('currency'), str):
        try:
            converted_data['currency'] = Currency(converted_data['currency'])
        except ValueError:
            logger.warning(f"Invalid currency from DB: {converted_data['currency']}")
            converted_data['currency'] = None
    
    # Use model_construct to preserve enum objects
    return cls.model_construct(**converted_data)
```

### Critical DynamoDB Rules
1. **Always copy data** before modification in `from_dynamodb_item()`
2. **Use `model_construct()`** instead of `model_validate()` to preserve enums
3. **Convert enums manually** from strings to enum objects before construction
4. **Handle Decimal conversion** explicitly for numeric fields
5. **Set context** when using `model_validate()`: `context={'from_database': True}`

## ConfigDict Standards

### Standard Configuration
All models use consistent ConfigDict:
```python
model_config = ConfigDict(
    populate_by_name=True,           # Allow both field names and aliases
    json_encoders={                  # Handle complex types in JSON
        Decimal: str,
        uuid.UUID: str
    },
    use_enum_values=True,           # Serialize enums as values
    arbitrary_types_allowed=True    # Allow custom types like Decimal
)
```

### When to Use `use_enum_values=True`
- **Always use** for consistent JSON serialization
- **Remember**: This affects `model_validate()` behavior
- **Mitigation**: Use `model_construct()` when preserving enum objects

## DTO Patterns

### Create DTOs
- Include only fields needed for creation
- Exclude auto-generated fields (IDs, timestamps)
- Include required validation for business rules:
```python
class AccountCreate(BaseModel):
    user_id: str = Field(alias="userId")
    account_name: str = Field(max_length=100, alias="accountName")
    account_type: AccountType = Field(alias="accountType")
    # ... other creation fields
    
    @model_validator(mode='after')
    def check_currency_if_balance_exists(self) -> Self:
        if self.balance is not None and self.currency is None:
            raise ValueError("Currency required with balance")
        return self
```

### Update DTOs
- All fields optional for partial updates
- Exclude immutable fields (user_id, created_at)
- Handle update-specific validation:
```python
class AccountUpdate(BaseModel):
    account_name: Optional[str] = Field(default=None, alias="accountName")
    # ... other updateable fields
    
    @model_validator(mode='after') 
    def validate_update_consistency(self) -> Self:
        # Cross-field validation for updates
        # Note: Cannot access existing entity state here
        # Complex validation belongs in service layer
        return self
```

### Update Method Pattern
Main models should provide update methods:
```python
def update_account_details(self, update_data: 'AccountUpdate') -> bool:
    """Update account with DTO data. Returns True if changed."""
    updated_fields = False
    update_dict = update_data.model_dump(exclude_unset=True, by_alias=False)
    
    for key, value in update_dict.items():
        if key not in ["account_id", "user_id", "created_at"] and hasattr(self, key):
            if getattr(self, key) != value:
                setattr(self, key, value)
                updated_fields = True
    
    if updated_fields:
        self.updated_at = int(datetime.now(timezone.utc).timestamp() * 1000)
    return updated_fields
```

## Validation Conventions

### Timestamp Validation
Standard pattern for timestamp fields:
```python
@field_validator('created_at', 'updated_at', 'assigned_at')
@classmethod
def check_positive_timestamp(cls, v: Optional[int]) -> Optional[int]:
    if v is not None and v < 0:
        raise ValueError("Timestamp must be positive milliseconds since epoch")
    return v
```

### Business Rule Validation
- Use `model_validator(mode='after')` for cross-field validation
- Keep validation focused on data integrity
- Move complex business logic to service layer
- Provide clear error messages with context

### Validation Context
Use validation context for different scenarios:
```python
# In from_dynamodb_item()
return cls.model_validate(data, context={'from_database': True})

# In field validators
if info.context and info.context.get('from_database'):
    # Handle database-specific conversion
    return v
```

## Event Models

### Use Dataclasses for Events
Events use `@dataclass` instead of Pydantic for performance:
```python
@dataclass
class BaseEvent:
    event_id: str
    event_type: str
    timestamp: int
    user_id: str
    data: Optional[Dict[str, Any]] = None
    
    def to_eventbridge_format(self) -> Dict[str, Any]:
        return {
            'Source': self.source,
            'DetailType': self.event_type,
            'Detail': json.dumps({...})
        }
```

### Event Initialization Pattern
Events auto-generate required fields:
```python
@dataclass
class FileUploadedEvent(BaseEvent):
    def __init__(self, user_id: str, file_id: str, **kwargs):
        super().__init__(
            event_id=str(uuid.uuid4()),
            event_type='file.uploaded',
            timestamp=int(datetime.now().timestamp() * 1000),
            user_id=user_id,
            data={'fileId': file_id, **kwargs}
        )
```

## Type Hints & Imports

### Standard Imports
```python
import uuid
import enum
import logging
from datetime import datetime, timezone
from decimal import Decimal
from typing import Optional, Dict, Any, List
from typing_extensions import Self

from pydantic import BaseModel, Field, field_validator, model_validator, ConfigDict, ValidationInfo
```

### Type Hint Conventions
- Use `Optional[Type]` for nullable fields
- Use `Self` for methods returning the same type
- Use `Dict[str, Any]` for flexible dictionaries
- Import `ValidationInfo` when using field validators with context

## Error Handling

### Validation Errors
Provide descriptive error messages:
```python
raise ValueError(f"Invalid currency: '{value}'. Valid options: {valid_list}")
raise ValueError(f"Currency must be Currency enum, got {type(v).__name__}: {v}")
```

### Logging in Models
- Use module-level logger: `logger = logging.getLogger(__name__)`
- Log warnings for data inconsistencies from database
- Don't log validation errors (let caller handle)

## Model Organization

### File Structure
- One main model per file (Account, Transaction, Category)
- Include related DTOs in same file (AccountCreate, AccountUpdate)
- Include related enums and utilities in same file
- Separate complex shared types (Money, Currency) into own files

### Documentation
- Include docstrings for all models and complex methods
- Document enum values and their meanings
- Explain validation rules and business constraints
- Document DynamoDB serialization behavior

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mazerunner70) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
