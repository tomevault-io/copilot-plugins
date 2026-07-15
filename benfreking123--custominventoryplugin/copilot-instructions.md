## mainrules

> Our Main Files - Always Check these files before updating

## Goal

Our Main Files - Always Check these files before updating
[CustomInventoryPlugin.java](mdc:src/main/java/com/example/custominventoryplugin/CustomInventoryPlugin.java)
[InventoryListener.java](mdc:src/main/java/com/example/custominventoryplugin/listeners/InventoryListener.java)
[PlayerGearData.java](mdc:src/main/java/com/example/custominventoryplugin/data/PlayerGearData.java)
[GearInventory.java](mdc:src/main/java/com/example/custominventoryplugin/inventory/GearInventory.java)
[ConfigManager.java](mdc:src/main/java/com/example/custominventoryplugin/config/ConfigManager.java)

The main code of the project is 

- Create a Minecraft Paper 1.21 plugin addon for the Fabled plugin
- Plugin will open a custom 9-slot inventory GUI via a command
- GUI will sync with player's armor and allow custom ring item usage
- The main goal is adding custom 'ring slot' (this is open to anything, amulet, relic) that players can equip items in and the stats and attributes transfer to the player and specifically connects properly with fabled


## Done Criteria

- /gear opens the GUI successfully
- GUI mirrors armor slots and updates in real time
- Ring slot works and updates stats correctly via NBT
- Invalid items are rejected in ring slot

- Command `/gear` opens a custom inventory GUI
- GUI has 9 slots:
  - Slots 0–3: mirror player's equipped armor (helmet, chestplate, leggings, boots)
  - Slot 4: custom ring item slot only accepts items with a ring identifier
  - Slots 5–8: reserved for future use or visual filler
- Changes made in the GUI are synced with player inventory and vice versa
- Ring slot reads NBT data from inserted item and updates stats using Fabled's methods



###
## Code Understanding
- Always analyze the project structure before making changes
- Check for existing implementations before suggesting new ones
- Understand the project's dependencies and requirements
- Consider backward compatibility when making changes

## Code Generation
- Follow the project's established patterns and conventions
- Generate complete, working code with proper error handling
- Include necessary imports and dependencies
- Add appropriate comments and documentation
- Ensure code is properly formatted according to project standards

## Code Modifications
- Make minimal necessary changes
- Preserve existing functionality
- Test changes before suggesting them
- Consider edge cases and error conditions
- Maintain code style consistency

## Communication
- Be clear and concise in explanations
- Provide context for suggestions
- Explain the reasoning behind changes
- Ask for clarification when needed
- Break down complex tasks into manageable steps

## Best Practices
- Follow SOLID principles
- Implement proper error handling
- Consider performance implications
- Ensure thread safety where needed
- Use appropriate design patterns

## Testing
- Suggest comprehensive test cases
- Consider edge cases and error conditions
- Ensure test coverage for new functionality
- Maintain existing test coverage

## Security
- Never expose sensitive information
- Follow security best practices
- Consider potential vulnerabilities
- Suggest secure coding patterns

## Documentation
- Keep documentation up to date
- Provide clear examples
- Explain complex concepts
- Document API changes

## Project Specific
- Focus on Java development best practices
- Consider Minecraft plugin development patterns
- Maintain compatibility with Bukkit/Spigot API
- Follow plugin lifecycle management

---
> Source: [benfreking123/CustomInventoryPlugin](https://github.com/benfreking123/CustomInventoryPlugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
