## opendxp

> This is the **opendxp core** library. It provides models, events, base controllers, caching, routing, and infrastructure used by all OpenDXP bundles.

# OpenDXP Core — Claude Instructions

## Repository Role

This is the **opendxp core** library. It provides models, events, base controllers, caching, routing, and infrastructure used by all OpenDXP bundles.

**Related bundles:** Bundles that depend on this core (e.g. `admin-bundle`) may live in different locations depending on the developer's setup. When you need to read bundle source files, check these paths in order and use the first one that exists:

1. `../admin-bundle/` — sibling directory (monorepo / symlinked workspace)
2. `../../vendor/open-dxp/admin-bundle/` — project root one level up
3. `../../../vendor/open-dxp/admin-bundle/` — project root two levels up

## Admin-Related Topics

Admin UI features, admin events, and admin extension points are owned and documented by **admin-bundle** (`open-dxp/admin-bundle`).

If you are looking to:
- Subscribe to admin events → admin-bundle `docs/10_Extension_Points/01_Events.md`
- Extend the admin UI with JS/CSS → admin-bundle `docs/10_Extension_Points/02_Admin_UI_Assets.md`
- Add JavaScript UI extensions (menus, key bindings) → admin-bundle `docs/10_Extension_Points/03_Admin_UI_JavaScript.md`
- Configure perspectives → admin-bundle `docs/10_Extension_Points/04_Perspectives.md`
- Add custom permissions → admin-bundle `docs/10_Extension_Points/05_Permissions.md`
- Use deeplinks into admin → admin-bundle `docs/10_Extension_Points/06_Deeplinks.md`
- Configure a custom admin login entry point → admin-bundle `docs/10_Extension_Points/07_Custom_Admin_Login.md`
- Customize the site settings panel → admin-bundle `docs/20_Documents/01_Site_Custom_Settings.md`

Find admin-bundle on disk via `composer show open-dxp/admin-bundle | grep path`,
or browse on GitHub: https://github.com/open-dxp/admin-bundle/blob/1.x/docs/

If an existing file in `doc/` only contains a redirect notice, follow the link to admin-bundle.

## Documentation

Core documentation lives in `doc/` using the `NN_TopicName/` folder convention.
Each folder has a `README.md` as its section index.

**doc/ structure overview:**
```
doc/
├── 01_Getting_Started/
├── 02_MVC/
├── 03_Documents/
├── 04_Assets/
├── 05_Objects/
├── 06_Multi_Language_i18n/
├── 07_Workflow_Management/
├── 18_Tools_and_Features/
├── 19_Development_Tools_and_Details/
├── 20_Extending_OpenDxp/
├── 21_Deployment/
├── 22_Administration_of_OpenDxp/
├── 23_Installation_and_Upgrade/
└── 26_Best_Practice/
```

If a file in `doc/` only contains a redirect notice, the content lives in admin-bundle — follow the link there.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/open-dxp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/open-dxp)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
