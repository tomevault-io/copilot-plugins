## home-tour

> **DTO Refactoring Manual**


**DTO Refactoring Manual**

**Purpose:** This manual provides guidelines for refactoring and improving Data Transfer Objects DTOs) in a NestJS application.

**Goals:**

* Improve code readability and maintainability
* Follow NestJS best practices
* Enhance code consistency and flexibility

**Refactoring Steps:**

### 1. Consistent Naming Conventions

* Use `PascalCase` for class and interface names (e.g., `UserCreateDto` instead of `userCreateDto`).
* Use `camelCase` for property names (e.g., `userName` instead of `username`).

### 2. Validate Decorators

* Use `@ValidateNested()` instead of `@Validate()` when validating nested objects.
* Consider using `@ApiModelProperty()` to add documentation for each property.

### 3. Use Interfaces for DTOs

* Instead of using classes for DTOs, use interfaces. This approach is more lightweight and flexible.
* Use the `@nestjs/mapped-types` package to create mapped types for your DTOs.

### 4. Simplify Constructor Logic

* Instead of using constructors with multiple arguments, use a single object argument and estructuring it.
* Use the `class-transformer` package to automatically convert JSON data to DTO objects.

### 5. Remove Unnecessary Dependencies

* Remove unnecessary imports and dependencies from DTOs.

### 6. Consistent Use of Decorators

* Use `@IsOptional()` consistently throughout the DTOs to indicate optional properties.
* Use `@IsEnum()` consistently throughout the DTOs to indicate enum values.

### 7. Use NestJS Built-in Validators

* Use NestJS built-in validators (e.g., `@IsString()`, `@IsEnum()`) instead of custom implementations.
* Configure validator options globally whenever possible.

### 8. Remove Redundant Imports

* Remove unused imports from DTOs.

### 9. Optional and Not Empty Validators

* When using `@IsOptional()` and `@IsNotEmpty()` together, `@IsOptional()` takes precedence. This means that if the property is not provided, it won't be validated, but if it is provided, the u/IsNotEmpty()` decorator will ensure it's not an empty string.

**Example Refactored DTO:**
```typescript
import { IsString, IsEnum, IsOptional } from 'class-validator';
import { EnumExample } from './enum-example.enum';

export interface ExampleDto {
u/IsString()
name: string;

@IsOptional()
@IsEnum(EnumExample)
status?: EnumExample;

@IsOptional()
@IsDate()
createdAt?: Date;
}
```        

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ntnduc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
