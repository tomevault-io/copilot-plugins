## addonprofiles

> - ✅ `profile.autoDeps`

# AddonProfiles - WoW Classic Addon

## Critical Gotchas

### Field Names
- ✅ `profile.autoDeps`
- ❌ `profile.autoDependencies` (wrong - will silently fail)

### Scope Checking
- ✅ Check BOTH: `activeName == name AND activeScope == scope`
- ❌ Only checking name will match across scopes incorrectly

### WoW Classic API
- ❌ `C_Timer.After()` - not available in Classic
- ✅ `ReloadUI()` - use directly after `ActivateProfile()`

### Confirmation Dialogs
- ❌ `StaticPopupDialogs` - causes main window hide/show issues
- ✅ `AceGUI:Create("Frame")` - reliable custom dialogs

### Search Box Focus
- Known limitation: AceGUI EditBox loses focus when `PopulateAddonList()` recreates widgets
- This is standard AceGUI behavior - accept it

## Database Structure
```lua
AddonProfiles.db.global.profiles = {}  -- Account-wide profiles
AddonProfiles.db.char.profiles = {}    -- Current character profiles
AddonProfiles.db.sv.char[charKey] = {} -- All characters (for cross-char viewing)
```

## UI Color System
- GREEN = Active/Applied profile
- GOLD = Selected for editing
- GRAY = Profile from another character (view only)
- WHITE = Normal profile

## Cross-Character Profile Access
```lua
if self.currentProfile.charKey then
    profile = AddonProfiles.db.sv.char[charKey].profiles[name]
else
    profile = AddonProfiles.ProfileManager:GetProfile(name, scope)
end
```

## UI Layout
- Three panels: `SetRelativeWidth(0.28)`, `0.44`, `0.28` (must sum to 1.0)
- Always call `container:DoLayout()` and `frame:DoLayout()` after populating
- Interface Version: 11508 (WoW 1.15.7)

## Testing
```bash
make test  # Run all 38 tests before committing
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmervine) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
