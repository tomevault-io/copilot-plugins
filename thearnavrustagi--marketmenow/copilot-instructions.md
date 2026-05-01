## adapters

> Conventions for platform adapter packages


# Adapter Development Rules

## Structure

Every adapter package follows this layout:

```
adapters/yourplatform/
├── __init__.py      # create_yourplatform_bundle(settings) -> PlatformBundle
├── adapter.py       # PlatformAdapter implementation
├── renderer.py      # ContentRenderer implementation
├── uploader.py      # Uploader implementation
├── settings.py      # pydantic BaseSettings for env vars
└── cli.py           # Typer sub-commands (optional)
```

## Protocol Compliance

- Implement protocols from `marketmenow.ports` via structural subtyping.
  Never subclass or inherit from the Protocol class.
- Required: `PlatformAdapter`, `ContentRenderer`, `Uploader`.
- Optional: `AnalyticsCollector`.
- All adapter methods are `async def`.
- `platform_name` is a `@property` returning a lowercase string.

## Bundle Factory

Expose `create_yourplatform_bundle(settings)` in `__init__.py`.
Construct a `PlatformBundle(adapter=..., renderer=..., uploader=...)`.

## Registration

Add a `_try_yourplatform()` function in `marketmenow/core/registry_builder.py`.
Use lazy imports inside a try/except so missing credentials cause graceful skip.

## Isolation

- Never import from other adapter packages.
- Never import adapter code in `src/marketmenow/` (except `core/registry_builder.py`).
- Settings use `pydantic.BaseSettings` or `pydantic_settings.BaseSettings`
  reading credentials from environment variables.

## Browser Automation

Adapters that post via browser (Twitter, Reddit, Facebook, LinkedIn) use Playwright
with chromium. Store browser state files in project root (gitignored).

## CLI Integration

Create a Typer app in `cli.py` and register it in `marketmenow/cli.py` via
`app.add_typer(yourplatform_app, name="yourplatform", hidden=True)`.
Adapter CLIs are hidden from `mmn --help` — the user-facing CLI uses
`mmn run <workflow>`. Hidden commands remain callable by the web frontend.

---
> Source: [thearnavrustagi/marketmenow](https://github.com/thearnavrustagi/marketmenow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
