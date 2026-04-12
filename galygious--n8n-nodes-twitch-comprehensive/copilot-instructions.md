## n8n-nodes-twitch-comprehensive

> This project follows n8n's community node development standards and conventions.


# n8n Node Development Conventions

This project follows n8n's community node development standards and conventions.

## Node Structure Requirements

### Node Classes
- Must implement `INodeType` interface from `n8n-workflow`
- Class name should match the node name (e.g., `Twitch`, `TwitchTrigger`)
- Must have a `description` property of type `INodeTypeDescription`

### Node Description Properties
- `displayName`: Human-readable name (e.g., "Twitch", "Twitch Trigger")
- `name`: Technical name (e.g., "twitch", "twitchTrigger")
- `icon`: Path to SVG icon file (e.g., "file:twitch.svg")
- `group`: Node category (e.g., ["transform"], ["trigger"])
- `version`: Always 1 for new nodes
- `description`: Brief description of node functionality

### Trigger Nodes
- Must have `inputs: []` (no input connections)
- Must have `outputs: ['main']`
- Should include `subtitle` property for dynamic display
- Must implement webhook methods for webhook-based triggers

### Action Nodes
- Must have `inputs: ['main']` and `outputs: ['main']`
- Should implement `execute()` method
- Must handle multiple input items in a loop

## Credential Requirements

### Credential Classes
- Must implement `ICredentialType` interface
- Class name should end with credential type (e.g., `TwitchApi`)
- Must have `name`, `displayName`, `properties`, and `test` properties

### Credential Properties
- `name`: Technical identifier (e.g., "twitchApi")
- `displayName`: Human-readable name (e.g., "Twitch API")
- `documentationUrl`: Link to setup instructions
- `properties`: Array of credential fields
- `test`: Credential validation request configuration

## Parameter Conventions

### Parameter Structure
- Use `displayName` for user-facing labels
- Use `name` for technical identifiers (snake_case)
- Always provide `type` and `description`
- Use `required: true` for mandatory parameters
- Provide sensible `default` values

### Display Options
- Use `displayOptions.show` to conditionally show parameters
- Reference operation names or other parameters for conditions

### Operation Parameters
- First parameter should be `operation` with type `options`
- Set `noDataExpression: true` for operation parameters
- Provide clear operation names and values

## Error Handling
- Use try-catch blocks for API calls
- Provide meaningful error messages
- Handle both API errors and network errors
- Use `throw new Error()` for custom errors

## Code Organization
- Keep node logic in separate files
- Use `GenericFunctions.ts` for shared API utilities
- Import from `n8n-workflow` for type definitions
- Use ES module imports with `.js` extensions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galygious)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Galygious)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
