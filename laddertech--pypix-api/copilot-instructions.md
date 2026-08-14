## pypix-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Testing
```bash
# Run mock tests (default)
make test
# Or directly with pytest
pytest tests/tests_mock

# Run integration tests (requires API credentials)
pytest tests/tests_integration
```

### Code Quality
```bash
# Check code style and linting
make lint
# Or directly
ruff check .

# Format code
make format
# Or directly
ruff format .

# Fix linting issues and format
make fix
```

### Building and Publishing
```bash
# Sync dependencies
make sync

# Build package
make build

# Clean previous builds
make clean

# Publish to PyPI (requires credentials)
make publish
```

### Version Management
```bash
# Increment patch version (0.5.0 -> 0.5.1)
make bump-patch

# Increment minor version (0.5.0 -> 0.6.0)
make bump-minor

# Increment major version (0.5.0 -> 1.0.0)
make bump-major
```

## Architecture Overview

This is a Python library for integrating with Brazilian bank APIs, focused on PIX payment system operations.

### Core Components

**Authentication Layer (`pypix_api/auth/`)**
- `oauth2.py`: OAuth2 client. `OAuth2Client(token_url, client_id, cert, pvk, cert_pfx, pwd_pfx, sandbox_mode, client_secret)`. `token_url` is required (use `<Bank>.TOKEN_URL`). Two client-auth flows: mTLS (default, BB/Sicoob) or HTTP Basic via `client_secret` (Sicredi). Token cache per scope.
- `mtls.py`: Mutual TLS support — PEM via `cert`/`pvk` OR PKCS#12 via `cert_pfx`/`pwd_pfx`.

**Bank Integration Layer (`pypix_api/banks/`)**
- `base.py`: Abstract base class `BankPixAPIBase` that aggregates all method mixins. Instantiated with an `OAuth2Client`; subclasses define `BASE_URL`/`TOKEN_URL`/`SCOPES`.
- `bb.py`: Banco do Brasil (`BBPixAPI`, code 001) — adds `PixBBMethods` for BB-specific Pix/refund queries
- `sicoob.py`: Sicoob (`SicoobPixAPI`, code 756)
- `sicredi.py`: Sicredi (`SicrediPixAPI`, code 748) — per-resource API versioning via `_endpoint_url` (`RESOURCE_VERSIONS`), HTTP Basic auth
- `exceptions.py`: Bank-specific exception classes for error handling

**Method Mixins (`pypix_api/banks/methods/`)**
The base class uses multiple inheritance with method mixins for different PIX operations:
- `base_protocol.py`: `PixAPIProtocol` typing the interface the mixins expect
- `cob_methods.py`: Immediate charges (cobranças imediatas)
- `cobv_methods.py`: Charges with due date (cobranças com vencimento)
- `cobr_methods.py`: Recurring charges / Pix Automático (COBR)
- `lotecobv_methods.py`: Batch of due-date charges (lote de CobV)
- `loc_methods.py` / `locrec_methods.py`: Locations (payload/QR) for charges and recurrences
- `pix_methods.py`: PIX transaction operations (consult, refund)
- `pix_bb_methods.py`: BB-specific Pix/refund query variants
- `rec_methods.py`: Recurrence operations
- `solic_rec_methods.py`: Recurrence solicitation methods
- `webhook_methods.py`: Webhook configuration
- `webhook_cobr_methods.py`: Charge (COBR) webhook specific methods
- `webhook_rec_methods.py`: Recurrence webhook specific methods

**Models (`pypix_api/models/`)**
- `pix.py`: `PixCobranca` dataclass. Note: minimal helper — PIX operations send/return raw `dict`, not typed models.

**Scopes (`pypix_api/scopes/`)**
- `registry.py`: OAuth2 scope registry system
- `base.py`, `bb.py`, `sicoob.py`, `sicredi.py`: bank-specific scope definitions

**Observability (optional, top-level modules)**
- `error_handling.py`, `logging.py`, `metrics.py`, `observability.py`: opt-in framework (errors, structured logging, metrics, health checks). Imported conditionally in `__init__.py`. NOT wired into the bank mixins' HTTP calls automatically — it must be used explicitly.

### Key Design Patterns

1. **Inheritance Structure**: Each bank class inherits from `BankPixAPIBase`, which aggregates all method mixins
2. **OAuth2 Dependency**: Banks are instantiated with an OAuth2Client instance, not credentials directly
3. **Method Organization**: PIX operations are organized into logical groups as separate mixins
4. **Error Handling**: Standardized exception hierarchy for consistent error management across banks

### Adding New Bank Support

To add a new bank:
1. Create a new file in `pypix_api/banks/`
2. Inherit from `BankPixAPIBase`
3. Define `BASE_URL` and `TOKEN_URL` class attributes
4. Override bank-specific methods if needed
5. Add scope definitions in `pypix_api/scopes/`

### Testing Strategy

- **Mock Tests** (`tests/tests_mock/`): Unit tests with mocked API responses
- **Integration Tests** (`tests/tests_integration/`): Real API calls requiring credentials

Always run mock tests before commits. Integration tests require proper `.env` configuration with bank credentials.

---
> Source: [laddertech/pypix-api](https://github.com/laddertech/pypix-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
