## validation-decorators

> **IMPORTANT: This rule is configured to automatically apply to all files ending with .dto.ts**

# DTO Validation Standards [AUTO-ATTACHED TO ALL *.dto.ts FILES]

**IMPORTANT: This rule is configured to automatically apply to all files ending with .dto.ts**

## Use Class Validator Decorators

Always use [class-validator](mdc:https:/github.com/typestack/class-validator) decorators when creating DTO (Data Transfer Object) classes to ensure proper validation of incoming data.

### Required Pattern

```typescript
import { IsString, IsNumber, IsOptional, MaxLength, IsEmail } from 'class-validator';

export class UserDto {
  @IsString()
  @MaxLength(100)
  name: string;
  
  @IsEmail()
  email: string;
  
  @IsNumber()
  age: number;
  
  @IsString()
  @IsOptional()
  address?: string;
}
```

### Common Validators

Use these common validators based on the property type:

- `@IsString()` - For string properties
- `@IsNumber()` - For number properties
- `@IsBoolean()` - For boolean properties
- `@IsDate()` - For date properties
- `@IsEmail()` - For email properties
- `@IsOptional()` - For optional properties (place this before other validators)
- `@MaxLength(n)` - For limiting string length
- `@MinLength(n)` - For ensuring minimum string length
- `@Min(n)` - For minimum numeric values
- `@Max(n)` - For maximum numeric values
- `@IsNotEmpty()` - To ensure the value is not empty

### Examples in the Codebase

See these examples in our codebase:

- [BasicWhatsappDto](mdc:src/utils/whatsapp/dto/basic_whatsapp.dto.ts) - Shows basic validation
- [PaymentReceivedNotificationDto](mdc:src/utils/whatsapp/dto/payment_received_notification.dto.ts) - Shows inheritance and additional validation

### Benefits

- Runtime validation of request data
- Self-documenting code
- Consistent error responses
- Type safety
- Separation of validation logic from business logic
- Easy integration with NestJS pipes

### ⚠️ Do Not Skip Validation

Never create DTO classes without validation decorators. Missing validation can lead to:

- Runtime errors
- Security vulnerabilities 
- Inconsistent data
- Harder debugging

---
> Source: [WebNaresh/abhyasika_microservice_1](https://github.com/WebNaresh/abhyasika_microservice_1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
