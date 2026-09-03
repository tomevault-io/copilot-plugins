## sae-delivery-integrations

> High-signal context for agents working in this Odoo 18 carrier integration codebase.

# Carrier Integrations — Agent Hints

High-signal context for agents working in this Odoo 18 carrier integration codebase.

## Architecture

- **transport_booking_core** — Carrier-agnostic per-leg booking framework
- **apc_uk_shipping** — APC Overnight adapter (depends on core)
- **dpd_local_uk_shipping** — DPD Local adapter (depends on core)

## Adapter Register Pattern

Adapters extend `TransportBookingAdapter` and register via decorator:

```python
from odoo.addons.transport_booking_core.booking import (
    TransportBookingAdapter, register_adapter, BookingResult, TransportBookingError,
)

@register_adapter
class CarrierAdapter(TransportBookingAdapter):
    provider_code = "carrier_name"  # matches delivery.carrier.transport_provider
```

## Package Dimensions / Weight Source Priority

When building payload items, read in this order:

1. **sale.package.line** (set on SO) — `length/width/height/weight` in mm
2. **stock.quant.package** — dimensions from `package_type_id.packaging_length/width/height`
3. **package_type_id.base_weight** — fallback weight if package has none
4. **picking.weight** — last resort (dimensions = 0)

## APC Label Structure

APC v3 label response nests the base64 label inside each Item:
```
Orders[].Order.ShipmentDetails.Items.Item[].Label.Content
```
Not in a separate `Labels` block. The `_apc_extract_label` helper walks this structure.

## Key Gotchas

- **Odoo field names differ**: `stock.package.type` uses `packaging_length` not `length`
- **Units**: Odoo stores dimensions in mm; APC/DPD expect cm — divide by 10
- **Emails/Mobile**: Added to APC payload via `Email`/`MobileNumber` in Contact block; fallback from `to_location`/`from_location` partner
- **Transport legs required**: Sales orders need transport legs for API booking
- **MCP writes disabled**: Changes to carriers/packages must be made in Odoo UI or via server

## Deploy

- Python-only changes: restart Odoo server
- View/XML changes: upgrade module (`-u <module>` or Apps → Upgrade)
- Cloudpepper: upgrade via Apps button (no SSH needed)

## Reference

See `docs/NEW_CARRIER_ADAPTER_HANDOFF.md` for full adapter build guide.

---
> Source: [eartisan-uk/sae-delivery-integrations](https://github.com/eartisan-uk/sae-delivery-integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
