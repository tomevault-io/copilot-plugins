## bluepilot

> BluePilot codebase architecture, conventions, and safety rules


# BluePilot Codebase Rules

Read and follow the full documentation in `AGENTS.md` at the repo root before making changes.

## Critical Architecture Rule

BluePilot uses a three-layer inheritance architecture. Minimize changes to upstream files.

```
Layer 1: Stock OpenPilot  → selfdrive/ui/onroad/, system/, common/   (MINIMIZE CHANGES)
Layer 2: SunnyPilot       → sunnypilot/, selfdrive/ui/sunnypilot/    (MINIMIZE CHANGES)
Layer 3: BluePilot        → bluepilot/, selfdrive/ui/bp/             (OUR CODE)
```

When you must change upstream files (e.g., to wire in BP classes), wrap changes in comments:
```python
# BluePilot: import BP alert renderer override
from openpilot.selfdrive.ui.bp.onroad.alert_renderer_bp import AlertRendererBP
# End BluePilot
```
This makes upstream merges easy — search `grep -r "BluePilot:"` to find all touchpoints.

## Naming Conventions

- BP files: `_bp` suffix (e.g., `alert_renderer_bp.py`)
- BP classes: `BP` suffix (e.g., `AlertRendererBP`)
- BP params: `FordPref*`, `BP*`, `Show*` prefixes
- Always inherit from stock/SP parent classes
- Always call `super().__init__()` and `super()._render()` in overrides

## Inheritance Chain

```
AlertRenderer (stock) → AlertRendererBP (BluePilot)
HudRenderer (stock) → HudRendererSP (SP) → HudRendererBP (BluePilot)
ModelRenderer (stock) → ModelRendererBP (BluePilot)
AugmentedRoadView (stock) → AugmentedRoadViewBP (BluePilot)
```

Wiring via conditional imports in `selfdrive/ui/layouts/main.py`.

## Key Directories

- `bluepilot/` - Portal, params, logger, UI widgets
- `selfdrive/ui/bp/onroad/` - BP onroad renderers (TICI)
- `selfdrive/ui/bp/mici/onroad/` - BP onroad renderers (MICI)
- `selfdrive/ui/bp/layouts/settings/bluepilot.py` - Settings menu
- `opendbc_repo/opendbc/car/ford/carcontroller.py` - Ford control logic
- `bluepilot/params/params.json` - Parameter definitions
- `system/manager/process_config.py` - Process registry

## Safety Rules

- NEVER allow gas and brake simultaneously
- Rate-limit all control signals
- Portal blocks modifications when driving (403)
- Use Params for feature flags, not compile-time flags
- New processes added via `procs +=`, never replacing stock

## For Full Details

See `AGENTS.md` in the repo root for complete documentation including:
- Ford 4-signal lateral control system
- Longitudinal control with lead-aware gas limiting
- BluePilot Portal API endpoints
- Parameter system with 60+ definitions
- Messaging system (Cereal/Cap'n Proto)
- Build system and testing
- Quick reference guide for common tasks

---
> Source: [BluePilotDev/bluepilot](https://github.com/BluePilotDev/bluepilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
