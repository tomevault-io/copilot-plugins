## origin-mcp

> - Treat the installed Origin Start/Stop App as a generated artifact. When a

# Development and debugging rules

## Origin Start/Stop App

- Treat the installed Origin Start/Stop App as a generated artifact. When a
  change affects `addon.py`, the App launcher/installer/build scripts, or any
  `origin_mcp` code executed by the Origin-side bridge, do not test against a
  previously installed App.
- Before the real-Origin verification, stop the running bridge and rebuild and
  overwrite the local App installation:

  ```powershell
  python scripts\build_origin_app.py --force --install
  ```

- Start **Origin MCP Bridge Start** again after installation, then verify the
  live connection with `origin-mcp status` and
  `origin-mcp doctor --ping-origin`.
- Repacking and reinstalling both OPX files is required when testing App
  packaging, metadata, registration, or distributable installation. For normal
  local source debugging of an already registered App, overwriting the App
  folders with `--install` is sufficient.

---
> Source: [Ge-Shun/origin-mcp](https://github.com/Ge-Shun/origin-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
