## justquest

> **WoW Quest Automation Addon** | Lua (WoW API) | Interface 110205

# JustQuest AI Agent Instructions

**WoW Quest Automation Addon** | Lua (WoW API) | Interface 110205

## Architecture Overview

```
JustQuest.toc (load order)
├── Core/Constants.lua    → Static config, TIMING table, DEFAULTS
├── Core/Core.lua         → Module registry, event dispatch, dependency injection
├── Core/Config.lua       → AceDB wrapper, Get()/Set() API
├── Services/Services.lua → Timers (AceTimer), SafeCall, Debug, frame helpers
├── Services/GameData.lua → Item/quest data lookups
├── Engines/SafetyEngine  → STATE MACHINE (IDLE/QUEST/GOSSIP/HALT), whitelist/blacklist
├── Engines/QuestEngine   → Quest accept/turnin, GOSSIP_SHOW entry point
├── Engines/GossipEngine  → Dialog selection with safety validation
├── Engines/RewardEngine  → Quest reward analysis and selection
└── Engines/ContainerEngine → Auto-open reward containers
```

**State Flow:** `WoW Event → Core dispatch → SafetyEngine.SetState() → Engine handler → SafetyEngine.EndProcessing()`

## Critical Patterns

### Module Access (ALWAYS nil-check)
```lua
local config = JustQuest.Core.GetModule("Config")
local value = config and config.Get("section", "key") or DEFAULT

-- Module registration (file end)
JustQuest.Core.RegisterModule("ModuleName", Module, {"Config", "Services"})
```

### State Management
```lua
local safetyEngine = JustQuest.Core.GetModule("SafetyEngine")
safetyEngine.SetState("QUEST", "accepting_quest")  -- Start processing
safetyEngine.EndProcessing("quest_accepted")       -- Return to IDLE
safetyEngine.IsBusy()  -- true if QUEST/GOSSIP/CONFIRM/HALT
```

### Safe WoW API Calls
```lua
local success, result = pcall(C_QuestLog.GetQuestInfo, questID)
if not success then return nil end

-- Or use Services wrapper
local result = Services.SafeCall(C_Item.GetItemInfo, itemLink)
```

### Timer System (namespace keys to avoid collisions)
```lua
Services.ScheduleOnce("QuestEngine_retry", 0.25, function() ... end)
Services.CancelScheduled("QuestEngine_retry")
-- Timing constants: JustQuest.Constants.TIMING.SERVER_SYNC_DELAY
```

## Safety System (CRITICAL)

| System | Behavior | Use For |
|--------|----------|---------|
| **Whitelist** | BYPASS safety → immediate select | Known-safe options to auto-select |
| **Blacklist** | SKIP option (select next safe one) | Teleports, story skips, dangerous choices |
| **Colored Text** | HALT automation | Any non-default colored text requires manual review |
| **Repeat Protection** | Skip same option in TTL window | Prevents infinite loops |

**Safety Priority:** Whitelist > Blacklist/Colored Text > Normal Selection

### Blacklist Behavior (Updated)
- Blacklisted options are **skipped**, not halted
- Automation selects the first **safe** option instead
- Only halts if ALL options are unsafe (blacklisted, colored, or costly)
- Key functions: `SafetyEngine.IsBlacklisted()`, `ValidateInteraction()`

### Colored Text Detection (NEW)
- ANY colored text in gossip/quest options halts automation
- Color codes: `|cAARRGGBB`, `|cnCOLOR_NAME:`, `|cnIQn:`
- Plain text = safe, colored text = requires manual selection
- Key function: `HasAnyColorCode(text)`

### Repeat Protection Details
- `ShouldSkipRepeat(npcGUID, optionID, totalOptions)` prevents selecting same option twice in TTL window
- Essential for preventing infinite loops in story gossip chains
- TTL reduced for single-option menus (likely "back" navigation)
- Even whitelisted options respect repeat protection (prevents spam)

### ForceGossip Protection
- Story/lore NPCs use ForceGossip flag (gossip doesn't auto-close after selection)
- When `ignoreForceGossip=false`: automation halts on story moments
- When `ignoreForceGossip=true`: blacklist becomes the safety net for story content
- Key function: `SafetyEngine.ShouldHaltForForceGossip()`

### WoW API Text vs UI Text
- `C_GossipInfo.GetOptions()` returns **plain text** (no color codes)
- UI frames preserve colors: `frame:GetFontString():GetText()` has `|cnCOLOR_NAME:text|r`
- Color detection extracts from UI frames via `GetColoredGossipOptions()`
- `gossipOptionID` can be **nil since 10.0.7** - use `orderIndex` fallback

## Verification Workflow

1. **Search first:** `grep_search "FunctionName" includePattern="Engines/**"`
2. **Read targeted:** `read_file` ±30 lines around match
3. **Verify in-game:** Ask user to run `/script` commands
4. **Propose → confirm → implement**

### In-Game Debug Commands
```lua
/script local s=JustQuest.Core.GetModule("SafetyEngine"); print(s.GetState(), s.GetReason())
/script print(C_GossipInfo.GetNumOptions())
/script print(QuestFrame and QuestFrame:IsShown())
```

## Code Standards

- **Early returns** over deep nesting (max 3 levels)
- **No temporary debug code** in source files
- **Lua patterns** not regex: `string.find(str, "text", 1, true)`
- **No globals** except `JustQuest` namespace
- Combat lockdown blocks UI actions—check `InCombatLockdown()`

## Key Files by Task

| Task | Primary Files |
|------|---------------|
| Quest accept/turnin | `QuestEngine.lua` → OnQuestDetail, OnQuestComplete |
| Gossip selection | `GossipEngine.lua` → HandleGossip |
| Safety validation | `SafetyEngine.lua` → IsBlacklisted, IsWhitelisted, SetState |
| Reward selection | `RewardEngine.lua` → ProcessQuestReward |
| Container opening | `ContainerEngine.lua` → ProcessContainerItem |
| Timing constants | `Constants.lua` → TIMING table |
| Config read/write | `Config.lua` → Get, Set |

## Anti-patterns to Avoid

- Reading entire files instead of targeted searches
- Assuming API text has color codes (extract from UI frames)
- Assuming `gossipOptionID` exists (can be nil - use `orderIndex` fallback)
- Timer keys without module prefix (causes collisions)
- Deep nesting instead of early returns
- Adding `print()` debug statements (use `/script` verification)

---
> Source: [wealdly/JustQuest](https://github.com/wealdly/JustQuest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
