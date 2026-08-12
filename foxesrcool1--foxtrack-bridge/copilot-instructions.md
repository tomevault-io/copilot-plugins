## foxtrack-bridge

> These rules exist because v2.2.0 shipped a config-wipe hazard: POST `/api/config`

# FoxTrack Bridge — engineering invariants

## Config compatibility (non-negotiable)

These rules exist because v2.2.0 shipped a config-wipe hazard: POST `/api/config`
replaced the whole config wholesale, so a Settings save (or any save) made while
the dashboard's local printer list was empty — e.g. after a failed `/api/config`
load — silently overwrote `config.json` with an empty `printers` array. The guard
in `resolveConfigUpdate` (server.go) and the partial Settings payload in
`web/ui.html` are the fix; do not regress them.

1. **Never overwrite a non-empty config with an empty one.** A request that would
   replace a non-empty `printers` list with an empty one must be refused unless it
   carries an explicit `confirm_clear_printers: true`. A body that omits the
   `printers` key is a partial update and must leave existing printers untouched.
2. **Config schema changes must be additive.** New fields are optional and
   `omitempty`; never rename or remove existing `config.json` fields. Existing
   configs must keep loading with no migration and no re-adding of printers.
3. **Corrupt-config handling must preserve the original file.** On a load/parse
   failure, back the file up (e.g. `config.json.corrupt-<timestamp>`) — never
   delete or truncate it — before starting with an empty config.
4. **Updates must never require users to re-add printers.** Any change to config
   location or format must fall back to and migrate prior locations without data
   loss.

## Scope guardrails

- Do not modify telemetry parsing, webhook/history/snapshot senders, or
  `bridge_commands` logic without explicit sign-off.
- Bambu (MQTT) and Klipper (Moonraker/LAN) paths must both keep working.
- The binary is not writable inside Docker; self-update cannot run there.

## Release checklist

Before pushing any release tag:

1. Build the binary locally and run it against a real config with saved printers.
2. Confirm all printers render in the dashboard on first load.
3. Save a setting, then confirm the printer list survives on disk and in the UI.
4. Verify `/tailwind.css`, `/icons.css`, `/fonts.css`, and
   `/fonts/inter-latin.woff2` return 200.
5. Only then merge, tag, and push.

---
> Source: [FoxesRCool1/FoxTrack-Bridge](https://github.com/FoxesRCool1/FoxTrack-Bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
