## farm-management

> Global API response structure for all endpoints


# API Response Format

All API responses must use this structure:

```json
{
  "status": 1 or 0,
  "message": "success" if status = 1 or "some error message" if status = 0,
  "data": null or the response payload (omit when no data)
}
```

- **status**: `1` for success, `0` for error.
- **message**: `"success"` when status is 1; a clear error message when status is 0.
- **data**: Response payload when returning data; `null` or omitted when there is nothing to return.

Use the `ApiResponse` model and helpers in `app/schemas/response.py` (`success_response`, `error_response`) for consistency.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lakshit77)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lakshit77)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
