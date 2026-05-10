## stastarshipsimulator

> **Do NOT read or reference `STA2e_Core Rulebook_DIGITAL_v1.1.txt`** - This file is too large and not relevant for development. Use `starshiprules.md` and other extracted reference files instead.

# STA Starship Simulator

## Important: Ignored Files

**Do NOT read or reference `STA2e_Core Rulebook_DIGITAL_v1.1.txt`** - This file is too large and not relevant for development. Use `starshiprules.md` and other extracted reference files instead.

## Project Overview

A web-based starship operations and combat simulator for **Star Trek Adventures 2nd Edition** tabletop RPG. This app allows players and game masters to manage starship combat encounters digitally, with each participant accessing the app from their own device (tablet, phone, laptop).

## Core Features

### Player Features
- **Player Profiles**: Create and manage character profiles including:
  - Attributes (Control, Daring, Fitness, Insight, Presence, Reason)
  - Disciplines (Command, Conn, Engineering, Medicine, Science, Security)
  - Talents and special abilities
  - Character role/position on the ship (Captain, Helm, Tactical, Ops, Engineering, Science, Medical)

- **Ship Management**: Enter and track ship details:
  - Ship systems (Comms, Computers, Engines, Sensors, Structure, Weapons)
  - Ship departments (Command, Conn, Engineering, Medicine, Science, Security)
  - Scale, Resistance, crew complement
  - Weapons and special abilities
  - Talents

- **Action Interface**: On their turn, players can:
  - Select their action (major action + minor action)
  - Roll dice using the 2d20 system
  - See results and effects automatically calculated
  - Spend Momentum or add Threat as needed

### Game Master Features
- **Encounter Setup**: Configure combat scenarios including:
  - Enemy ships (stats, weapons, Scale)
  - Environmental zones (nebulae, asteroid fields, planetary bodies)
  - Terrain effects (difficult terrain, hazardous terrain)
  - Starting positions

- **NPC Management**: Control enemy vessels and NPCs
- **Threat Pool**: Manage the Threat pool
- **Override Controls**: Manually adjust values when needed

### Shared View Screen
A display mode for a shared screen/TV showing:
- **Tactical Map**: Zone-based map showing ship positions and terrain
- **Ship Status**:
  - Shield status (forward, aft, port, starboard)
  - Hull breaches and damage
  - System damage/disabled systems
  - Power allocation
- **Resource Pools**:
  - Momentum (group pool, max 6)
  - Threat (GM pool)
- **Turn Order**: Current initiative and whose turn it is

## Game Mechanics Reference

The app implements Star Trek Adventures 2e starship combat rules:

### Distance/Range System
- **Contact**: Docked/landed state
- **Close**: Same zone (0 zones)
- **Medium**: Adjacent zone (1 zone)
- **Long**: 2 zones away
- **Extreme**: 3+ zones away

### Dice System
- **2d20 System**: Roll 2d20, each die at or under target number = success
- **Target Number**: Attribute + Discipline
- **Complications**: Natural 20s generate complications
- **Critical Success**: Natural 1s count as 2 successes
- **Challenge Dice**: d6 for damage (1-2 = 1, 3-4 = 0, 5-6 = 1 + Effect)

### Key Resources
- **Momentum**: Earned from extra successes, spent for benefits (max 6 in pool)
- **Threat**: GM resource, players can add to gain benefits

### Ship Positions & Actions
Each bridge position has specific minor and major actions available:
- **Captain/Command**: Direct, Rally, Assist
- **Helm/Conn**: Maneuver, Evasive Action, Attack Pattern
- **Tactical/Security**: Fire Weapons, Lock On, Target Systems
- **Operations**: Transporters, Power Management, Sensors
- **Engineering**: Damage Control, Boost Power
- **Science**: Scan, Analysis, Modulate Shields

## Technical Architecture

### Stack (TBD - to be decided during implementation)
- Frontend: Web-based SPA (React/Vue/Svelte)
- Backend: Node.js or similar for real-time sync
- Database: For persistent player/ship data
- Real-time: WebSockets for live updates across devices

### Key Technical Requirements
- Mobile-responsive design (tablets are primary use case)
- Real-time synchronization between all connected clients
- Offline capability for dice rolling (nice to have)
- Session/room system for different game tables

## File Reference

- **GitHub Issues** - **Check this first!** Tracks planned features and current development priorities. Use `gh issue list` to see open issues, organized by milestones (Alpha, Beta, v1.0) and labels (station types, core-mechanics, etc.)
- `starshiprules.md` - Extracted starship combat rules from STA 2e Core Rulebook
- `STA2e_Core Rulebook_DIGITAL_v1.1.txt` - Full rulebook text (human reference only, DO NOT READ)
- `ADDING_ACTIONS.md` - Quick guide for adding new actions using the declarative system

## Action Framework (IMPORTANT!)

**The project now uses a declarative, configuration-based system for actions!**

### Why This Matters
Instead of writing ~100 lines of custom code per action, you can now add actions in **~10 lines of configuration** (30 seconds of work!).

### How to Add New Actions

**For simple buff actions** (instant effects, no roll required):
1. Add config to `sta/mechanics/action_config.py`
2. Add action name to `BUFF_ACTIONS` array in `sta/web/templates/combat.html`
3. Done! (~30 seconds)

**For task roll actions** (require a 2d20 roll):
1. Add config to `sta/mechanics/action_config.py`
2. They automatically work with the dice panel - no UI changes needed!
3. Done! (~30 seconds)

### Key Files
- `sta/mechanics/action_config.py` - Action configurations (10+ actions already defined)
- `sta/mechanics/action_handlers.py` - Generic execution handlers
- `sta/models/combat.py` - ActiveEffect model for tracking buffs
- `sta/web/routes/api.py` - Generic `/api/encounter/<id>/execute-action` endpoint
- `ADDING_ACTIONS.md` - Complete developer guide with examples

### Already Configured Actions (RFT)
These actions are already configured and ready to use:
- **Tactical**: Calibrate Weapons, Targeting Solution, Modulate Shields
- **Science**: Calibrate Sensors, Scan For Weakness, Sensor Sweep
- **Engineering**: Damage Control, Regain Power, Regenerate Shields
- **Conn**: Attack Pattern, Evasive Action, Maneuver
- **Command** (Beta): Rally

See `ADDING_ACTIONS.md` for detailed examples and patterns.

## Testing

**Run tests before pushing to GitHub!**

```bash
# Install test dependencies (first time only)
pip install -r requirements-dev.txt

# Run all tests
pytest

# Run with coverage report
pytest --cov=sta

# Run specific test file
pytest tests/test_turn_order.py

# Run specific test class
pytest tests/test_turn_order.py::TestTurnClaiming

# Run with verbose output
pytest -v
```

### Test Structure
- `tests/conftest.py` - Shared fixtures (test database, sample data, helpers)
- `tests/test_turn_order.py` - Turn claiming, passing, alternation, round advancement
- `tests/test_actions_tactical.py` - Tactical station actions
- `tests/test_actions_science.py` - Science station actions
- `tests/test_actions_conn.py` - Conn/Helm station actions
- `tests/test_actions_engineering.py` - Engineering station actions
- `tests/test_actions_command.py` - Command station actions
- `tests/test_actions_standard.py` - Standard actions (Pass, etc.)
- `tests/test_action_logging.py` - Combat log functionality

### Writing New Tests
When adding new actions, add corresponding tests that verify:
1. Action executes successfully
2. Action is correctly categorized as minor/major
3. Turn order changes appropriately
4. Action is logged to combat log

## Development Notes

When implementing features:
1. Refer to `starshiprules.md` for combat mechanics
2. Ask the user if you need clarification on rules edge cases
3. Prioritize the player turn experience - this is the core loop
4. The view screen should be visually appealing and readable from a distance
5. Keep the GM controls powerful but simple
6. **Run tests before pushing**: `pytest`
7. **After completing a feature**, add the `ready-for-testing` label to the GitHub issue: `gh issue edit <number> --add-label "ready-for-testing"`

## Future Considerations
- Character sheet integration (full character management, not just combat stats)
- Campaign management
- Combat log/history
- Sound effects for dramatic moments
- Printable summaries

---
> Source: [tommertron/staStarShipSimulator](https://github.com/tommertron/staStarShipSimulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
