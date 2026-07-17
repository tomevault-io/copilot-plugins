## flask-migrate-db-safety

> Stop Flask before flask migrate-db — never migrate while the dev server is running


# Flask migrate-db safety

SQLite (`armillarylab.db` in the project root) must not be migrated while the dev server has the database open.

The dev server uses auto-reload by default. Only the **worker child** opens SQLite; the reloader watcher parent does not (see `config/flask_process.py`).

## Before running `flask migrate-db`

**Always do this first:**

1. Check the terminals folder for an active Flask/dev-server process (`flask run`, `python cli.py`, `app.run`, gunicorn on port 5000/8080).
2. If a server **may be running**, **do not** run `flask migrate-db` yet.
3. **Ask the user** to confirm the Flask server is fully stopped (Ctrl+C in the terminal running it), then wait for confirmation before continuing.
4. Only after confirmation (or verified no server running), run:
   ```powershell
   flask db info
   flask migrate-db    # checks DB health first; does NOT auto-restore
```

If `migrate-db` reports an invalid database, stop Flask and inspect `armillarylab.db`. Manual restore only: `python scripts/restore_db.py` (server still stopped).

## Never

- Run `flask migrate-db` while the dev server is running.
- Copy, restore, or overwrite `armillarylab.db` while the server is running.
- Run `flask init-db --force` or `flask db reset` without explicit user approval.

## If the user asks to migrate

Reply with a short stop-server reminder if you have not verified the server is down, e.g.:

> Please stop the Flask dev server (Ctrl+C in its terminal), then confirm and I'll run `flask migrate-db`.

## Related safe commands (server should still be stopped for file copies)

- `git checkout HEAD -- armillarylab.db` (restore from git)
- Copying `armillarylab.db.backup_*` over `armillarylab.db`

`flask db info` and `flask db backup` are read-only / copy-out — OK while running, but prefer stopping the server for backups too.

---
> Source: [salehram/armillarylab](https://github.com/salehram/armillarylab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
