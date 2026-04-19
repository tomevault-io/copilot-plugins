## agent4socials

> Work from the project map first; use targeted reads and narrow search to save time


# Efficiency: use the map, stay focused

The project has many files. To work quickly and avoid slow, broad searches:

1. **Start from the map**  
   Read `.cursor/PROJECT_MAP.md` at the start of a task (or when the task touches a feature area). Use it to identify the **exact files or directories** that matter for the request. For a shorter API-route listing, see `docs/CODEBASE_MAP.md`.

2. **Prefer targeted reads**  
   Use `read_file` on the specific paths from the map (e.g. `ConnectView.tsx`, `dashboard/inbox/page.tsx`) instead of opening many unrelated files. Read only what you need for the current change.

3. **Narrow search**  
   When you need to search:
   - Use **grep** with a path scoped to the feature (e.g. `path:apps/web/src/app/dashboard/inbox`) instead of repo-wide.
   - Use **codebase_search** only when the map doesn’t cover the task or the change spans multiple areas.

4. **Avoid**  
   - Running broad "find everything about X" across the whole repo when the map already points to 1–3 files.  
   - Reading large files end-to-end when only one section is relevant; use offset/limit or grep first.

5. **Update the map**  
   If you add a new feature or a new important file that isn’t in the map, add a short entry to `.cursor/PROJECT_MAP.md` (and optionally a one-line pointer in `docs/CODEBASE_MAP.md` if you added API routes) so the next run stays focused.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guyk-bot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
