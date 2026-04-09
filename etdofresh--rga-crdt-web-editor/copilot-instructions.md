## rga-crdt-web-editor

> Subject Line [Required]


# Git Commit Message Rules

## Format 

Subject Line [Required]

- Detail Line 1 [Optional]
- Detail Line 2 [Optional]
- ...  

## Rules 
- Use imperative mood 
- Present active tense 
- Start with a capital letter 
- Start with a verb (Add, Update, Remove, Fix, etc) 
- Does not include prefix [fix:, feat:, chore:, refactor:, etc] 
- Rules apply to subject line and detail lines 
- Does not require details, but if change is larger, include it  
- Do not include backticks before the commit message 
- Do not include backticks after the commit message 
- The first character should be a capital letter
- Include an extra new line between the subject line and detail lines
- Each detail line should start with a dash (-) 

## Examples  

### Example1 
Update API POST endpoint to support dynamic paths and improve URL construction 

### Example2
Rename PlayerController to CharacterController for clarity and consistency 

### Example3
Move PlayerController to CharacterController for better organization 

### Example4
Remove unused assets and clean up project structure 

### Example5 
Enhance ColyseusManager and GameRoom for improved room management and connection handling.

- Update ColyseusManager to utilize roomCode from Discord API or URL query parameters for dynamic room joining. 
- Modify GameRoom to store and log roomCode in metadata for better matchmaking and debugging. 
- Ensure fallback behavior for roomCode when not provided, enhancing user experience during connection attempts.  

### Example6 
Add UICanvas prefab and metadata for UI layout management.

- Introduce UICanvas prefab to manage the user interface layout. 

### Example7 
Refactor PlayerController and CharacterCreatorUI to streamline character appearance updates.

- Introduce UpdateColyseusCharacterAppearance method in PlayerController for better code organization. 
- Update CharacterCreatorUI to call the new method after saving character options, ensuring consistent state updates across the application.  

# Changes
{changes}

# IMPORTANT
- Do not include backticks before the commit message 
- Do not include backticks after the commit message

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ETdoFresh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ETdoFresh)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
