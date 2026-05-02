## infra-tak

> `nodered/build-flows.js` has a `FEEDS` array that becomes **static** ArcGIS engine tabs in the generated `flows.json`. **Keep `FEEDS` empty.** Real feeds are created via the ArcGIS Configurator (dynamic tabs preserved on deploy). Never commit production feed names (e.g. CA AIR INTEL, POWER-OUTAGES) as static tabs — every customer would inherit them.

# infra-TAK — Cursor rules

## Node-RED: do not ship named engine tabs in `flows.json`
`nodered/build-flows.js` has a `FEEDS` array that becomes **static** ArcGIS engine tabs in the generated `flows.json`. **Keep `FEEDS` empty.** Real feeds are created via the ArcGIS Configurator (dynamic tabs preserved on deploy). Never commit production feed names (e.g. CA AIR INTEL, POWER-OUTAGES) as static tabs — every customer would inherit them.

## Node-RED: enterprise operator flow (v0.6.2+)
After **Update Now**: shipped flows have **no** example feeds. Operators use **Configurator** for ArcGIS/TFR, then **Node-RED editor** → **TAK Mission API TLS** → admin cert paths under **`/certs/`** (host mount) + **key passphrase** → **Deploy**. Console deploy must mount `/opt/tak/certs/files` and run `nodered/deploy.sh` — see `docs/RELEASE-v0.6.2-alpha.md`.

## Reuse what works first
When adding a variant (e.g. remote deploy, new target, new environment): **try what we know works in the existing case (e.g. local), then tweak as necessary.** Do not invent a different flow, script, or timing until the known-good approach has been tried and only the minimal changes (URLs, paths, where commands run) applied.

---
> Source: [takwerx/infra-TAK](https://github.com/takwerx/infra-TAK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
