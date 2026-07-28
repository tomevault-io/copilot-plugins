## beacon-live-admin

> Rules for Elixir, Phoenix, and Phoenix LiveView

Guidelines:
- Consult latest Phoenix documentation at @https://hexdocs.pm/phoenix
- Consult latest Phoenix LiveView documentation at @https://hexdocs.pm/phoenix_live_view
- Do not add a new `alias` unless that module is referenced more than once in the module, prefer the fully-qualified name if module is referenced just once
- Do not group alias, put one per line.
- Try to place new function into existing modules related to the task in hand, instead of creating new files and modules
- Use components from @lib/beacon/live_admin/components to write HEEx templates
- Add `@spec` when creating new functions but do not change existing ones
- Add `@doc` when creating new functions but do not change existing ones
- Do not overwrite `@doc false`
- Do not overwrite `@moduledoc false`
- Do not include `@doc` nor `@spec` to Phoenix.LiveView callbacks like mount/3, render/1, handle_*, 
- Do not include `@doc` nor `@spec` to Phoenix.LiveComponent callbacks like mount/1, update/2

---
> Source: [BeaconCMS/beacon_live_admin](https://github.com/BeaconCMS/beacon_live_admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
