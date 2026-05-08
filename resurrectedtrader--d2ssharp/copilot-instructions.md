## d2ssharp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

D2SSharp is a C# library for reading and writing Diablo 2 save files (.d2s character saves and .d2i shared stash files). It supports the original D2 LOD format (version 96), Diablo 2 Resurrected format (version 97+), and the latest D2R format (version 105) with its restructured header and new item serialization.

## Build Commands

```bash
# Build the solution
dotnet build D2SSharp.sln

# Run all tests
dotnet test

# Run a specific test
dotnet test --filter "FullyQualifiedName~RoundTripTests.RoundTrip_CompactItem_Gold"

# Run tests with verbose output
dotnet test --logger "console;verbosity=detailed"
```

## Architecture

### Core Components

- **D2Save** (`Model/D2Save.cs`): Root model for character save files. Contains all sections: Character, Quests, Waypoints, Skills, Items, Corpses, MercItems, IronGolem, Demon (v103+).

- **D2StashSave** (`Model/D2StashSave.cs`): Model for shared stash files. Contains a list of D2StashTab entries. Each tab has a `StashTabType` (Normal, AdvancedStash, Chronicle) that determines its body content. Chronicle tabs track found set/unique/runeword items.

- **Item** (`Model/Item.cs`): Complete item model with all properties. Handles both compact save format (quest items, gold, gems, runes) and complete format (equipment with stats).

- **BitReader/BitWriter** (`IO/`): Low-level bit manipulation for reading/writing the binary format. Bits are read LSB-first within each byte.

### External Data System

The library uses embedded game data tables to parse items. For standard saves, no explicit external data is needed:

```csharp
// Read/write using built-in embedded data (versions 96, 97, 99, 105)
var save = D2Save.Read(bytes);
int written = save.Write(buffer);
```

For modded games with custom items/stats:

```csharp
// Load from directory with version subdirectories (e.g., MyData/96/, MyData/99/)
var modData = new TxtFileExternalData(@"C:\path\to\MyData");

// Or load from a single directory for a specific version
var modData = new TxtFileExternalData(@"C:\path\to\MyData\99", version: 99);

// Use for reading/writing modded saves
var save = D2Save.Read(bytes, modData);
int written = save.Write(buffer, modData);
```

- **TxtFileExternalData.Default**: Shared default instance with embedded data for versions 96, 97, 99, 105
- **IExternalData** (`Data/IExternalData.cs`): Interface for providing stat and item type information

The external data provides:
- `StatInfo`: Bit widths, save offsets, and value shifts for each stat ID
- `ItemTypeInfo`: Item classification (armor/weapon/gold/stackable/etc.)
- Item code to index mapping

### Save Version Handling

The library uses `saveVersion` to distinguish formats:
- **Version 96**: Original D2 LOD format (32-bit item codes, 7-bit strings, 10-bit item format)
- **Version 97+**: D2R format (Huffman-encoded item codes, 8-bit strings, compact 3-bit item format, 4-field realm data)
- **Version 100+**: Advanced stash category data, chronicle data (item find tracking)
- **Version 103+**: DemonSection ("lf" magic) after IronGolem
- **Version 104+**: New header format (403 bytes), Name moved from Character to PreviewData, expanded SaveTimes/Experiences arrays, GameVersion field
- **Version 105+**: Item quantity uses 1-bit presence flag for ALL items (not just stackable)

### Shared Stash Tab Format

Each stash tab has a 64-byte header: Magic(4) + StashFormat(4) + ItemFormat(4) + Gold(4) + Size(2) + Season(2) + TabType(1) + Reserved(43).

- **StashFormat < 2**: TabType is forced to Normal on read (game ignores the byte). Chronicle tabs are skipped entirely on write.
- **StashFormat >= 2**: TabType determines the tab body content:
  - `Normal` (0): Items (JM section)
  - `AdvancedStash` (1): Items with stackable support (JM section)
  - `Chronicle` (2): Chronicle section (magic 0xC0EAEDC0) tracking found set/unique/runeword items

The chronicle tab writer in the game has a size calculation bug (`add ax, 40h` at 0x140311e98) that adds 64 bytes of stale buffer data to the tab size. The reader ignores these bytes. The library preserves them in `ChronicleSection.TrailingData` for byte-exact round-trip.

### Item Serialization

Items branch based on the `CompactSave` flag:
- **Compact items**: Minimal data (gold, gems, runes, quest items)
- **Complete items**: Full equipment with seed, level, quality, affixes, stats

Key bit-level encoding details:
- Stats use `SaveBits`, `SaveAdd`, and `ValShift` from ItemStatCost.txt
- Paired stats (cold/poison damage) have trailing values without their own ID
- Realm data size differs by version (96 bytes vs 128 bytes)

### Stat Value Encoding

When reading: `semantic_value = (raw_value - SaveAdd) << ValShift`
When writing: `raw_value = (semantic_value >> ValShift) + SaveAdd`

## Version Conversion

The library supports converting between formats via `D2Save.Write(buffer, targetVersion)`. Handles three boundaries: v96↔v97+ (1.14↔D2R), v<=103↔v104+ (old↔new header), and v<=104↔v105+ (item quantity format).

Key conversion logic in `D2Save.PrepareForVersion()`:

**1.14 → D2R:**
- `Character.Name` → `Character.Preview.Name` (D2R uses UTF-8 preview name)
- Populate `Preview.LeftHand/RightHand/Torso/Head` from equipped items
- `Preview.Transform` looked up from `Character.AppearanceTints[(int)Composits.*]`
- `Preview.Flags` = `ItemFlags.Targeting` for primary weapon only
- Zero `BodyLocation` for non-equipped items (D2R fails otherwise)

**D2R → 1.14:**
- `Character.Preview.Name` → `Character.Name`
- Zero all Preview items

**Note:** BodyLocation swaps between internal representation and visual:
- `BodyLocation.RightArm` (primary weapon slot) → `Preview.LeftHand` (left on screen)
- `BodyLocation.LeftArm` (offhand slot) → `Preview.RightHand` (right on screen)

## Mod Compatibility

- **TrailingData**: `D2Save.TrailingData` captures any bytes after the last known section (IronGolem or Demon). These are preserved during round-trip for mod compatibility.
- **TxtFileExternalData**: Load mod-specific .txt files for custom items/stats by providing a directory with version subdirectories.
- Files with missing sections (e.g., Expansion flag set but no MercItems/IronGolem) will fail to parse.

## Section Magic Values

Each section defines a `Magic` constant used for validation. Useful for debugging hex dumps:

| Section | Magic | Hex (LE) |
|---------|-------|----------|
| D2Save header | 0xAA55AA55 | `55 AA 55 AA` |
| QuestSection | "Woo!" (0x216F6F57) | `57 6F 6F 21` |
| WaypointSection | "WS" (0x5357) | `57 53` |
| PlayerIntroSection | 0x7701 | `01 77` |
| PlayerStats | "gf" (0x6667) | `67 66` |
| SkillsSection | "if" (0x6669) | `69 66` |
| ItemsSection | "JM" (0x4D4A) | `4A 4D` |
| CorpseSection | "JM" (0x4D4A) | `4A 4D` |
| MercItemsSection | "jf" (0x666A) | `6A 66` |
| IronGolemSection | "kf" (0x666B) | `6B 66` |
| Item (v96 only) | "JM" (0x4D4A) | `4A 4D` |
| DemonSection (v103+) | "lf" (0x666C) | `6C 66` |
| D2StashTab header | 0xAA55AA55 | `55 AA 55 AA` |
| ChronicleSection | 0xC0EAEDC0 | `C0 ED EA C0` |

## Overlay API

The overlay API (`D2SaveOverlay.cs`) provides zero-copy, blittable struct access to the fixed-size header sections of save files via `MemoryMarshal.AsRef`. This allows direct read/write of header fields without parsing the full save.

Two layout structs exist for different save versions:

- **`D2SaveLayout`** (765 bytes): For v<=103 saves. Character section includes 16-byte Name field and 144-byte PreviewData.
- **`D2SaveLayoutV104`** (833 bytes): For v>=104 saves. Character section removes Name (now in PreviewData) and expands PreviewData to 228 bytes (+68 bytes total).

```csharp
var data = File.ReadAllBytes("save.d2s");

// Check version first, then use the appropriate layout
uint version = BitConverter.ToUInt32(data, 4);
if (version >= 104)
{
    ref var overlay = ref D2SaveLayoutV104.From(data);
    overlay.Character.Level = 99;
    D2SaveLayoutV104.UpdateChecksum(data);
}
else
{
    ref var overlay = ref D2SaveLayout.From(data);
    overlay.Character.Level = 99;
    D2SaveLayout.UpdateChecksum(data);
}
```

Both layouts provide a `Name` property that handles version-aware name access, `From()` for validation, and `UpdateChecksum()` for recalculating the checksum after modifications. Using the wrong layout for a version throws `InvalidDataException`.

## Testing

All tests use `TxtFileExternalData.Default` which provides embedded stat/item info for versions 96, 97, 99, and 105. For modded saves, use a custom `TxtFileExternalData` instance loaded from mod-specific .txt files.

Round-trip tests verify that read -> write produces identical bytes.

### Test Resources

Resources are organized by save version:
- `Resources/96/` - D2 1.14 format saves (version 96)
- `Resources/97/` - D2R format saves (version 97)
- `Resources/99/` - D2R 1.5+ format saves (version 99)
- `Resources/105/` - D2R latest format saves (version 105) with new header and item format
- `Resources/Modded/` - Modded saves with custom .txt files in `Txt/99/`

### Running Specific Test Groups

```bash
# Conversion tests
dotnet test --filter "Conversion"

# All round-trip tests
dotnet test --filter "RoundTrip"
```

---
> Source: [ResurrectedTrader/D2SSharp](https://github.com/ResurrectedTrader/D2SSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
