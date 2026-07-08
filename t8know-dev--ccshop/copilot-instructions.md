## ccshop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **CC:Tweaked (ComputerCraft Tweaked)** Lua application for modded Minecraft. It implements an interactive item display shop using physical pedestals and a monitor UI. The shop supports multiple categories, materials, quantity selection, and payment via Numismatics depositor, integrated with Applied Energistics 2 (AE2) for stock checking.

## Running the Code

There is no build system. Code runs directly in-game on a ComputerCraft computer:
```
lua shop.lua
```

The program runs until terminated with Ctrl+T (in-game). Ensure all required peripherals are attached and configured in `config.lua`.

**Directory layout:** The script expects `config.lua`, `items.lua`, and `db.lua` to be in the `/ccshop/` directory (same folder as `shop.lua`). Logs are written to `/ccshop/shop_debug.log` and purchases to `/ccshop/purchases.json`.

## Architecture

The system is split across multiple Lua files for modularity:

- **shop.lua** — Main orchestrator that loads modules and runs the main loops.
- **config.lua** — Peripheral names, messages, timeouts, pedestal list, and validation function.
- **items.lua** — Categories, materials, quantity tiers, price definitions, and helper functions.
- **db.lua** — Purchase logging (ndjson format) to `/ccshop/purchases.json`.
- **modules/** — Modular components with single responsibilities:
  - **logging.lua** — Logging utilities with log level control.
  - **config.lua** — Enhanced configuration loading and validation.
  - **peripherals.lua** — Peripheral management, AE2 cache, relay helpers.
  - **state.lua** — Centralized state management with observer pattern and reset functions.
  - **pedestal.lua** — Pedestal rendering and management.
  - **ui.lua** — Basalt UI creation and updates.
  - **screens.lua** — Screen rendering logic.
  - **events.lua** — Event handling and state transitions.
  - **payment.lua** — Payment detection and idle timeout monitoring.
  - **dispense.lua** — Item dispensing from ME interfaces to barrel. Runs as 4th parallel coroutine.

The modules use dependency injection to avoid circular dependencies and enable testing.

### Flow of Screens

The shop operates as a five‑screen state machine with Screen 3 having two sub‑states:

1. **Category selection** — Welcome screen. Pedestals show category icons (iron, certus quartz, redstone, dye). Right‑click selects a category. No cancel button. Idle timeout does not apply.

2. **Material selection** — Shows only materials belonging to the chosen category that have sufficient stock in AE2. Right‑click selects a material, left‑click returns to category selection. Cancel button visible.

3. **Quantity selection and payment** — Two sub‑states:
   - **3A: Selecting a quantity** — Pedestals show available quantity tiers (from `minQty` up to AE2 stock). Right‑click chooses a quantity, left‑click returns to material selection. Cancel button visible.
   - **3B: Awaiting payment** — Triggered immediately after quantity selection. Selected pedestal label changes to `"[<qty>]"`. Depositor is configured and unlocked. Monitor shows calculated price and insert instruction. Cancel button visible.

4. **Dispensing progress** — Progress bar and counter as items move from ME interfaces to barrel in batches of 64. No cancel button. DispenseMonitorLoop handles progression.

5. **Thank‑you screen** — Shows "Purchase complete. Thank you!" and auto‑returns to screen 1 after `CONFIRM_DELAY` (3s). No cancel button.

### Peripheral Integration

- **Display pedestals** (Pedestals mod) — Show items/labels, receive `pedestal_left_click`/`pedestal_right_click` events.
- **AE2 adapter** (`ae2cc_adapter`) — Queries available stock for material items.
- **Numismatics depositor** — Accepts spur currency, signals payment via redstone relay.
- **Speaker** (`speaker_212`) — Plays harp sound for selection confirm (right‑click) and bass sound for cancel/back actions (left‑click, cancel button).
- **Monitor** — Shows Basalt UI with header, hint line, and cancel button.
- **Barrel** (`minecraft:barrel_59`) — Receives dispensed items; Item Storage API peripheral.
- **ME Interfaces** (list in config) — Stock items configured by player, drained via `pushItem()`.

### Concurrency

The main orchestrator runs four coroutines in parallel using `parallel.waitForAny`:
- `basalt.run()` — Basalt UI main loop.
- `events.eventLoop` — Listens for pedestal click events and handles state transitions.
- `payment.paymentMonitorLoop` — Monitors payment detection and idle timeouts.
- `dispense.dispenseMonitorLoop` — Handles item dispensing from ME interfaces to barrel.

## Key Data Structures

### Pedestal List (`config.lua`)
```lua
PEDESTALS = {
  "display_pedestal_12",  -- leftmost
  "display_pedestal_10",
  "display_pedestal_11",
  "display_pedestal_5",
  "display_pedestal_9",
  "display_pedestal_7",   -- rightmost
}
```
The script centers the currently active options across all pedestals; unused pedestals are cleared.

### Categories (`items.lua`)
```lua
CATEGORIES = {
  { label = "Metals",   item = "minecraft:iron_ingot" },
  { label = "Crystals", item = "ae2:certus_quartz_crystal" },
  { label = "Redstone", item = "minecraft:redstone" },
  { label = "Dyes",     item = "minecraft:blue_dye" },
}
```

### Materials (`items.lua`)
```lua
MATERIALS = {
  {
    label    = "Iron Ingot",
    item     = "minecraft:iron_ingot",
    category = "Metals",       -- must match a CATEGORIES label exactly
    minQty   = 64,             -- first quantity option shown; must be in QUANTITIES
    basePrice = 10,            -- price in spurs for minQty units
  },
  ...
}
```

### Quantity Tiers (`items.lua`)
```lua
QUANTITIES = { 1, 8, 32, 64, 256, 512, 1024, "4k", "16k", "32k" }
```
Helpers `quantityToNumber(qty)` and `findQuantityIndex(num)` convert between string/numeric representations.

### State (`modules/state.lua`)
The state is managed by the `state.lua` module, which provides controlled access via `state.getState()`, `state.updateState(changes)`, `state.resetState()`, and `state.resetToMainScreen()` functions. `resetToMainScreen()` is the preferred way to reset to the main screen (clears selections and payment fields while preserving `lastActivity` and pedestal state). The state structure remains:

```lua
local state = {
    screen = 1,               -- 1=category, 2=materials, 3=quantity/payment, 4=dispensing, 5=thankyou
    subState = nil,           -- nil, "selecting", or "confirming" (screen 3 only)
    selectedCategory = nil,
    selectedMaterial = nil,
    selectedQty = nil,
    calculatedPrice = nil,
    lastActivity = os.clock(),
    currentOptions = {},      -- pedestal index → option table { item, label, qty/category/material }
    currentPedestalIndices = {}, -- which pedestal indices are currently used
    lastSelectedPedestal = nil, -- last selected pedestal index
    availableQuantities = nil, -- list of numeric quantities available for selected material
    paymentBaseline = nil,    -- baseline relay input state for payment detection
    paymentDeadline = nil,    -- os.clock() deadline for payment timeout
    paymentPaid = false,
    paymentCheckCount = 0,    -- counter for payment detection checks
    dispensedCount = nil,     -- items dispensed so far
    dispensedTotal = nil,     -- total items to dispense
}
```

## Dependencies

- **Basalt** — UI framework, must be present on the ComputerCraft computer (`require("basalt")`).
- **CC:Tweaked built‑ins** — `peripheral`, `fs`, `os`, `parallel`, `colors`, `textutils`.
- **Mod peripherals** (must be attached and named correctly):
  - `display_pedestal` (Pedestals mod)
  - `ae2cc_adapter` (AE2 CC Bridge)
  - `Numismatics_Depositor` (Numismatics)
  - `redstone_relay` (CC:Tweaked redstone integration)
  - `minecraft:barrel_59` and `ae2:interface_*` (dispensing)

## Configuration Files

### `config.lua`
- Peripheral name constants (`RELAY_LOCK`, `AE2_ADAPTER`, `DEPOSITOR`, `SPEAKER_NAME`, `MONITOR`, `PEDESTALS`, `BARREL`, `ME_INTERFACES`).
- Dispensing constants (`DISPENSE_BATCH_SIZE`, `INTERFACE_PULL_DELAY`, `INTERFACE_RETRY_DELAY`, `INTERFACE_MAX_RETRIES`).
- Timing constants (`IDLE_TIMEOUT`, `PAYMENT_TIMEOUT`, `CONFIRM_DELAY`, `DISPENSE_DELAY`).
- UI message table (`MSG`) with required keys:
  ```lua
  MSG = {
    header           = "* SHOP *",
    screen1_hint     = "Right-click a pedestal to select a category",
    screen2_hint     = "RMB: select   LMB: go back",
    screen3_hint_select  = "RMB: select quantity   LMB: go back",
    screen3_base_price   = "Base price: %d spurs for %s units",
    screen3_price_calc   = "Price: %d spurs",
    screen3_insert       = "Please insert %s into the depositor",
    screen4_dispensing    = "Dispensing...",
    screen4_progress      = "%d/%d  %d%%",
    screen5_thanks        = "Purchase complete. Thank you!",
    cancel_btn       = "CANCEL",
    error_ae2        = "AE2 network unavailable",
    error_deposit    = "Depositor unavailable",
    error_relay      = "Relay unavailable",
    timeout_msg      = "Session timed out. Returning to main screen.",
  }
  ```
- `validatePeripherals()` – attempts to wrap each peripheral; called at startup.

### `items.lua`
- `CATEGORIES`, `MATERIALS`, `QUANTITIES` tables.
- Helper functions for quantity conversion and lookup (`quantityToNumber`, `findQuantityIndex`).
- Required structure:
  ```lua
  -- Categories
  CATEGORIES = {
    { label = "Metals",   item = "minecraft:iron_ingot"          },
    { label = "Crystals", item = "ae2:certus_quartz_crystal"     },
    { label = "Redstone", item = "minecraft:redstone"            },
    { label = "Dyes",     item = "minecraft:blue_dye"            },
  }

  -- Materials
  MATERIALS = {
    {
      label     = "Iron Ingot",
      item      = "minecraft:iron_ingot",
      category  = "Metals",   -- must match a CATEGORIES label exactly
      minQty    = 64,         -- first quantity option shown; must exist in QUANTITIES
      basePrice = 10,         -- price in spurs for minQty units
    },
    -- add more...
  }

  -- Quantity tiers (ordered smallest → largest)
  -- Allowed values: 1, 8, 32, 64, 256, 512, 1024, "4k", "16k", "32k"
  QUANTITIES = { 1, 8, 32, 64, 256, 512, 1024, "4k", "16k", "32k" }
  ```

### `db.lua`
- `log(record)` – appends a purchase record in ndjson format.
- `readAll()` – reads all logged purchases.

## Pedestal API Notes

The `display_pedestal` peripheral from the Pedestals mod **does not have a `setLabel` method**. Instead, use `setItem(item, label)` with the label as the second argument. (This was discovered and fixed in commit `d3fa13f`; earlier versions incorrectly called `setLabel`.)

The code uses `pcall(pedestals[idx].setItem, opt.item, label)` and wraps `setLabelRendered` calls in `pcall` because that method may or may not exist.

Example from `setPedestalOptions`:
```lua
if label then
    local ok, err = pcall(pedestals[idx].setItem, opt.item, label)
else
    local ok, err = pcall(pedestals[idx].setItem, opt.item)
end
```

## Cancel button behaviour

| Screen | Cancel button visible | Cancel action |
|---|---|---|
| 1 – Category | No | — |
| 2 – Material | Yes | Return to Screen 1 |
| 3A – Quantity selecting | Yes | Return to Screen 1 |
| 3B – Awaiting payment | Yes | Lock depositor, return to Screen 1 |
| 4 – Dispensing | No | — |
| 5 – Thank you | No | — |

- Button position: top‑left corner of monitor.
- Label: `"[ CANCEL ]"`.

## Idle timeout behaviour

- Applies to: Screens 2, 3A, 3B.
- Duration: 120 s of no user interaction.
- Action: if on 3B → lock depositor first; then return to Screen 1.
- `state.lastActivity = os.clock()` is reset on every user interaction (pedestal click, cancel press).

## Event Handling

The `eventLoop` listens for `pedestal_left_click` and `pedestal_right_click` events. Each event carries:
- The pedestal object or name (as `eventData[2]`).
- A table describing the item in hand (`eventData[3]`), containing `name`, `count`, `displayName`.

The script maps the pedestal object/name to an index using `pedestalObjectToIndex` and `pedestalIndexByName` lookups, then retrieves the corresponding option from `state.currentOptions`.

## Adding a New Pedestal

1. Add the peripheral name to the `PEDESTALS` array in `config.lua`.
2. The script will automatically center options across all defined pedestals; no other changes are needed.

## Adding a New Category or Material

1. Add a category entry to `CATEGORIES` in `items.lua` (label and representative item).
2. Add material entries to `MATERIALS` with the correct `category` label, `minQty`, and `basePrice`.
3. Ensure the `item` field matches the exact in‑game item ID (e.g., `"minecraft:iron_ingot"`).
4. The material will appear in the UI only when its stock in AE2 is at least `minQty`.

## Debugging

- The script writes detailed logs to `/ccshop/shop_debug.log` via `writeLog()`.
- Logs include peripheral wrapping results, stock queries, event data, and state transitions.
- The debug file is appended each run; clear it manually if it grows too large.
- Purchase records are stored in `/ccshop/purchases.json` (ndjson format).


## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---
> Source: [t8know-dev/ccshop](https://github.com/t8know-dev/ccshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
