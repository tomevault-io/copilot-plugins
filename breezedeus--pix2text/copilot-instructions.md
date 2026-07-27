## pix2text

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Code Style Guidelines

### Logging

- Use lazy logging with `%s` format placeholders instead of f-strings
- This avoids unnecessary string formatting when the log level is disabled

```python
# Good
logger.info("Processing case: %s", case_id)
logger.debug("Result: %s, time: %.2fs", result, time_cost)

# Bad
logger.info(f"Processing case: {case_id}")
logger.debug(f"Result: {result}, time: {time_cost:.2f}s")
```

### Imports

- All imports must be placed at the top of the file
- Do not place imports in the middle of the code

```python
# Good
from __future__ import annotations

import json
import logging
from typing import Any, Dict, Optional

from .utils import load_env


def my_function():
    pass

# Bad
def my_function():
    from .utils import load_env  # Don't do this
    pass
```

---
> Source: [breezedeus/Pix2Text](https://github.com/breezedeus/Pix2Text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
