## hyprland-dock

> This is a Hyprland application dock implemented with Quickshell and Qt/QML. It runs either as a standalone Quickshell configuration or as an Omarchy Quattro overlay plugin hosted by the existing Omarchy shell.

# AGENTS.md

## Project overview

This is a Hyprland application dock implemented with Quickshell and Qt/QML. It runs either as a standalone Quickshell configuration or as an Omarchy Quattro overlay plugin hosted by the existing Omarchy shell.

## Development

- Run the dock with `./scripts/run`.
- Keep the standalone entry point at `shell.qml` and the Omarchy plugin entry point at `Overlay.qml`.
- Keep shared host behavior in `DockHost.qml`; never start a second Quickshell process from the plugin.
- Put reusable visual components in `components/`.
- Keep user-facing defaults in `config/dock.json` and mirror fallback defaults in `shell.qml`.
- Installed user settings live outside the application at `~/.config/hyprland-dock/dock.json`; updates must never overwrite them.
- Keep `install.sh`, `uninstall.sh`, and `scripts/hyprland-dock` compatible with custom XDG directory variables.
- Prefer Quickshell APIs over shelling out to external commands.
- Use freedesktop desktop-entry IDs without the `.desktop` suffix.
- Preserve live configuration reloads.

## Validation

Before committing changes:

```bash
timeout 6s ./scripts/run --no-color
bash -n install.sh uninstall.sh scripts/hyprland-dock scripts/run
omarchy plugin validate .
/usr/lib/qt6/bin/qmllint -I "$OMARCHY_PATH/shell" \
  Overlay.qml DockHost.qml components/Dock.qml components/DockItem.qml shell.qml
git diff --check
```

A portal warning about an application ID already being registered can occur when another Quickshell process is running; it is not a dock failure.

## Style

- Use two-space indentation in QML.
- Keep JavaScript helpers small and local to the component that owns the behavior.
- Add new configuration options to both `config/dock.json` and the README.

---
> Source: [nick-friedrich/hyprland-dock](https://github.com/nick-friedrich/hyprland-dock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
