## notenix

> A standalone, minimal, auto-updating NixOS configuration for laptops/desktops.

# notenix — Portable NixOS GNOME Desktop Flake

A standalone, minimal, auto-updating NixOS configuration for laptops/desktops.
All NixOS modules live in this repo under the `notenix.*` option namespace.
No external module frameworks — nixpkgs and disko are the only flake inputs.

## Repo structure

```
flake.nix               — nixosConfigurations (notenix, vm-headless, vm-gnome) + install package
modules/                — all NixOS option modules, imported as nixosModules.default
  default.nix           — imports all category modules below
  system/
    install.nix         — notenix.system.install.* (hostname, user, locale, keyboard)
    nix.nix             — notenix.system.nix.* (flakes, GC, unfree, fast shutdown)
    autoupgrade.nix     — notenix.system.autoupgrade.* (daily flake rebuild + notify)
  boot/
    systemd-boot.nix    — notenix.boot.systemd-boot.* (EFI boot, kernel)
  desktop/
    gnome.nix           — notenix.desktop.gnome.* (GNOME, GDM, extensions, dconf)
  applications/
    flatpak.nix         — notenix.applications.flatpak.* (Flathub, package list)
  network/
    networkmanager.nix  — notenix.network.networkmanager.*
  hardware/
    bluetooth.nix       — notenix.hardware.bluetooth.*
    printing.nix        — notenix.hardware.printing.*
    sound.nix           — notenix.hardware.sound.*
  security/
    sudo.nix            — notenix.security.sudo.wheelNeedsPassword
hosts/notenix/
  configuration.nix     — reference host; all notenix.* options for the machine
  disk.nix              — disko disk layout
_files/                 — helper scripts (notify-users.sh)
```

## Flake inputs

| Input | Purpose |
|-------|---------|
| `nixpkgs` | nixos-25.11 |
| `disko` | disk partitioning for install |

## Option namespace

All module options live under `notenix.*`. Example:

```nix
notenix.system.install = {
  enable          = true;
  hostName        = "mymachine";
  userName        = "youruser";
  userDescription = "Your Name";
  timeZone        = "Europe/Ljubljana";
  locale          = "sl_SI.UTF-8";
  keyboardLayout  = "si";
};
notenix.boot.systemd-boot.enable        = true;
notenix.desktop.gnome.enable            = true;
notenix.applications.flatpak.enable     = true;
notenix.system.nix.enable               = true;
notenix.system.autoupgrade.enable       = true;
notenix.system.autoupgrade.flakeRepo    = "github:yourusername/yourrepo";
notenix.network.networkmanager.enable   = true;
notenix.hardware.bluetooth.enable       = true;
notenix.hardware.printing.enable        = true;
notenix.hardware.sound.enable           = true;
```

## nixosConfigurations

| Name | Purpose |
|------|---------|
| `notenix` | Reference configuration for the real laptop; used by `nixos-rebuild` |
| `vm-headless` | Minimal headless VM for smoke-testing (user: `user` / pass: `notenix`) |
| `vm-gnome` | Full GNOME desktop VM for visual/interactive testing |

## Service debugging (small-first)

Use shortest reproducible path.

1. Locate option + module
  - Check host `hosts/<name>/configuration.nix`.
  - Check module under `modules/**` for option behavior.
2. Check runtime unit
  - `systemctl status <unit>`
  - `journalctl -u <unit> -n 200 --no-pager`
3. Check dependency chain
  - Timers, sockets, targets, reverse proxy (if used).
4. Check app-native logs/tools
  - Prefer native CLI status command when available.
5. Close minimal
  - Cause, smallest fix, single verify command.

Run VMs:
```bash
nix run .#vm          # headless
nix run .#vm-gnome    # GNOME desktop (needs QEMU display)
```

## Adding a new host

1. Copy `hosts/notenix/` to `hosts/<yourhostname>/`
2. Edit `hosts/<yourhostname>/configuration.nix` — update identity and module options
3. Register in `flake.nix` under `nixosConfigurations`:
   ```nix
   <yourhostname> = lib.nixosSystem {
     inherit system;
     modules = [
       self.nixosModules.default
       disko.nixosModules.disko
       ./hosts/<yourhostname>/configuration.nix
       ./hosts/<yourhostname>/disk.nix
     ];
   };
   ```

## Adding a feature flag

`default.yaml` is the **single source of truth** for all dynamic kanal tabs (features, extensions, apps). `constants.py`, `cli.py`, and `gui/window.py` all read it at runtime — no Python changes needed for ordinary features or extensions.

### 1. `pkgs/kanal/src/kanal/default.yaml`

**Bool feature** (toggle in the Features tab):
```yaml
# under tabs → id: features → items:
- key: notenix.features.myFeature
  id: myFeature
  const: MY_FEATURE          # becomes KEY_FEATURE_MY_FEATURE constant auto-magically
  title: My Feature
  subtitle: "One-line description shown in the GUI"
  default: false
```

Optional `extra` field wires a bool feature to also add/remove an entry from the GNOME extensions list:
```yaml
  extra:
    type: gnome_extension
    value: tailscale-status   # extension id in notenix.desktop.gnome.extensions
```

**List extension** (toggle in the Extensions tab):
```yaml
# under tabs → id: extensions → items:
- id: my-extension-id        # matches the _extPkgs key in gnome.nix
  title: My Extension
  subtitle: "What it does"
  default: false
```

**List extension with selectable package source** (source picker shown when Experimental is ON):
```yaml
- id: my-extension-id
  title: My Extension
  subtitle: "What it does"
  default: false
  nix_source_key: notenix.desktop.gnome.myExtSource    # NixOS option written to machine.nix
  nix_hash_key:   notenix.desktop.gnome.myExtSourceHash # sha256 stored for upstream fetches
  sources:
    - id: unstable
      label: "Unstable (nixpkgs-unstable)"
      default: true          # this id is omitted from machine.nix (matches NixOS module default)
    - id: stable
      label: "Stable (nixpkgs)"
    - id: upstream-main
      label: "Upstream main"
      fetch_url: "https://example.com/repo/-/archive/main/repo-main.tar.gz"
      # fetch_url triggers an automatic nix-prefetch-url --unpack when selected in kanal
      # result stored in nix_hash_key; used by NixOS module via builtins.fetchTarball
```

The `nix_source_key` id must match the `default` of the first source with `default: true` — that value is **not written** to machine.nix (module default takes over). All other source ids are written as `lib.mkForce "id"`.

The `experimental` feature flag (also in `default.yaml`) gates source picker visibility in the UI. Source pickers are hidden and reset to default when experimental is disabled.

### 2. `modules/system/features.nix`

Add the NixOS option and its `config` block:
```nix
myFeature = mkOption {
  type        = types.bool;
  default     = false;
  description = "One-line description.";
};
```
Inside `config = mkMerge [`:
```nix
(mkIf cfg.myFeature {
  # NixOS config here
})
```

For extensions only — if the extension package isn't already in `_extPkgs` in `modules/desktop/gnome.nix`, add it there too:
```nix
"my-extension-id" = pkgs.gnomeExtensions.myExtension;
```

### 3. Test the build
```bash
nix build path:.#nixosConfigurations.notenix.config.system.build.toplevel
```
Build must succeed before committing.

---

## Known extension issues

### gtk4-desktop-icons-ng-ding (Desktop Icons NG)

`gnomeExtensions.gtk4-desktop-icons-ng-ding` v129 (nixpkgs unstable, 2026-05) **crashes on GNOME 49.4** with:

```
Adw-DING: Gjs:ERROR: assertion failed: (gtype != G_TYPE_INVALID)
cannot register existing type 'GDesktopAppInfoLookup'
```

The extension subprocess (`adw-ding.js`) ABORTs and respawns in an infinite crash loop. Extension settings open fine but no icons appear on the desktop. Extension is **disabled by default** in `default.yaml`. Do not enable until a fixed package version lands in nixpkgs.

---

## Install on a real machine

Boot NixOS minimal ISO, then:

```bash
nix run github:khartahk/notenix \
  --extra-experimental-features "nix-command flakes" \
  --no-write-lock-file
```

## Deploying changes to the running laptop

```bash
nixos-rebuild boot --sudo --ask-sudo-password \
  --flake .#notenix \
  --target-host uporabnik@<ip>
```

Use `switch` instead of `boot` to activate immediately without reboot.

## Checking auto-update status

```bash
systemctl status nixos-upgrade.service
journalctl -u nixos-upgrade.service -f
systemctl list-timers nixos-upgrade.timer
sudo nix-env --list-generations --profile /nix/var/nix/profiles/system
sudo nixos-rebuild switch --rollback
```

---
> Source: [n1x05/notenix](https://github.com/n1x05/notenix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
