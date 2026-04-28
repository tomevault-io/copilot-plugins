## twitter-trends

> - Use the project `logger = get_logger()` helper; never call `print()` except in CLI scripts.

- Use the project `logger = get_logger()` helper; never call `print()` except in CLI scripts.  
- Wrap each major block in:
  ```python
  start = time.time()
  try:
      …
  finally:
      logger.info("X completed in %.1fs", time.time() - start)
On any unhandled exception, call metrics.increment("pipeline_errors") then re-raise.

Emit a gauge metrics.set("last_run_timestamp", now_ts) at job start.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Paramfere) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
