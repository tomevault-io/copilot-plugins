## vw-meteor

> Modified version of the VW Meteor truck mod for Euro Truck Simulator 2 (originally from Steam Workshop ID 2232719369). This is a personalized local version with custom engines, sounds, transmissions, and tuning.

# VW Meteor - ETS2 Mod (Modified)

## Overview
Modified version of the VW Meteor truck mod for Euro Truck Simulator 2 (originally from Steam Workshop ID 2232719369). This is a personalized local version with custom engines, sounds, transmissions, and tuning.

## Project Structure

```
ets2moddev/
  backup_vw/                          # Original unmodified .scs backup
  truck.jpg                           # Mod thumbnail image
  volkswagen_meteor/                  # Unpacked mod source (this gets packaged into .scs)
    manifest.sii                      # Mod manifest
    mod_description.txt               # Mod description shown in-game
    vw_meteor.jpg                     # In-mod thumbnail
    def/vehicle/truck/volkswagen.meteor/
      engine/                         # Engine definitions (.sii) and sound files (.sui, .soundref)
      transmission/                   # Transmission definitions
      chassis/                        # Chassis definitions (suspension, weight)
      paint_job/                      # Paint jobs (factory colors only)
      accessory/tank/                 # Fuel tank definitions
      badge_b/                        # Badge overrides
    vehicle/truck/volkswagen_meteor/  # 3D models, textures (.dds, .tobj)
    material/ui/                      # UI icons and dashboard textures
```

## Key Technical Notes

### SII File Format
- Files use `SiiNunit { }` wrapper with typed data blocks
- Array declarations use `property:count` (NO space before colon), e.g., `sounds:5`
- Unit names (before first dot) cannot use game-reserved namespaces like `scania` or `volvo`
- Use short prefixes: `sc13_`, `scv8_`, `vd13_`, `vd16_` for engine names
- `@include "file.sui"` pulls in sound definitions

### Sound System
- `.soundref` files point to `.bank#event` paths
- `.sui` files group multiple soundref includes and declare sound arrays
- Sound banks must exist in the base game or DLC (verify in game.log)
- Current engine sounds: Volvo FH (default), Scania 2016, Scania V8, Volvo 500, Volvo 750

### Packaging (.scs)
- `.scs` files are standard ZIP archives (NOT HashFS)
- **NEVER use PowerShell `Compress-Archive`** - it creates incompatible ZIPs (backslashes, missing directory entries)
- Always use .NET `System.IO.Compression.ZipFile` API in Update mode
- Workflow: restore backup original .scs first, then patch all modified files on top
- Target path: `D:/Documents/Euro Truck Simulator 2/mod/volkswagen_meteor.scs`

### Packaging Command Pattern
```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
Copy-Item backup_vw/volkswagen_meteor_original.scs "D:/Documents/Euro Truck Simulator 2/mod/volkswagen_meteor.scs" -Force
$zip = [System.IO.Compression.ZipFile]::Open("D:/Documents/Euro Truck Simulator 2/mod/volkswagen_meteor.scs", 'Update')
# ... add/update entries with forward slashes
$zip.Dispose()
```

### Common Pitfalls
- `suitable_for[]` in transmissions must match the engine that references them via `defaults[]`, or remove `suitable_for` entirely
- `part_type: factory` makes items show in the default tab; `aftermarket` shows in aftermarket tab
- `residual_travel[]` controls suspension firmness (lower = firmer)
- `retarder:` value in transmissions controls retarder braking strength
- Torque curves: array of `(RPM, multiplier)` where multiplier 0-1

### Debugging
- Game log: `D:/Documents/Euro Truck Simulator 2/game.log.txt`
- Look for `<ERROR> [unit]` lines for SII parsing issues
- `Failed to open file` indicates missing sound banks or textures
- Close ETS2 before replacing the .scs file

## What Has Been Modified

### Engines (def/vehicle/truck/volkswagen.meteor/engine/)
- **MAN D2676 variants**: d2676_294 (400HP), d2676_324 (440HP), d2676_397 (540HP)
- **Claude VW Edition**: d2676_claude (510HP) - progressive torque curve, smooth cruise
- **Scania DC13**: sc13_450 (450HP), sc13_530 (530HP)
- **Scania DC16 V8**: scv8_580 (580HP), scv8_730 (730HP)
- **Volvo D13K**: vd13_460 (460HP), vd13_500 (500HP)
- **Volvo D16**: vd16_600 (600HP), vd16_750 (750HP)
- All engines compatible with all 4 chassis types
- Engine sounds: Volvo FH (default/MAN), Scania 2016, Scania V8, Volvo 500, Volvo 750

### Transmissions (def/vehicle/truck/volkswagen.meteor/transmission/)
- 3 overdrive variants: 12-speed OD, 16-speed OD, 16-speed OD with retarder
- All transmissions universal (no `suitable_for` restrictions)
- Retarder strength: 5 on retarder-equipped transmissions

### Chassis (def/vehicle/truck/volkswagen.meteor/chassis/)
- Firm suspension on all 4 chassis types (reduced `residual_travel`)
- Real VW Meteor weight: ~9200kg (6x4), ~8700kg (6x2)

### Fuel Tanks (def/vehicle/truck/volkswagen.meteor/accessory/tank/)
- Added 1300L aluminum tanks (650+650 split) for all chassis
- All tank icons changed to MAN TGX Euro6 aluminium style

### Sounds
- Engine: Volvo FH banks (default)
- Retarder: Volvo FH16 banks
- Per-engine sounds via .sui include files

### Paint Jobs
- Removed 228 brand/company skins
- Kept 9 factory colors + Silver Meteor VW

### Other
- All prices set to 1
- Custom thumbnail (truck.jpg)
- Mod unlocked from level 0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devlj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
