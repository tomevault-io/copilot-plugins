## oxygenos-ota-remover

> This is a KernelSU (KSU) and Magisk compatible module that removes OTA (Over-The-Air) update components from OxygenOS 16. The primary goal is to prevent forced system updates on OnePlus devices running OxygenOS 16 / Android 16.

# OxygenOS 16 OTA Remover - KernelSU/Magisk Module

## Project Overview

This is a KernelSU (KSU) and Magisk compatible module that removes OTA (Over-The-Air) update components from OxygenOS 16. The primary goal is to prevent forced system updates on OnePlus devices running OxygenOS 16 / Android 16.

**Target device:** OnePlus 15 (Android 16, OxygenOS 16)
**Compatibility:** Generic OxygenOS 16 devices

## Project Info

- **License:** CC BY-NC-SA 4.0 (no commercial use)
- **Owner:** badger
- **Repo:** GitHub (public)

## Module Structure

```
.
├── META-INF/
│   └── com/google/android/
│       ├── update-binary            # Magisk installer (KSU ignores this)
│       └── updater-script           # Dummy (#MAGISK)
├── module.prop                      # Module metadata (REQUIRED)
├── customize.sh                     # Install logic (KSU vs Magisk)
├── service.sh                       # Post-boot fallback (pm disable-user)
├── uninstall.sh                     # Re-enables packages on removal
├── build.sh                         # Build script (creates zip in build/)
├── CHANGELOG.md                     # Semver changelog (MUST be updated every release)
├── LICENSE                          # CC BY-NC-SA 4.0
├── CLAUDE.md
└── .gitignore
```

Note: No static `system/` overlay directory — `customize.sh` creates the overlay dirs dynamically at install time for Magisk. KSU uses the `REMOVE` variable.

## OxygenOS 16 OTA Packages (OnePlus 15)

Verified paths from device:

| Package | APK Path | Description |
|---------|----------|-------------|
| `com.oplus.ota` | `/system_ext/app/OTA/OTA.apk` | Main OTA updater |
| `com.oplus.cota` | `/product/priv-app/Cota/Cota.apk` | Component OTA (silent updates) |
| `com.cota.notification` | `/product/priv-app/NOTIFICATION/NOTIFICATION.apk` | COTA notification service |
| `com.oplus.romupdate` | `/product/app/RomUpdate/RomUpdate.apk` | ROM update checker |
| (no package) | `/system_ext/app/SystemAppUpdateService/` | System app update service |

**Not targeted:**
- `com.updateme` — user-installed app in `/data/app/`, uninstall normally
- `com.google.android.configupdater` — Google config updater, not OTA
- `update_engine`, `otapreopt` — low-level binaries, removing breaks system

## Key Concepts

### Dual Compatibility (KSU + Magisk)

- **Magisk** uses `.replace` files inside overlay dirs to remove system apps
- **KernelSU** does NOT support `.replace` — uses `REMOVE` variable in `customize.sh`
- **APatch** also supports `REMOVE` variable (same as KSU)
- Detect environment: `$KSU` is `true` on KernelSU, `$APATCH` is `true` on APatch

### Overlay Paths

Apps live on different partitions — the module overlays must mirror the actual mount points:
- `/system_ext/app/OTA/` → `$MODPATH/system_ext/app/OTA/.replace`
- `/product/priv-app/Cota/` → `$MODPATH/product/priv-app/Cota/.replace`
- `/product/app/RomUpdate/` → `$MODPATH/product/app/RomUpdate/.replace`

### Two-Layer Removal Strategy

1. **Overlay** (primary): Hides the APK directories so the apps don't exist at boot
2. **service.sh** (fallback): Runs `pm disable-user --user 0` after boot as belt-and-suspenders

## Development Guidelines

### Shell Scripts
- Use `#!/system/bin/sh` shebang (NOT `#!/bin/bash`)
- Both KSU and Magisk provide BusyBox ash shell — keep scripts POSIX-compatible
- Use `ui_print` for user-facing output during installation
- Available env vars in `customize.sh`: `$MODPATH`, `$TMPDIR`, `$ZIPFILE`, `$ARCH`, `$API`, `$KSU`

### Release Process (MUST follow for every release)

1. **Update `CHANGELOG.md`** — add a new `## [x.y.z] - YYYY-MM-DD` section with all changes
2. **Bump version** in `module.prop` — update both `version=` and `versionCode=`
3. **Build**: `./build.sh` — outputs `build/oxygenos16-ota-remover-vX.Y.Z.zip`
4. **Commit & tag**: `git tag vX.Y.Z`
5. **Push**: `git push origin main --tags`
6. **GitHub Release**: create release from the tag, attach the zip from `build/`

Semver rules:
- **PATCH** (1.0.x): bug fixes, minor script tweaks
- **MINOR** (1.x.0): new packages targeted, new features
- **MAJOR** (x.0.0): breaking changes, major restructure

### Building & Testing
- Build: `./build.sh` (outputs to `build/` directory)
- Install: push zip to device, install via KSU/Magisk Manager
- Verify after reboot: `pm list packages -d | grep -iE "ota|cota|romupdate"` (should show disabled)
- Check module active: `ls /data/adb/modules/oxygenos16-ota-remover/`

### Discovering Packages on Other Devices
```bash
pm list packages | grep -iE "ota|update|fota"
pm path <package_name>
find /system /system_ext /product /vendor -iname "*ota*" -o -iname "*update*" 2>/dev/null
```

### Important Caveats
- Do NOT modify `/system` directly — use the systemless overlay
- `service.sh` provides a fallback since OxygenOS may re-register OTA services
- `uninstall.sh` re-enables all packages when the module is removed
- Paths may vary between OxygenOS builds and device variants

---
> Source: [crankshift/oxygenos-ota-remover](https://github.com/crankshift/oxygenos-ota-remover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
