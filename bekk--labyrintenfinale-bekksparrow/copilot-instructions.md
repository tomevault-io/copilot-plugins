## labyrintenfinale-bekksparrow

> Explains that you need to restart the app after having changed the routes

# restart-on-route-change

Whenever you make changes to the route configuration (such as adding, removing, or renaming routes), you must restart the app for the changes to take effect.

- Run `npm run dev` from the project root to restart both the frontend and backend.
- This is necessary because the backend and router are auto-run together, and route changes are not picked up by hot reload.

**Purpose:**
This ensures that any updates to the routing system are properly recognized by the app during development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bekk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
