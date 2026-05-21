## code-tests

> When making tests please follow the below


# Tests
When making tests please follow the below
- `asyncio_mode = auto` is set in pytest.ini so when making tests don't mark tests with pytest.mark.asyncio
- When testing if errors are thrown (`with pytest.raises(...)`) do not try to match exception text to some predefined value

---
> Source: [Metaculus/forecasting-tools](https://github.com/Metaculus/forecasting-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
