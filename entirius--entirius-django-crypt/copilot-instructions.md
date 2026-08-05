## entirius-django-crypt

> Symmetric encryption helper (Fernet) for Volkanos Django modules — distribution `entirius-django-crypt`, Django app `django_crypt`.

# AGENTS.md

Symmetric encryption helper (Fernet) for Volkanos Django modules — distribution `entirius-django-crypt`, Django app `django_crypt`.

## Commands

| Command | Meaning |
|---|---|
| `make install` | sync dependencies (uv, incl. extras) |
| `make check` | lint + format-check (ruff) |
| `make fix` | auto-fix lint + format |
| `make test` | test suite (pytest + pytest-django) |

## Conventions

- English only: code, docs, commits, branches, PRs.
- MPL-2.0: every non-trivial source file carries the license header (pre-commit inserts it).
- Toolchain: uv + ruff + hatchling + pytest; all config in `pyproject.toml`; `uv.lock` committed.
- Git flow: `master` (production) + `develop` (integration); changes land via PR; semver tag on `master`.
- Never rename the package / Django app_label / DB table prefix `django_crypt` — it is a schema contract.
- Migrations are part of the public contract — never edit an already released migration.
- Default: do not commit — git is the user's call.

## Architecture

```
src/django_crypt/
├── __init__.py             # Crypt class (encrypt/decrypt/generate_salt)
├── apps.py                 # DjangoCryptConfig (is_volkanos=True)
├── settings.py             # CRYPT_SALT (loaded from django.conf.settings)
└── management/commands/
    ├── encrypt.py          # ./manage.py encrypt "<text>"
    ├── decrypt.py          # ./manage.py decrypt "<ciphertext>"
    └── generate_salt.py    # ./manage.py generate_salt
```

No models, no migrations — pure helper + management commands.

## Public API

```python
from django_crypt import Crypt

class Crypt:
    salt: str

    def __init__(self, salt: str | None = None)
        # If salt is None, loads from settings.CRYPT_SALT
        # Raises ValueError if both salt and CRYPT_SALT are None

    @staticmethod
    def generate_salt() -> str
        # Returns Fernet-compatible base64 key (32 bytes encoded)

    def encrypt(self, text: str | None) -> str | None
        # Returns Fernet ciphertext (non-deterministic)
        # Returns None if input is None

    def decrypt(self, text: str | None) -> str | None
        # Returns plaintext from Fernet ciphertext
        # Returns None if input is None
        # Raises cryptography.fernet.InvalidToken on bad input
```

## Settings

| Setting | Default | Purpose |
|---------|---------|---------|
| `CRYPT_SALT` | None | Fernet key (base64). Generate via `python manage.py generate_salt` |

## Gotchas

- **Non-deterministic ciphertext** — `encrypt(x)` and `encrypt(x)` produce **different** ciphertexts.
  Don't use `Crypt.encrypt` for lookup-key hashing. Use HMAC-SHA256 (or similar deterministic hash)
  for unique-indexed lookup columns.
- **CRYPT_SALT must be Fernet-compatible** — 32 url-safe base64-encoded bytes. Generate via
  `Crypt.generate_salt()` or `Fernet.generate_key()`. Random strings will fail at runtime.
- **Salt rotation requires re-encrypt** — if you change `CRYPT_SALT`, all existing ciphertexts
  become undecryptable. Plan migration: dual-key window with `Crypt(salt=old)` for read +
  `Crypt(salt=new)` for write, then bulk re-encrypt.
- **Errors leak structure** — `decrypt()` on garbage raises `cryptography.fernet.InvalidToken`.
  Don't surface raw exception in HTTP responses (info leak).

## References

- [Fernet docs](https://cryptography.io/en/latest/fernet/)

---
> Source: [entirius/entirius-django-crypt](https://github.com/entirius/entirius-django-crypt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
