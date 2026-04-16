## build-nix-graphene

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository builds custom GrapheneOS images using Nix via the robotnix framework. It contains:
- `robotnix/` - Git submodule containing the robotnix build system
- `custom-build/` - Custom GrapheneOS configuration with Firefox (Fenix) preinstalled

## Build Commands

Build the image (from `custom-build/` directory):
```bash
nix build .#img                    # Bootable image zip for fastboot update
nix build .#factoryImg             # Factory image with radio/bootloader
nix build .#ota                    # OTA zip for adb sideload
```

Build with signed keys (requires sandbox exception):
```bash
nix-build --arg configuration ./configuration.nix -A img \
  --option extra-sandbox-paths /keys=$(pwd)/keys
```

Generate signing keys:
```bash
nix-build --arg configuration ./configuration.nix -A generateKeysScript -o generate-keys
./generate-keys ./keys
```

## Architecture

**Robotnix Module System**: Uses NixOS-style modules to compose Android builds. Key components:
- `flavors/` - Android distribution flavors (grapheneos, lineageos)
- `modules/` - Feature modules (signing, apps, kernel, etc.)
- `apks/` - Nix expressions for building/fetching APKs

**Configuration Flow**:
1. `flake.nix` calls `robotnix.lib.robotnixSystem` with configuration
2. Configuration is evaluated through `default.nix` which loads all modules
3. Modules provide options and build outputs (img, ota, factoryImg, etc.)

**Custom Build Structure** (`custom-build/`):
- `flake.nix` - Defines robotnix configuration and exposes build outputs
- `configuration.nix` - Device/flavor settings and prebuilt apps
- `apks/fenix/` - Nix expression to fetch Firefox APK

## Key Configuration Options

```nix
{
  device = "caiman";           # Pixel device codename
  flavor = "grapheneos";       # grapheneos or lineageos
  signing.enable = true;       # Enable for production builds
  signing.keyStorePath = "/path/to/keys";
  ccache.enable = true;        # Faster rebuilds (needs sandbox config)
  apps.prebuilt.<name> = { apk, privileged, packageName, ... };
}
```

Supported GrapheneOS devices: akita, barbet, bluejay, bramble, caiman, cheetah, comet, coral, felix, flame, husky, komodo, lynx, oriole, panther, raven, redfin, shiba, sunfish, tangorpro, tegu, tokay

## Adding Prebuilt APKs

```nix
apps.prebuilt.MyApp = {
  enable = true;
  apk = pkgs.fetchurl { url = "..."; sha256 = "..."; };
  certificate = "PRESIGNED";  # Or path to signing cert
  packageName = "com.example.app";
  privileged = false;         # true for system privileges
  partition = "product";      # product, system, or vendor
};
```

## Build Requirements

- 250GB+ free disk space, 16GB+ RAM
- `/tmp` must NOT be tmpfs (intermediate builds are huge)
- User namespaces enabled (`CONFIG_USER_NS`)
- For ccache: add `/var/cache/ccache` to `nix.settings.sandbox-paths`

## Binary Cache

```bash
cachix use robotnix  # Provides pre-built kernels and browser builds
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/longregen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
