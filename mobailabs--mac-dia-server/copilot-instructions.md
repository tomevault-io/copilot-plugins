## auth-config

> - All API endpoints require an API key via the `Authorization: Bearer <API_KEY>` header.

# Authentication & Configuration

## API Key Authentication
- All API endpoints require an API key via the `Authorization: Bearer <API_KEY>` header.
- The key is validated in [src/security.py](mdc:src/security.py) using FastAPI dependencies.
- The expected key is loaded from the `.env` file in the project root (not in version control).

## Environment Configuration
- `.env`: Should contain `API_KEY=your_actual_api_key`.
- [src/security.py](mdc:src/security.py): Loads and checks the API key for each request.

## Example Usage
```
curl -X POST http://localhost:8000/v1/audio/speech \
  -H "Authorization: Bearer YOUR_API_KEY" ...
```

---
> Source: [mobailabs/mac-dia-server](https://github.com/mobailabs/mac-dia-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
