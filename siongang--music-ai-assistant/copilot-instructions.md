## music-ai-assistant

> Rules for implementing and using ML model providers


# Provider Abstraction Rules

## What is a Provider?

A provider is a concrete implementation of a music processing capability using a specific ML model.
Providers live in `backend/app/providers/{capability_name}/`.

## The Contract

Every provider must:
1. Extend `BaseProvider[InputSchema, OutputSchema]` from `backend/app/providers/base.py`
2. Implement `def run(self, input: InputSchema) -> OutputSchema`
3. Include `ModelMetadata` in its output (model name, version, params used, processing time)
4. Include `confidence` only when the model exposes a meaningful score or a documented heuristic exists; otherwise use `null`
5. Register itself in `backend/app/providers/registry.py` with a unique `provider_key`

## Provider Key Convention

Format: `{model_family}_{variant}` — all lowercase, underscores only.  
Examples: `demucs_htdemucs`, `demucs_mdx23c`, `basic_pitch_v2`

## What Providers May NOT Do

- Import from `app.services.*`
- Import from `app.models.*` (DB models)
- Call the database directly
- Call other providers
- Assume a specific storage layout (accept paths, return paths or data)

## When Adding a New Provider

```python
# backend/app/providers/stems/my_new_model.py

from app.providers.base import BaseProvider
from app.capabilities.stems import StemSeparationInput, StemSeparationOutput

class MyNewModelProvider(BaseProvider[StemSeparationInput, StemSeparationOutput]):
    provider_key = "my_new_model_v1"
    capability = "stem_separation"
    
    def run(self, input: StemSeparationInput) -> StemSeparationOutput:
        # ONLY place where you import the model library
        import my_new_model_lib
        ...
```

Then register in `backend/app/providers/registry.py` — nothing else changes.

## Swapping the Default Provider

Change `DEFAULT_PROVIDERS["stem_separation"]` in `backend/app/core/constants.py`.  
No service, no job handler, no API endpoint needs to change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siongang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
