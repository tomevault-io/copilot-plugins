## refinery-authorizer

> Rules for exception handling and error responses


# Exceptions Guidelines

Handle errors gracefully with appropriate HTTP status codes and clear error messages.

## HTTP Status Codes

**Standard status codes:**
- `200 OK`: Authorization granted
- `400 Bad Request`: Invalid request format or missing required fields
- `403 Forbidden`: Authorization denied
- `500 Internal Server Error`: Unexpected server error

## Error Response Format

**Consistent error structure:**
```python
# Success
{"status": "authorized"}

# Denied
{"status": "not authorized"}

# Error
{"status": "error", "message": "description of error"}
```

## Request Validation

**Check required fields:**
```python
@app.post("/authorize")
def authorize(body: dict, response: Response):
    if "resource" not in body:
        response.status_code = status.HTTP_400_BAD_REQUEST
        return {"status": "error", "message": "resource is required"}
    
    if "subject" not in body:
        response.status_code = status.HTTP_400_BAD_REQUEST
        return {"status": "error", "message": "subject is required"}
```

**Validate nested structure:**
```python
subject = body.get("subject", {})
if "identity" not in subject:
    response.status_code = status.HTTP_400_BAD_REQUEST
    return {"status": "error", "message": "subject.identity is required"}
```

## Safe Data Access

**Handle missing or None values:**
```python
# Use .get() with defaults
metadata = subject.get("metadata_public") or {}

# Check array existence before indexing
if not subject.get("verifiable_addresses"):
    response.status_code = status.HTTP_400_BAD_REQUEST
    return {"status": "error", "message": "verifiable_addresses is required"}

# Safe array access
addresses = subject.get("verifiable_addresses", [])
if addresses and addresses[0].get("verified"):
    # Process
```

## Exception Handling

**Catch and handle exceptions:**
```python
@app.post("/authorize")
def authorize(body: dict, response: Response):
    try:
        # Authorization logic
        if body["resource"] == "kratos:admin":
            return resolve_kratos_admin(body, response)
    except KeyError as e:
        response.status_code = status.HTTP_400_BAD_REQUEST
        return {"status": "error", "message": f"Missing required field: {e}"}
    except (IndexError, AttributeError) as e:
        response.status_code = status.HTTP_400_BAD_REQUEST
        return {"status": "error", "message": f"Invalid data structure: {e}"}
    except Exception as e:
        # Log the full exception for debugging
        import logging
        logging.error(f"Unexpected error: {e}", exc_info=True)
        response.status_code = status.HTTP_500_INTERNAL_SERVER_ERROR
        return {"status": "error", "message": "Internal server error"}
```

## Authorization Denial

**Return 403 for authorization failures:**
```python
def resolve_kratos_admin(body: dict, response: Response):
    subject = body["subject"]["identity"]
    
    if not is_authorized(subject):
        response.status_code = status.HTTP_403_FORBIDDEN
        return {"status": "not authorized"}
    
    response.status_code = status.HTTP_200_OK
    return {"status": "authorized"}
```

## Best Practices

1. Always validate request structure before processing
2. Use appropriate HTTP status codes (400 for bad requests, 403 for denied, 500 for errors)
3. Return consistent error response format
4. Use safe access patterns (`.get()` with defaults) for optional fields
5. Log unexpected exceptions with full traceback for debugging
6. Don't expose internal error details in production responses
7. Handle edge cases (None values, empty arrays, missing keys)
8. Provide clear error messages for debugging

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/code-kern-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
