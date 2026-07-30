## python-imports

> Python imports must be at the top of the file


# Python Imports

All imports MUST be placed at the top of the file, following standard Python conventions (stdlib, third-party, local). Do NOT place imports inside functions, methods, or conditional blocks.

The only exception is when a top-level import would cause a **circular dependency**. In that case, add a brief comment explaining why.

```python
# ✅ Correct — imports at the top
from datetime import UTC, datetime

from kubernetes.client import CoreV1Api

from nv_config_manager_installer.schema import NVConfigManagerInstallConfig


def do_something():
    v1 = CoreV1Api()
    ...

# ❌ Wrong — import buried inside a function
def do_something():
    from kubernetes.client import CoreV1Api
    v1 = CoreV1Api()
    ...

# ✅ Exception — circular dependency with comment
def get_app():
    from nv_config_manager_installer.tui.app import NVConfigManagerApp  # avoid circular import
    return NVConfigManagerApp()
```

---
> Source: [NVIDIA/nv-config-manager](https://github.com/NVIDIA/nv-config-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
