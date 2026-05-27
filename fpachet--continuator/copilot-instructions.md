## continuator

> Before making architecture-level changes, read:

# Agent Orientation

Before making architecture-level changes, read:

- `docs/current_architecture.md`
- `docs/class_map.md`
- `docs/public_api.md`
- `docs/context_bp_design.md`

The current implementation should be treated as the "classic" Continuator
engine. Preserve the existing public imports unless a migration is explicitly
planned:

```python
from ctor.continuator import Continuator2
from ctor.variable_order_markov import Variable_order_Markov
```

Use the current test suite as the baseline:

```bash
python -m pytest -q
```

---
> Source: [fpachet/continuator](https://github.com/fpachet/continuator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
