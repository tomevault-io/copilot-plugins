## zalmanimai

> Keep WordPress plugin distributable as ZIP


# WordPress plugin – always ship as ZIP

- The Zalmanim Demo Addon lives in `apps/wordpress-plugin/zalmanim-demo-addon/` (source).
- **After any change** to files in `zalmanim-demo-addon/`, regenerate the distributable ZIP:
  - Path: `apps/wordpress-plugin/zalmanim-demo-addon.zip`
  - Content: the **folder** `zalmanim-demo-addon` (so the ZIP has one root folder, as WordPress expects).
- PowerShell from repo root:
  `Compress-Archive -Path "apps/wordpress-plugin/zalmanim-demo-addon" -DestinationPath "apps/wordpress-plugin/zalmanim-demo-addon.zip" -Force`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zalmanimrecords-stack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
