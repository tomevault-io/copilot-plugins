## more-npcs

> S1API NPC creation reference - conventions and API usage for Schedule I mod NPCs


# S1API NPC Creation Reference

Use this when creating or modifying NPCs for the More-NPCs mod. Full S1API docs: https://ifbars.github.io/S1API/api/S1API.Entities.html

**When unsure about paths (avatar layers, buildings, etc.), ALWAYS check S1API docs or source** — e.g. https://github.com/ifBars/S1API (Head.cs for head accessories, Buildings for building types). Do not guess paths.

## Project Convention: Schedule Actions

**Use `plan.Add(new WalkToSpec { ... })` for walk-to actions** — NOT the fluent `.WalkTo()` shortcut. This keeps schedules consistent with spec-based actions like `StayInBuildingSpec`, `UseVendingMachineSpec`.

```csharp
// ✅ Correct
plan.Add(new WalkToSpec { Destination = position, StartTime = 900, FaceDestinationDirection = true });

// ❌ Avoid
plan.WalkTo(position, 900, faceDestinationDir: true);
```

Other fluent methods (StayInBuilding, UseVendingMachine, UseATM, LocationDialogue) are fine — only WalkTo is standardized to plan.Add.

## NPC Structure

Source files live under `NPCs/{Northtown|Westville|Downtown|Docks|Suburbia|Uptown}/` (namespace stays `MoreNPCs.NPCs`).

1. **ConfigurePrefab** — Identity, appearance, customer/dealer, relationships, schedule. All prefab setup happens here.
2. **OnCreated** — Runtime init: `Appearance.Build()`, `Schedule.Enable()`, set `Aggressiveness`, `Region`.

## NPCPrefabBuilder (ConfigurePrefab)

| Method | Purpose |
|-------|---------|
| `WithIdentity(id, firstName, lastName)` | Unique ID and display name |
| `WithAppearanceDefaults(av => {...})` | Avatar settings, hair, layers, colors |
| `WithSpawnPosition(Vector3)` | World spawn position |
| `EnsureCustomer()` | Enables customer behavior |
| `WithCustomerDefaults(cd => {...})` | Spending, orders, standards, affinities |

### Name blacklist

- Never use `Marcus` as an NPC first name.

## Spending limits (guidelines)

Spending values are **guidelines/suggestions** per region, not hard caps. Use increments of 100 (e.g. 400, 500, 600). Regional ranges:

| Region | Min–Max (weekly) |
|--------|------------------|
| Northtown | 200–800 |
| Westville | 400–700 |
| Downtown | 400–1000 |
| Docks | 600–1000 |
| Suburbia | 700–1400 |
| Uptown | 1000–2000 |

**Extra-rich Uptown NPCs** (Daniel J Dalby, Darius Cole, Edward Boog) use 1000–2000 or 2000–8000 — above the guideline.

## Orders per week

Keep `WithOrdersPerWeek` max at 4 or lower — higher values spread the weekly budget thin and make per-order spend too small.
| `WithRelationshipDefaults(r => {...})` | Delta, unlock type, connections |
| `WithSchedule(plan => {...})` | Schedule actions |

## Schedule Actions (PrefabScheduleBuilder)

**Always call `plan.EnsureDealSignal()` first** for customer NPCs.

### StayInBuilding usage rule

- Prefer `plan.StayInBuilding(buildingVar, startTime, durationMinutes)` for any in-game building that has a typed `Building.Get<...>()` class.
- Use `plan.Add(new StayInBuildingSpec { BuildingName = "..." })` only for custom/untypeable building names.
- Do not use `BuildingName` string for base-game buildings when a typed building exists.

**Vanilla vs mod-added names**

- **Base-game locations** exposed under `S1API.Map.Buildings`: use `var b = Building.Get<T>();` and `plan.StayInBuilding(b, startTime, durationMinutes)`. Examples: `FishWarehouse`, `SouthOverpassBuilding`, `Cafe`, `Supermarket`, `SlopShop`, `HylandBank`, etc. Check S1API docs/source for the exact type name; do not guess `BuildingName` strings for those when `T` exists.
- **Mod-only enterables** (e.g. doors/buildings created at runtime in `MoreNPCs.Utils.BuildingSetup` like `"Apartment Building 2"`, `"Red Docks Shipping Container"`): there is no `Building.Get<>()` — use `plan.Add(new StayInBuildingSpec { BuildingName = "Exact Name", ... })` only for those.

| Action | Usage |
|--------|-------|
| WalkTo | `plan.Add(new WalkToSpec { Destination = pos, StartTime = 900, FaceDestinationDirection = true })` |
| StayInBuilding | `plan.StayInBuilding(building, startTime, durationMinutes)` or `plan.Add(new StayInBuildingSpec { BuildingName = "Name", StartTime = 900, DurationMinutes = 60 })` |
| UseVendingMachine | `plan.UseVendingMachine(startTime)` |
| UseATM | `plan.UseATM(startTime)` |
| LocationDialogue | `plan.LocationDialogue(destination, startTime, faceDestinationDir)` |

**Time format**: HHMM — first two digits = hour (00–23), last two = minutes (00–59). Examples: `900` = 09:00, `1258` = 12:58. Invalid: `1298` (12:98), `1696` (16:96). Minutes must be 00–59.

**DurationMinutes**: **Total length of the stay in minutes** — a plain integer (e.g. `149`, `559`), **not** `HHMM` clock time. Do not encode hours/minutes as four digits here (that is only for `StartTime`). Set so the stay ends 1 minute before the next activity. Formula: (next_StartTime_minutes - this_StartTime_minutes) - 1. For overnight: (1440 - this) + next - 1. Max 1439 (1 min before same time next day).

**Home stays across midnight**: Prefer **one** `StayInBuilding` from evening through the next morning (duration = minutes from evening start until 1 min before the next day’s first outing) instead of ending a home block exactly at `00:00` and starting another at `000`. The split can make the NPC briefly leave and re-enter the same building.

**StartTime order**: Do not reorder or vary StartTimes in a way that makes a later entry earlier than a previous one (e.g. first 1405, second 1355) — that breaks schedule flow.

**Idle after leaving a building or standing from a seat**: Usually means the schedule has **no action** for some real-time minutes after one block ends. Common causes:

1. **Duration / next-start mismatch** — If `StayInBuilding` or `SitSpec` ends more than ~1 minute before the next line’s `StartTime`, the NPC has nothing to do until that time fires. Re-check `DurationMinutes` with the formula above (same for `SitSpec`: end time = start + duration; next action should be `end + 1` minute in clock terms).
2. **Long gap to the next timed action** — Example: leave a stay at 10:28 but the next entry is `SitSpec` at 10:39 → 11 minutes with no `WalkTo` or other action; they will stand around. Fix by tightening duration so the stay ends at 10:38, or add `plan.Add(new WalkToSpec { Destination = ..., StartTime = 1029, ... })` (or similar) toward the next location.
3. **Building → building / sit → building without a walk leg** — The base game often expects movement to be **scheduled** (e.g. `WalkToSpec` to the next area) when chaining distant locations. If they still freeze right after an activity despite tight times, add a `WalkToSpec` to the next destination with `StartTime` equal to the minute after the previous block ends.

After **bulk edits** to `StartTime` (e.g. phase offsets), re-run duration math so every chain still ends “1 minute before” the next line.

### Building occupancy (planning guideline)

When placing `StayInBuilding` / vanilla venues in schedules, use this **soft target** so crowds stay plausible and nav congestion stays lower:

- Treat each **vanilla enterable** as if it should usually have **at least 2** NPCs “worth” of presence across the day (spread across mod + vanilla schedules — you cannot count vanilla NPCs precisely in code here, so use judgment when several mod NPCs share one venue).
- Aim for **no more than 5 mod NPCs in the same building at the same clock window** when you can stagger or alternate venues instead.
- If many NPCs must share one building, **offset `StartTime` by 60+ minutes** (see schedule similarity rule) unless they are an intentional group.

### Schedule similarity rule

- New or updated schedules should not be more than ~40% similar to another existing NPC schedule pattern.
- Avoid cloning the same building chain + near-identical time blocks across multiple NPCs.
- If two NPCs visit the same location and are not explicitly intended to travel together by user request, offset their visit or walk times by at least 60 minutes.
- If the user explicitly says two NPCs travel together, same-time visits are allowed for those NPCs only.

## Numeric values: use two decimals

Use **two decimal places** for values like `Aggressiveness`, affinities, `WithMutualRelationRequirement`, `WithDependence`, etc. Single decimals (e.g. `0.5f`) give ~10% resolution; two decimals (e.g. `0.52f`, `0.47f`) allow more variation.

```csharp
// ✅ Prefer
Aggressiveness = 0.48f;
.WithAffinities(new[] { (DrugType.Marijuana, 0.52f), ... })

// ❌ Avoid (single decimal)
Aggressiveness = 0.5f;
```

## AvatarDefaultsBuilder (appearance)

- `Gender`, `Height`, `Weight`, `SkinColor`, `HairColor`, `HairPath`
- **Colors**: Use ~20% less vibrant/saturated colors than you'd expect — in-game render tends to boost saturation by ~20%. Reduce saturation (push RGB toward gray) or scale bright colors down. E.g. a vivid red 1.0,0.2,0.1 → use 0.8,0.16,0.08.
- `WithFaceLayer(path, color)`, `WithBodyLayer(path, color)`, `WithAccessoryLayer(path, color)`
- Paths: `"Avatar/Hair/Spiky/Spiky"`, `"Avatar/Layers/Top/T-Shirt"`, etc.

## S1API Clothing Options By Placement

Use these doc-backed appearance paths instead of guessing. Sources:
- Namespace index: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.html
- Shirts: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.BodyLayerFields.Shirts.html
- Pants: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.BodyLayerFields.Pants.html
- Body accessories (gloves): https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.BodyLayerFields.Accessories.html
- Accessory bottom: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Bottom.html
- Accessory chest: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Chest.html
- Accessory feet: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Feet.html
- Accessory hands: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Hands.html
- Accessory head: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Head.html
- Accessory neck: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Neck.html
- Accessory waist: https://ifbars.github.io/S1API/api/S1API.Entities.Appearances.AccessoryFields.Waist.html

### Body Layer: Top / Shirts (`WithBodyLayer`)

- `Avatar/Layers/Top/Buttonup`
- `Avatar/Layers/Top/ChestHair1`
- `Avatar/Layers/Top/FastFood T-Shirt`
- `Avatar/Layers/Top/FlannelButtonUp`
- `Avatar/Layers/Top/GasStation T-Shirt`
- `Avatar/Layers/Top/HazmatSuit`
- `Avatar/Layers/Top/Nipples`
- `Avatar/Layers/Top/Overalls`
- `Avatar/Layers/Top/RolledButtonUp`
- `Avatar/Layers/Top/T-Shirt`
- `Avatar/Layers/Top/Tucked T-Shirt`
- `Avatar/Layers/Top/UpperBodyTattoos`
- `Avatar/Layers/Top/V-Neck`

### Body Layer: Bottom / Pants (`WithBodyLayer`)

- `Avatar/Layers/Bottom/CargoPants`
- `Avatar/Layers/Bottom/FemaleUnderwear`
- `Avatar/Layers/Bottom/Jeans`
- `Avatar/Layers/Bottom/Jorts`
- `Avatar/Layers/Bottom/MaleUnderwear`

### Body Layer: Accessories / Gloves (`WithBodyLayer`)

- `Avatar/Layers/Accessories/FingerlessGloves`
- `Avatar/Layers/Accessories/Gloves`

### Accessory Slot: Bottom (`WithAccessoryLayer`)

- `Avatar/Accessories/Bottom/LongSkirt/LongSkirt`
- `Avatar/Accessories/Bottom/MediumSkirt/MediumSkirt`

### Accessory Slot: Chest (`WithAccessoryLayer`)

- `Avatar/Accessories/Chest/Blazer/Blazer`
- `Avatar/Accessories/Chest/BulletProofVest/BulletProofVest`
- `Avatar/Accessories/Chest/BulletProofVest/BulletProofVest_Police`
- `Avatar/Accessories/Chest/CollarJacket/CollarJacket`
- `Avatar/Accessories/Chest/OpenVest/OpenVest`

### Accessory Slot: Feet (`WithAccessoryLayer`)

- `Avatar/Accessories/Feet/CombatBoots/CombatBoots`
- `Avatar/Accessories/Feet/DressShoes/DressShoes`
- `Avatar/Accessories/Feet/Flats/Flats`
- `Avatar/Accessories/Feet/Sandals/Sandals`
- `Avatar/Accessories/Feet/Sneakers/Sneakers`

### Accessory Slot: Hands (`WithAccessoryLayer`)

- `Avatar/Accessories/Hands/Polex/Polex`

### Accessory Slot: Head (`WithAccessoryLayer`)

- `Avatar/Accessories/Head/Beanie/Beanie`
- `Avatar/Accessories/Head/BucketHat/BucketHat`
- `Avatar/Accessories/Head/Cap/Cap`
- `Avatar/Accessories/Head/Cap/Cap_FastFood`
- `Avatar/Accessories/Head/ChefHat/ChefHat`
- `Avatar/Accessories/Head/Cowboy/CowboyHat`
- `Avatar/Accessories/Head/FlatCap/FlatCap`
- `Avatar/Accessories/Head/LegendSunglasses/LegendSunglasses`
- `Avatar/Accessories/Head/MushroomHat/MushroomHat`
- `Avatar/Accessories/Head/Oakleys/Oakleys`
- `Avatar/Accessories/Head/PoliceCap/PoliceCap`
- `Avatar/Accessories/Head/PorkpieHat/PorkpieHat`
- `Avatar/Accessories/Head/RectangleFrameGlasses/RectangleFrameGlasses`
- `Avatar/Accessories/Head/Respirator/Respirator`
- `Avatar/Accessories/Head/SantaHat/SantaHat`
- `Avatar/Accessories/Head/SaucePan/SaucePan`
- `Avatar/Accessories/Head/SmallRoundGlasses/SmallRoundGlasses`
- `Avatar/Accessories/Head/TrashCrown/TrashCrown`

### Accessory Slot: Neck (`WithAccessoryLayer`)

- `Avatar/Accessories/Neck/GoldChain/GoldChain`

### Accessory Slot: Waist (`WithAccessoryLayer`)

- `Avatar/Accessories/Waist/Apron/Apron`
- `Avatar/Accessories/Waist/Belt/Belt`
- `Avatar/Accessories/Waist/HazmatSuit/HazmatSuit`
- `Avatar/Accessories/Waist/PoliceBelt/PoliceBelt`
- `Avatar/Accessories/Waist/PriestGown/PriestGown`

## Outfit conventions

- Belts: brown, black, or white — avoid colored belts (e.g. no blue belt).
- Gang members: pants within the scheme (white or blue, not gray).
- Same color scheme: use different clothing items; avoid same color on adjacent layers (e.g. vest and shirt). Keep shoes white.

### Anti-default rule

Avoid defaulting NPCs to:
- `Avatar/Accessories/Waist/Belt/Belt`
- `Avatar/Layers/Top/T-Shirt`
- `Avatar/Layers/Bottom/Jeans`
- generic shoes

This combination is overused and should not be the baseline.

If used, the outfit must include additional structure or it is invalid.

### Shirt + belt interaction rule

If a belt is used, the top must visually support it.

Valid:
- `Tucked T-Shirt`
- `Buttonup`
- `FlannelButtonUp`
- `RolledButtonUp`
- `V-Neck` if it sits correctly

Invalid:
- `T-Shirt` + `Belt` with no layering

Reason:
An untucked `T-Shirt` with a belt creates a broken silhouette and looks unfinished.

### T-Shirt usage rule

`T-Shirt` should not be treated as a complete outfit.

If using `T-Shirt`, add at least 2:
- chest layer (`Blazer`, `BulletProofVest`, `CollarJacket`, `OpenVest`)
- neck accessory (`GoldChain`)
- head accessory
- gloves (`FingerlessGloves` or `Gloves`)
- non-default pants (`CargoPants`, `Jorts`)
- different footwear

If not, use `Tucked T-Shirt` instead.

### Visual structure rule

Every NPC outfit must have visible structure using existing slots.

At least one of the following must be true:
- layered top (shirt + chest item)
- structured top (`Buttonup`, `FlannelButtonUp`, `RolledButtonUp`, `Tucked T-Shirt`, `V-Neck`)
- non-default pants
- accessory presence (neck, head, or hands)

Flat outfits (single loose top + jeans + shoes) are not allowed.

### Variation rule

NPCs should not resolve to the same outfit template.

Actively vary:
- tops (`Buttonup`, `FlannelButtonUp`, `RolledButtonUp`, `Tucked T-Shirt`, `V-Neck`, `Overalls`)
- bottoms (`CargoPants`, `Jeans`, `Jorts`)
- footwear (`CombatBoots`, `DressShoes`, `Flats`, `Sandals`, `Sneakers`)
- accessories across available slots

Avoid creating multiple NPCs with only minor color differences.

### Dealer outfit rule

Dealers must look more intentional and more detailed than civilians.

Do not use only:
- `T-Shirt`
- `Jeans`
- `Belt`
- `Sneakers`

Dealer NPCs must include at least 3 of the following:
- structured or non-default top (`Buttonup`, `FlannelButtonUp`, `RolledButtonUp`, `Tucked T-Shirt`, `V-Neck`, `Overalls`)
- chest layer (`Blazer`, `BulletProofVest`, `CollarJacket`, `OpenVest`)
- head accessory
- neck accessory (`GoldChain`)
- gloves (`FingerlessGloves` or `Gloves`)
- non-default pants (`CargoPants` or `Jorts`)
- non-default footwear (`CombatBoots`, `DressShoes`, `Sandals`, `Flats`)

### Dealer visual identity

Dealers should:
- use more layering than civilians
- use more accessories than civilians
- avoid relying only on belts for detail
- have a clearer outfit identity instead of looking like generic filler NPCs

Good examples:
- `FlannelButtonUp` + `GoldChain`
- `Tucked T-Shirt` + `OpenVest`
- `Buttonup` + `CollarJacket`
- `CargoPants` instead of always `Jeans`
- `CombatBoots` or `DressShoes` instead of always `Sneakers`

### Dealer signing fee (`WithSigningFee`)

Base signing fee on where the dealer lives (`Region` / `WithHome`): Northtown **500**, Westville **1000**, Downtown **2000**, Docks **3000**, Suburbia **4000**, Uptown **5000**.

### Civilian baseline rule

Civilians can be simple, but should still avoid full defaults.

At least one major slot should differ from the default pattern:
- top
- pants
- shoes
- chest
- head
- neck
- hands

Simple is fine. Repeated templates are not.

## Hair conventions

Do not default all low-gender-value NPCs to `Spiky`.
Do not default all high-gender-value NPCs to `ShoulderLength`.

Hair should vary across the full available set.

### Hair variety rule

Use the available hairstyles intentionally across NPCs instead of collapsing into one masculine default and one feminine default.

Available styles include:
- `Afro`
- `Balding`
- `BowlCut`
- `Bun`
- `BuzzCut`
- `CloseBuzzCut`
- `DoubleTopKnot`
- `Franklin`
- `FringePonyTail`
- `HighBun`
- `Jesus`
- `LongCurly`
- `LowBun`
- `MessyBob`
- `MidFringe`
- `Mohawk`
- `Monk`
- `Peaked`
- `Receding`
- `ShoulderLength`
- `SidePartBob`
- `Spiky`
- `Tony`

### Body-shape rule

The 0-1 gender value should be treated as body shape / presentation guidance, not a hard gender lock.

Lower values should generally bias toward more masculine body presentation.
Higher values should generally bias toward more feminine body presentation.
Middle values should be used often and should not be avoided.

This value affects body presentation, not a mandatory hairstyle category.

### Hair bias rule

Hairstyles can be biased by presentation, but not hard-locked.

General guidance:
- lower values should more often use styles that read more masculine
- higher values should more often use styles that read more feminine
- cross-use is valid and should still appear sometimes

Examples:
- lower-value NPCs can still use `Jesus`, `LongCurly`, or `ShoulderLength`
- higher-value NPCs can still use `MessyBob`, `SidePartBob`, `BuzzCut`, or `CloseBuzzCut`

The goal is preference, not restriction.

### Distribution rule

Do not let one hairstyle dominate a category.

Avoid patterns like:
- all masculine-presenting NPCs = `Spiky`
- all feminine-presenting NPCs = `ShoulderLength`

Spread usage across short, medium, long, neat, rough, and stylized cuts.

### Role + hair rule

Hair should support the NPC role.

Examples:
- plain civilians can use simpler/common cuts
- dealers should usually have more identity in hair, outfit, or both
- older or rougher NPCs should more often use styles like `Balding`, `Receding`, `Monk`, or other less polished options where appropriate
- not every feminine-presenting NPC needs long hair
- not every masculine-presenting NPC needs short hair

### Anti-default hair rule

If the generator keeps selecting the same hairstyle repeatedly, force variation before reusing it again.

Repeated default hair is the same problem as repeated default clothing.

## Connections (WithConnectionsById)

- **No duplicate pairs**: Don't use the same two connection IDs together twice. Contact locations will collide. If you need overlapping connections, prefer connecting to one single lightly-used NPC instead; the user can add more connections later.
- **Stereotype the vibe**: If a connection feels a certain way (e.g. white/suburban name, rough/blue-collar), the NPC you create should match that vibe. Same goes for the connection target — pick connection IDs whose characters fit the NPC you're building.

## Buildings

Use `Building.Get<BuildingType>()` for typed buildings (e.g. `Building.Get<ShermanHouse>()`).

### Northtown housing (max 3 NPCs per building)

For new Northtown NPCs, avoid North Apartments. Use these for overnight stays instead; max 3 NPCs per building:
- `Nightclub`
- `NorthIndustrialBuilding`
- `NorthWarehouse`
- `BudsBar`
- `ShootingRange`
- `Pillville`
- `Arcade`
- `SauerkrautSupreme`

NPCs can also use `SitSpec` at Taco Ticklers. Use the **`@Businesses/Taco Ticklers/Fast Food Booth`** paths exactly as listed under **Hyland Point seat paths** (booth root only — do not append `/fast food booth/Seat`).

### Regional hangout guidance (user-provided)

Use these as preferred day-time hangout anchors when building schedules. If a region has few true hangouts, use seat paths/benches as the fallback.

#### Westville
- Primary hangout: `Community Center`
- Westville is mostly residential; bench seating is expected and valid.
- Preferred Westville seat paths:
  - `Map/Hyland Point/Region_Westville/Apartment Building/Walls/OutdoorBench`
  - `Map/Hyland Point/Region_Westville/Apartment Building/Walls (1)/OutdoorBench`
  - `Map/Hyland Point/Region_Westville/Apartment Building/Walls (2)/OutdoorBench`

#### Downtown
- Preferred hangout buildings:
  - `Slop Shop`
  - `Supermarket`
  - `Cafe`
  - `The Crimson Canary`
  - `Les Ordures Puantes`

#### Docks
- No major dedicated hangout buildings. Prefer benches/seating areas.

#### Suburbia
- No major dedicated hangout buildings. Prefer benches/seating areas.
- Preferred Suburbia seat paths:
  - `Map/Hyland Point/Region_Suburbia/Residential park/OutdoorBench (2)`
  - `Map/Hyland Point/Region_Suburbia/Residential park/OutdoorBench (1)`
  - `Map/Hyland Point/Region_Suburbia/Residential park/OutdoorBench`

#### Uptown
- Use only when the NPC profile makes sense for Uptown/civic/official traffic.
- Preferred contextual hangout buildings:
  - `Church`
  - `Hyland Medical`
  - `Hyland Bank`
  - `Fire station`
  - `Courthouse`
  - `Town hall`

### Hyland Point seat paths (verified)

Use these exact scene paths when building `SitSpec`/`LocationDialogue` style seated activities.

- `@Businesses/Taco Ticklers/Fast Food Booth`
- `@Businesses/Taco Ticklers/Fast Food Booth (1)`
- `@Businesses/Taco Ticklers/Fast Food Booth (2)`
- `@Businesses/Taco Ticklers/Fast Food Booth (3)`
- `Map/Hyland Point/Bus stops/Bus Stop (10)/OutdoorBench`
- `Map/Hyland Point/Region_Northtown/Shooting range/OutdoorBench`
- `Map/Hyland Point/Region_Northtown/Plaza/OutdoorBench (2)`
- `Map/Hyland Point/Region_Northtown/Basketball court/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (8)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (9)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (7)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (6)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (5)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (4)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (3)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (2)/OutdoorBench`
- `Map/Hyland Point/Bus stops/Bus Stop (1)/OutdoorBench`
- `Map/Hyland Point/Region_Westville/Corner Store/OutdoorBench`
- `Map/Hyland Point/Region_Westville/Slums Park/OutdoorBench`
- `Map/Hyland Point/Region_Westville/OutdoorBench`
- `Map/Hyland Point/Region_Westville/OutdoorBench (1)`
- `Map/Hyland Point/Region_Docks/WaterFront/OutdoorBench`
- `Map/Hyland Point/Region_Docks/WaterFront/OutdoorBench (1)`
- `Map/Hyland Point/Region_Docks/WaterFront/OutdoorBench (2)`
- `Map/Hyland Point/Region_Docks/OutdoorBench (1)`
- `@Businesses/PostOffice/Interior/Double Sofa`
- `Map/Hyland Point/Region_Downtown/Boutique Store/Interior/Double Sofa (1)`
- `Map/Hyland Point/Region_Downtown/Boutique Store/Interior/Double Sofa`
- `Map/Hyland Point/Region_Downtown/RE Office/Interior/Double Sofa (1)`
- `Map/Hyland Point/Region_Downtown/RE Office/Interior/Double Sofa`
- `Map/Hyland Point/Region_Downtown/TownCenter/OutdoorBench (1)`
- `Map/Hyland Point/Region_Downtown/TownCenter/OutdoorBench`
- `Map/Hyland Point/Region_Downtown/Diner/Round Outdoor Set/Outdoor chair`
- `Map/Hyland Point/Region_Downtown/Diner/Round Outdoor Set/Outdoor chair (2)`
- `Map/Hyland Point/Region_Northtown/Pawn shop/Interior/Double Sofa`
- `Map/Hyland Point/Region_Northtown/Pawn shop/Interior/Outdoor chair`
- `Map/Hyland Point/Region_Northtown/Pawn shop/OutdoorBench`
- `Map/Hyland Point/Region_Northtown/Pawn shop/OutdoorBench (1)`
- `Map/Hyland Point/Region_Westville/Encampment/OutdoorBench`
- `Map/Hyland Point/Sewer/Decoration/Outdoor chair Prop (3)`
- `Map/Hyland Point/Sewer/Decoration/Outdoor chair Prop (2)`
- `Map/Hyland Point/Sewer/Decoration/Outdoor chair Prop`
- `Map/Hyland Point/Region_Westville/Apartment Building/Walls/OutdoorBench`
- `Map/Hyland Point/Region_Westville/Apartment Building/Walls (1)/OutdoorBench`
- `Map/Hyland Point/Region_Westville/Apartment Building/Walls (2)/OutdoorBench`
- `Map/Hyland Point/Region_Suburbia/Residential park/OutdoorBench (2)`
- `Map/Hyland Point/Region_Suburbia/Residential park/OutdoorBench (1)`
- `Map/Hyland Point/Region_Suburbia/Residential park/OutdoorBench`
- `Map/Hyland Point/Region_Westville/Slums Park/OutdoorBench`
- `Map/Hyland Point/Region_Westville/Slums Gas Station/slums gas station/Shop/Outdoor chair`
- `Map/Hyland Point/Region_Northtown/Waterfront/OutdoorBench (1)`
- `Map/Hyland Point/Region_Northtown/Waterfront/Round Outdoor Set/Outdoor chair`
- `Map/Hyland Point/Region_Northtown/Waterfront/Round Outdoor Set/Outdoor chair (1)`
- `Map/Hyland Point/Region_Northtown/Waterfront/Round Outdoor Set/Outdoor chair (2)`
- `Map/Hyland Point/Region_Northtown/Hardware Store/Outdoor chair`

## Namespace & Registration

- Use `MoreNPCs.NPCs` (or project namespace) for NPC classes
- Physical NPCs: `IsPhysical => true`
- In OnCreated: `base.OnCreated()`, `Appearance.Build()`, `Schedule.Enable()`

## Advanced Scheduling Examples

### 1. Working a Day Shift
When an NPC has a job, group `StayInBuilding` and `WalkTo` with `DurationMinutes` taking up the full shift time.
```csharp
// Example: 9:00 AM to 5:00 PM (17:00) Shift at Taco Ticklers
plan.Add(new WalkToSpec { Destination = tacoTicklersPos, StartTime = 850 });
plan.StayInBuilding(Building.Get<TacoTicklers>(), 900, 479); // 480 mins total (8 hours) - 1 min
```

### 2. Overnight Logic (Spanning Midnight)
If an NPC sleeps at home from 22:00 to 07:00 next day.
```csharp
// Example: Sleeping at home
// Duration: 24h * 60m = 1440m. (1440 - 2200(start)) + 700(end) = 9 hours = 540 mins - 1 min = 539
plan.StayInBuilding(Building.Get<SuburbanHouse>(), 2200, 539);
```

## In-Depth Relationship Defaults

Use `WithRelationshipDefaults` to shape how the player unlocks and builds rapport with the NPC.

**More-NPCs — `WithDelta` (starting relationship / rapport step):** Do not exceed **2.0f** except for the four child NPCs (**Bobby Cooley**, **Evan Rowland**, **Bryce Sherman**, **Cory Lubbin**), who use **3.0f**. Those four also use **1f** affinity for every `DrugType` in `WithAffinities` and **`WithSpending` max weekly 250** (or lower).

- `Delta`: How much relationship changes per positive interaction. (Usually `0.05f` to `0.10f`).
- `UnlockType`: Determines how the NPC becomes accessible (e.g., `RelationUnlockType.BuyXp`, `RelationUnlockType.Introduction`).
- `WithDependence(float)`: How quickly the NPC loses addiction/relationship if not interacted with. (e.g., `0.02f`).
- `WithMutualRelationRequirement(...)`: Demands another relationship level before this NPC will talk to the player.

```csharp
.WithRelationshipDefaults(r => r
    .SetDelta(0.08f)
    .SetUnlockType(RelationUnlockType.BuyXp, xpCost: 15)
    .WithDependence(0.03f)
    .WithMutualRelationRequirement("SomeOtherNpcId", 0.5f) // Needs 50% relationship with "SomeOtherNpcId"
    .WithConnectionsById("NpcConnection1", "NpcConnection2")
)
```

## Location Dialogue (Greetings & Interactions)

Use `LocationDialogue` for custom interactions at specific nodes, rather than simply standing idle.

```csharp
// At 12:00, stand at a position and allow dialogue, facing the player
plan.LocationDialogue(standPosition, 1200, faceDestinationDir: true);
```

## Complete NPC Template

Use this as the definitive structure for building any new NPC.

```csharp
using S1API.Entities;
using S1API.Entities.Appearances;
using S1API.Entities.Buildings;
using S1API.Entities.Schedules;
using S1API.Entities.Schedules.Specs;
using UnityEngine;

namespace MoreNPCs.NPCs
{
    public class TemplateNPC : S1NPC
    {
        public override bool IsPhysical => true;

        public TemplateNPC()
        {
            Prefab = new NPCPrefabBuilder()
                .WithIdentity("MoreNPCs_TemplateId", "John", "Doe")
                .WithAppearanceDefaults(av => av
                    .WithGender(0.2f) // 0=masc, 1=fem presentation bias
                    .WithHeight(1.0f).WithWeight(1.0f)
                    .WithSkinColor(0.8f, 0.7f, 0.6f)
                    .WithHairColor(0.1f, 0.1f, 0.1f)
                    .WithHairPath("Avatar/Hair/MessyBob")
                    .WithBodyLayer("Avatar/Layers/Top/RolledButtonUp", new Color(0.4f, 0.4f, 0.5f)) // Colors slightly desaturated
                    .WithBodyLayer("Avatar/Layers/Bottom/CargoPants", new Color(0.2f, 0.2f, 0.2f))
                    .WithAccessoryLayer("Avatar/Accessories/Feet/CombatBoots/CombatBoots", Color.black)
                )
                .WithSpawnPosition(new Vector3(100f, 0f, 100f))
                .EnsureCustomer()
                .WithCustomerDefaults(cd => cd
                    .WithSpendingLimit(600) // Weekly guideline, multiple of 100
                    .WithOrdersPerWeek(3) // 4 or fewer
                    .WithAffinities(new[] {
                        (DrugType.Marijuana, 0.55f),
                        (DrugType.Amphetamine, 0.25f)
                    })
                )
                .WithRelationshipDefaults(r => r
                    .SetDelta(0.05f)
                    .SetUnlockType(RelationUnlockType.Introduction)
                    .WithConnectionsById("ExistingNPC1", "ExistingNPC2")
                )
                .WithSchedule(plan =>
                {
                    plan.EnsureDealSignal();

                    // Sleep overnight (23:00 to 08:00 -> 9 hours = 540 mins - 1 = 539)
                    plan.StayInBuilding(Building.Get<SuburbanHouse>(), 2300, 539);
                    
                    // Morning activities
                    plan.UseVendingMachine(800);
                    
                    // Day location
                    plan.StayInBuilding(Building.Get<Arcade>(), 900, 719); // 9:00 to 21:00 (12h) = 719m
                    
                    // Evening walk
                    plan.Add(new WalkToSpec { Destination = new Vector3(150, 0, 150), StartTime = 2100, FaceDestinationDirection = true });
                    
                    plan.UseATM(2230);
                });
        }

        public override void OnCreated()
        {
            base.OnCreated();
            Appearance.Build();
            Schedule.Enable();

            Aggressiveness = 0.45f;
            Region = MapRegion.Suburbia;
        }
    }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fannsonetti)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fannsonetti)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
