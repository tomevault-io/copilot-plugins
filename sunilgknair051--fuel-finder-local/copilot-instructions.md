## fuel-finder-local

> FuelFinder Local is a privacy-first, single-process FastAPI application.

# Repository guidance

FuelFinder Local is a privacy-first, single-process FastAPI application.

- Never commit or log a Tankerkönig API key.
- Never add browser storage, analytics, telemetry, cookies, geolocation, external frontend assets, or automatic polling.
- Keep the browser same-origin and the runtime bound to 127.0.0.1 by default.
- Mock every upstream request in tests.
- Preserve Tankerkönig / MTS-K and GeoNames attribution.
- Use Decimal for currency calculations and monotonic time for limit enforcement.

---
> Source: [sunilgknair051/fuel-finder-local](https://github.com/sunilgknair051/fuel-finder-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
