## fintrack

> DTO and validation patterns - records, Bean Validation, request/response naming


# DTO & Validation Patterns

## Use Java Records

All DTOs are Java `record` types. Never use classes with getters/setters for DTOs:

```java
public record CreateCreditCardRequest(
    @NotBlank(message = "Credit card name is required.")
    @Size(min = 2, max = 100, message = "Credit card name must be between 2 and 100 characters.")
    String name,

    @NotNull(message = "Credit limit is required.")
    @Positive(message = "Credit limit must be positive.")
    BigDecimal limit,

    Long parentCardId  // nullable optional fields have no annotation
) {}
```

## Naming Convention

| Type | Pattern | Example |
|------|---------|---------|
| Create/Update request | `Create{Entity}Request` | `CreateCreditCardRequest` |
| Create response | `{Entity}CreateResponse` | `CreditCardCreateResponse` |
| List response | `{Entity}ListResponse` | `CreditCardListResponse` |
| Detail response | `{Entity}DetailResponse` | `CreditCardDetailResponse` |
| Action response | `{Entity}ActionResponse` | `CreditCardActionResponse` |
| Flat entity DTO | `{Entity}Response` | `CreditCardResponse` |

## Validation Messages

- Always provide explicit `message` on validation annotations (in English)
- Messages must be user-friendly and specific about the constraint

## Package Structure

DTOs are organized per module: `dto/creditcard/`, `dto/user/`, `dto/invoice/`, etc.

## Mapping

Response DTOs can have a static `from()` factory method for entity-to-DTO conversion:

```java
public static CategoryListResponse from(Category category) {
    return new CategoryListResponse(category.getId(), category.getName(), category.getColor());
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/clevertondecker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
