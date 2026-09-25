## homeassistant-cz-energy-spot-prices

> This Home Assistant custom integration exposes Czech OTE electricity and gas spot prices, currency/unit conversions, buy/sell templates, and cheapest-price windows. Electricity supports hourly and quarter-hour intervals; gas uses daily prices.

# AGENTS.md - AI Coding Agent Guide

## Project and sources of truth

This Home Assistant custom integration exposes Czech OTE electricity and gas spot prices, currency/unit conversions, buy/sell templates, and cheapest-price windows. Electricity supports hourly and quarter-hour intervals; gas uses daily prices.

- Use the Python range and development dependencies in [pyproject.toml](pyproject.toml). Do not duplicate version pins here or install additional checkers implicitly.
- Runtime requirements and the integration release version belong in [manifest.json](custom_components/cz_energy_spot_prices/manifest.json). Preserve semantic versioning and the `X.Y.ZbN` beta convention; change the release version only when the task calls for it.
- Check [the test workflow](.github/workflows/tests.yaml) for CI environment preparation and commands.
- Read the affected implementation and nearby tests before editing. Preserve unrelated work and keep changes scoped to the requested behavior.

## Architecture and implementation references

All integration modules below are under `custom_components/cz_energy_spot_prices/`.

| File | Responsibility and useful examples |
| --- | --- |
| [__init__.py](custom_components/cz_energy_spot_prices/__init__.py) | Setup/unload, shared coordinator consumers, config-entry and entity-ID migration, subentry-to-runtime conversion |
| [coordinator.py](custom_components/cz_energy_spot_prices/coordinator.py) | Shared `SpotRateCoordinator` and `FxCoordinator`, per-entry `EntryCoordinator`, `EntryConfig`, interval/trade data, persistence and retries |
| [spot_rate.py](custom_components/cz_energy_spot_prices/spot_rate.py) | OTE SOAP/XML client, UTC interval parsing, gas placeholder handling |
| [cnb_rate.py](custom_components/cz_energy_spot_prices/cnb_rate.py) | CNB FX client and previous-day rate fallback |
| [cheapest_blocks.py](custom_components/cz_energy_spot_prices/cheapest_blocks.py) | `PriceBlockSearch`, validation, legacy block conversion, window calculations |
| [config_flow.py](custom_components/cz_energy_spot_prices/config_flow.py) | Parent configuration/options and native price-block subentry flows |
| [const.py](custom_components/cz_energy_spot_prices/const.py) | Shared constants and enums |
| [spot_rate_mixin.py](custom_components/cz_energy_spot_prices/spot_rate_mixin.py) | Shared entity updates; the constructor requires `hass`, `coordinator`, `device_id`, and `trade` |
| [sensor.py](custom_components/cz_energy_spot_prices/sensor.py) | Follow `SpotRateElectricitySensor` or `TodayGasSensor` and their existing base classes for price entities |
| [binary_sensor.py](custom_components/cz_energy_spot_prices/binary_sensor.py) | Follow `SearchBasedCheapestElectricitySensor` for search entities; also owns shared tomorrow-data entities |

Data flows from the OTE/CNB clients through shared source coordinators to `EntryCoordinator`, which selects intervals, converts prices, and applies templates. Entities consume coordinator data.

### Async work, schedules, and lifecycle

- Keep HTTP access in API clients invoked by source coordinators. Entity properties and per-entry calculations must not fetch data or block the event loop.
- Reuse Home Assistant's HTTP session via `async_get_clientsession(hass)`; clients must not close an injected session. Preserve standalone client session ownership behavior.
- Preserve existing publication schedules, jitter, capped exponential retries, and CNB fallback behavior. Read scheduling constants and methods before changing them; do not add independent polling loops.
- OTE publication checks use Prague time. FX refreshes use Home Assistant local midnight; this is the integration's refresh schedule, not a claim about CNB publication time.
- Preserve valid cached data on transient failures and the existing persistence format. Missing required FX rates must not produce fabricated converted prices.
- Unloading or reloading one entry must not stop shared coordinators still used by another. Release timers, retry callbacks, and listeners when their owner stops or the final shared consumer unloads. Preserve shared entity ownership during reloads.

## Code conventions and behavior contracts

- Add type hints to new or changed interfaces and follow nearby Home Assistant patterns. Use `typing.override` for overridden methods, `typing.final` for classes intended to be final, and `attr.dataclass` for data classes.
- Reuse the concrete implementations linked above instead of introducing parallel abstractions. Keep type suppressions narrow and explain why they are needed; do not silence a checker globally to accommodate a change.

### Readability and reuse

- Keep entity classes thin. Put parsing, conversion, and window selection in clients, coordinators, or calculation helpers; entities should primarily expose state, attributes, and availability.
- Separate calculations from orchestration. Prefer pure functions for calculations that need no Home Assistant state, passing time and configuration explicitly so behavior is easy to understand and test.
- Give shared business rules one implementation. Extract repeated validation, conversion, or selection rules when their semantics match; similar-looking code alone does not justify an abstraction or a new base class.
- Normalize configuration at the boundary. Convert persisted mappings into validated typed models during setup or configuration changes. Runtime calculations should consume those models instead of repeatedly reading mappings and applying defaults.
- Make units and timezones visible in names where ambiguity matters: prefer `interval_seconds`, `price_per_mwh`, and `start_utc` over `duration`, `value`, or `start`. Preserve externally visible names such as template variables.
- Use guard clauses for missing or invalid inputs and cohesive helpers for distinct responsibilities. Avoid arbitrary function-length limits and helpers that merely rename an obvious expression.
- Write comments that explain reasons and contracts: OTE quirks, DST arithmetic, compatibility constraints, and deliberate exceptions. Avoid comments that restate the next line.
- Keep derived state minimal. Compute values from their source when practical; when caching or maintaining multiple representations is necessary, make update and invalidation paths explicit.
- When touching template rendering/conversion, coordinator retry/persistence, or legacy/native window calculations, inspect related paths for shared rules. Check behavioral differences and preserve compatibility before consolidating them.
- Refactor duplication encountered in the requested change when it clarifies that change; keep broader cleanup separate. These guidelines do not require unrelated refactoring.

### Entity identity and configuration

- Preserve existing unique IDs exactly, including trade/interval suffixes, legacy search IDs, and fixed shared IDs. There is no universal unique-ID formula: derive new IDs from the closest existing entity implementation.
- ID changes require migration in `__init__.py:_migrate_unique_ids()` and entity-registry tests verifying identity survives migration without duplicate entities.
- Price-block searches use native config subentries. Follow `config_flow.py`, `PriceBlockSearch.from_mapping()`, and `__init__.py:_price_block_searches()` when changing their stored or runtime representation.
- Preserve subentry identity during reconfiguration and released legacy config formats through `async_migrate_entry()`. Migration must remain safe to retry and must not duplicate subentries.
- For new options, decide whether they belong to the parent entry/options or a price-block subentry. Update schema validation, setup/runtime conversion, and the relevant model (`EntryConfig` or `PriceBlockSearch`) together.
- Update user-facing strings in all existing translation files (`en.json`, `cs.json`, `sk.json`) and README documentation for user-visible changes.
- Register new entities through the relevant platform setup: sensor factories in `sensor.py`, or `binary_sensor.py:async_setup_entry()` for binary sensors. Preserve subentry association for search entities.

### Time and interval semantics

- Store interval keys and perform elapsed-time arithmetic in timezone-aware UTC. Interpret OTE dates and publication schedules in `Europe/Prague`; group display days using the configured Home Assistant timezone.
- Convert Prague midnight to UTC before adding OTE interval offsets, as the parser does. Indices are one-based: hourly index 1 starts at midnight; quarter-hour index 1 starts the first 15-minute interval.
- Never assume a local day contains 24 hours or 96 quarter-hours. Preserve distinct UTC instants during the repeated autumn hour and handle the missing spring hour.
- Cheapest windows must contain contiguous intervals and satisfy their requested duration. Preserve existing rules for ties, missing tomorrow data, and cross-midnight selection.
- Legacy cross-midnight behavior uses `CONF_ALLOW_CROSS_MIDNIGHT` and `IntervalSpotRateData`; native search validation and calculations also live in `cheapest_blocks.py`. Inspect both paths when changing window behavior.
- For time-related changes, test both DST transitions, midnight boundaries, hourly/quarter-hour intervals, and a Home Assistant timezone other than Prague.

### Prices and templates

- Preserve `Decimal` for price parsing and arithmetic. Construct new decimal constants from strings or integers; avoid introducing float conversions outside existing template compatibility boundaries.
- Divide the FX conversion factor by `Decimal(1000)` only for MWh-to-kWh conversion. MWh values retain the FX factor unchanged; apply conversion exactly once.
- Zero and negative electricity prices are valid. Do not use truthiness to decide whether a price exists. The raw OTE zero-gas placeholder is a commodity-specific exception handled by `is_unpublished_gas_price()`; do not generalize it to electricity or transformed prices.
- Preserve template variables and types. The shared interval path supplies `value` as a float and `hour`/`day` as UTC-aware timestamps, not integer hour/day numbers. The daily helper supplies `value` and `day`; retain compatibility with each path's existing contract.
- Use Home Assistant's `Template.async_render()` in the event-loop context; it is called synchronously in the existing calculation path. Preserve template validation/error handling and test invalid templates or render results when changing that behavior.
- The current template boundary converts prices through float and back to `Decimal`. Treat changes to that boundary as a behavior change requiring compatibility tests, not a mechanical precision cleanup.

## Verification

Run commands from the repository root:

```bash
# Install the declared development environment
uv sync --dev

# Run focused tests (choose files relevant to the change)
uv run pytest tests/test_sensors.py -v

# Run the full suite
uv run pytest

# Type-check the integration with the declared development checker
uv run pyrefly check custom_components/cz_energy_spot_prices

# Optional coverage
uv run pytest --cov=custom_components/cz_energy_spot_prices
```

- For Python changes, run the relevant tests and Pyrefly. Report pre-existing failures separately from regressions; do not claim an unchecked baseline is clean. Pyright configuration remains available for editor use, but do not implicitly install mypy or additional checkers.
- For bug fixes, add a regression test that fails before the fix and passes afterward. For new behavior, assert observable results and relevant edge cases rather than duplicating the implementation in a test.
- Run the full suite for changes to shared coordinators, setup/unload, or migration. Documentation-only edits need reference/format checks, not the Python suite.
- Reuse [tests/conftest.py](tests/conftest.py): `mock_config_entry`, `mock_ote_electricity` (today-only or today+tomorrow), `mock_ote_gas`, and `mock_cnb`. Reuse recorded XML/JSON in `tests/fixtures/` and the existing autospecced async patch patterns.
- Use frozen/controlled time and Home Assistant test helpers for scheduled callbacks. Avoid live network calls and real sleeps.
- Choose tests by behavior: `test_spot_rate.py`/`test_cnb_rate.py` for parsing, `test_update_schedule.py`/`test_fx_coordinator.py` for scheduling, `test_find_cheapest_window.py`/`test_cross_midnight.py` for windows, `test_init.py` for lifecycle/migration, `test_config_flow.py` for configuration, and `test_sensors.py`/`test_gas_sensors.py`/`test_binary_sensors.py` for entities. All are under `tests/`.
- Include relevant cases for zero/negative prices, unavailable FX, missing tomorrow data, invalid templates, DST, and multiple loaded entries when those paths are affected.
- CI intentionally upgrades Home Assistant and its test plugin to the latest compatible stable versions after syncing, then runs `uv run --no-sync pytest -vv`. A normal local `uv run pytest` uses the resolved project environment and may differ. When reproducing CI, follow the workflow and retain `--no-sync` after its explicit upgrade so the lockfile does not replace that environment.
- In the handoff, summarize the behavior changed, checks run and their results, and any checks blocked or omitted.

## Debugging

Enable `custom_components.cz_energy_spot_prices: debug` in the Home Assistant logger configuration. Inspect coordinator schedules, availability, and API parsing logs alongside the relevant recorded fixtures.

---
> Source: [rnovacek/homeassistant_cz_energy_spot_prices](https://github.com/rnovacek/homeassistant_cz_energy_spot_prices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
