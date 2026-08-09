## platypush

> - Run the full test suite:

# Agent Notes

## Verification Commands

- Run the full test suite:

  ```bash
  python -m pytest
  ```

- Run specific tests:

  ```bash
  python -m pytest tests/test_plugin_icon.py tests/test_pwa.py -v
  ```

- Run linting:

  ```bash
  python -m flake8 platypush tests
  ```

- Run formatting checks:

  ```bash
  python -m black --check .
  ```

- Apply automatic formatting:

  ```bash
  python -m black .
  ```

## Notes

- `pyproject.toml` configures `skip-string-normalization = true` for Black.
- `setup.cfg` configures `flake8` with a max line length of 120 and ignores
  `E203`, `W503`, `SIM104`, `SIM105`, and `SIM115`.

---
> Source: [blacklight/platypush](https://github.com/blacklight/platypush) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
