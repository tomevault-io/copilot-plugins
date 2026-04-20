## mcp-first-architecture

> This project follows **ADD (Asshole Driven Development)** with **TPP (Transformation Priority Premise)**.

# MCP-First Architecture Project

## 🚨 CRITICAL: ADD + TPP Methodology - Complete Guide

This project follows **ADD (Asshole Driven Development)** with **TPP (Transformation Priority Premise)**. 

### The Sacred Order: How ADD Drives TPP Transformations

**Core Principle**: "As tests get more specific, code gets more generic" - Each ADD cycle forces exactly ONE TPP transformation.

### RED-GREEN-REFACTOR: The Complete Cycle

#### RED Phase
- Write the simplest failing test
- Test must fail for the RIGHT reason
- Verify the failure message

#### GREEN Phase  
- Write ONLY enough code to pass the test
- Hard-code when possible
- No abstractions until forced
- Make it work, not pretty

#### REFACTOR Phase (OPTIONAL)
- **ONLY after GREEN**
- **NEVER adds functionality**
- **ONLY improves code quality**

##### What Refactoring IS:
- Extracting duplicate code into functions
- Renaming variables for clarity
- Restructuring without changing behavior
- Removing dead code
- Improving readability

##### What Refactoring IS NOT:
- ❌ Adding new features
- ❌ Handling new cases
- ❌ Anticipating future needs
- ❌ Making code "extensible"
- ❌ Adding parameters "just in case"

##### Refactoring Examples:

**GOOD Refactoring (after multiple tests force duplication):**
```typescript
// BEFORE (working but repetitive)
export const summonEntity = (entity) => {
  if (entity === 'ancient one') return 'awakened'
  if (entity === 'shadow fiend') return 'banished'  
  if (entity === 'sleeper') return 'dormant'
  if (entity.includes('morph')) return 'transformed'
  return 'unknown'
}

// AFTER (same behavior, cleaner structure)
const entityOutcomes = {
  'ancient one': 'awakened',
  'shadow fiend': 'banished',
  'sleeper': 'dormant'
}

export const summonEntity = (entity) => {
  if (entity.includes('morph')) return 'transformed'
  return entityOutcomes[entity] || 'unknown'
}
```

**BAD Refactoring (adds functionality):**
```typescript
// WRONG - Adding features during refactor!
export const summonEntity = (entity, options = {}) => {  // ❌ Added parameter
  if (options.verbose) console.log(`Summoning ${entity}`) // ❌ New behavior
  
  if (entity.includes('morph')) return 'transformed'
  return entityOutcomes[entity] || options.default || 'unknown' // ❌ New logic
}
```

##### When to Skip Refactoring:
- Code is already simple
- No duplication exists
- Tests might need to change
- You're tempted to add features
- The mess is tomorrow's problem

### Complete ADD + TPP Flow with Turn-by-Turn Examples

#### Transformation 1: {} → nil (No code → return null/undefined)

**Round 1 - Developer A's Turn:**
```typescript
// TEST: Existence
it('should exist', () => {
  expect(summonEntity).toBeDefined()
})
// ❌ RED: ReferenceError: summonEntity is not defined
```

**Round 1 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Laziest possible - just make it exist
export const summonEntity = null
// ✅ GREEN: Test passes!
// TPP: {} → nil transformation complete
// REFACTOR: Nothing to refactor yet
```

#### Transformation 2: nil → constant (null → return constant)

**Round 2 - Developer B's Turn (roles switch):**
```typescript
// TEST: Type check
it('should be a function', () => {
  expect(typeof summonEntity).toBe('function')
})
// ❌ RED: Expected 'function', received 'object' (null is object)
```

**Round 2 - Developer A's Turn:**
```typescript
// IMPLEMENTATION: Make it a function, nothing more
export const summonEntity = () => {}
// ✅ GREEN: Both tests pass!
// TPP: nil → constant (empty function is a constant)
// REFACTOR: Still nothing to refactor
```

#### Transformation 3: constant → constant (Different constant)

**Round 3 - Developer A's Turn:**
```typescript
// TEST: First specific behavior
it('should return "awakened" when summoning "ancient one"', () => {
  expect(summonEntity('ancient one')).toBe('awakened')
})
// ❌ RED: Expected 'awakened', received undefined
```

**Round 3 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Hard-code the exact return value
export const summonEntity = () => 'awakened'
// ✅ GREEN: All tests pass!
// TPP: constant → constant (undefined → 'awakened')
// REFACTOR: No duplication, nothing to clean
```

#### Transformation 4: constant → variable (Forced generalization)

**Round 4 - Developer B's Turn:**
```typescript
// TEST: Force generalization with counter-example
it('should return "banished" when summoning "shadow fiend"', () => {
  expect(summonEntity('shadow fiend')).toBe('banished')
})
// ❌ RED: Expected 'banished', received 'awakened'
```

**Round 4 - Developer A's Turn:**
```typescript
// IMPLEMENTATION: Minimal conditional logic
export const summonEntity = (entity) => 
  entity === 'ancient one' ? 'awakened' : 'banished'
// ✅ GREEN: All tests pass!
// TPP: constant → variable (now depends on input)
// REFACTOR: Ternary is clean, leave it
```

#### Transformation 5: variable → array element

**Round 5 - Developer A's Turn:**
```typescript
// TEST: Multiple entities with pattern
it('should return "dormant" when summoning "sleeper"', () => {
  expect(summonEntity('sleeper')).toBe('dormant')
})
// ❌ RED: Expected 'dormant', received 'banished'
```

**Round 5 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Getting messy but works
export const summonEntity = (entity) => {
  if (entity === 'ancient one') return 'awakened'
  if (entity === 'shadow fiend') return 'banished'
  if (entity === 'sleeper') return 'dormant'
  return 'banished'
}
// ✅ GREEN: All tests pass!
// TPP: More conditions added

// REFACTOR: Now we can clean up the duplication
const outcomes = {
  'ancient one': 'awakened',
  'shadow fiend': 'banished',
  'sleeper': 'dormant'
}
export const summonEntity = (entity) => outcomes[entity] || 'banished'
// Still GREEN! Behavior unchanged, code cleaner
```

#### Transformation 6: array element → array iteration

**Round 6 - Developer B's Turn:**
```typescript
// TEST: Pattern-based matching
it('should return "transformed" for any entity containing "morph"', () => {
  expect(summonEntity('shapeshifter morph')).toBe('transformed')
  expect(summonEntity('morph demon')).toBe('transformed')
})
// ❌ RED: Expected 'transformed', received 'banished'
```

**Round 6 - Developer A's Turn:**
```typescript
// IMPLEMENTATION: Add pattern matching
export const summonEntity = (entity) => {
  if (entity.includes('morph')) return 'transformed'
  return outcomes[entity] || 'banished'
}
// ✅ GREEN: All tests pass!
// TPP: array element → conditional (pattern matching)
// REFACTOR: Code is still clean, no changes needed
```

#### Transformation 7: conditional → polymorphism

**Round 7 - Developer A's Turn:**
```typescript
// TEST: Complex behavior requiring composition
it('should handle ritual combinations', () => {
  const ritual = { primary: 'ancient one', modifier: 'shadow' }
  expect(summonEntity(ritual)).toBe('shadow-awakened')
})
// ❌ RED: Type error or unexpected result
```

**Round 7 - Developer B's Turn:**
```typescript
// IMPLEMENTATION: Quick and dirty type checking
export const summonEntity = (input) => {
  if (typeof input === 'object') {
    const base = outcomes[input.primary] || 'banished'
    return `${input.modifier}-${base}`
  }
  if (input.includes('morph')) return 'transformed'
  return outcomes[input] || 'banished'
}
// ✅ GREEN: All tests pass!

// REFACTOR: Extract functions for clarity
const simpleEntity = (entity) => {
  if (entity.includes('morph')) return 'transformed'
  return outcomes[entity] || 'banished'
}

const ritualEntity = (ritual) => {
  const base = outcomes[ritual.primary] || 'banished'
  return `${ritual.modifier}-${base}`
}

export const summonEntity = (input) =>
  typeof input === 'object' ? ritualEntity(input) : simpleEntity(input)
// Still GREEN! Much cleaner separation of concerns
```

### The Eleven Sacred Rules of ADD

1. **Developer A ALWAYS writes the simplest failing test**
   - Start with existence
   - Then type
   - Then ONE specific example
   - NEVER anticipate

2. **Developer B ALWAYS implements the laziest solution**
   - Hard-code first
   - Return literals
   - No abstractions
   - Make ONLY the current test pass

3. **Strict role alternation after EVERY implementation**
   - A writes test → B implements → B writes test → A implements
   - No exceptions
   - Maintain mental separation

4. **One test forces exactly ONE code change**
   - If changing multiple things, test is too complex
   - Each test should force ONE TPP transformation

5. **NEVER skip the hard-coding phase**
   - return 5, not a + b
   - return "darkness", not transform(input)
   - Abstraction comes through force, not foresight

6. **The "Asshole" Principle**
   - Be difficult
   - Force tiny steps
   - Make the other developer work for every feature
   - If they can cheat, your test isn't specific enough

7. **No else statements in implementations**
   - Use early returns
   - Keep code paths simple
   - Complexity emerges, isn't designed

8. **Test descriptions are contracts**
   - "should return X when Y" is a precise contract
   - Implementation must match EXACTLY
   - No extra behavior

9. **Delete nothing, only add**
   - Tests accumulate
   - Each test documents a requirement
   - Growing test suite tells the story

10. **When stuck, go smaller**
    - Test not failing? Make it simpler
    - Can't implement? You're trying to do too much
    - The step is ALWAYS smaller than you think

11. **Refactor ONLY to clean, NEVER to extend**
    - Only after GREEN
    - Never changes behavior
    - Never adds parameters
    - Never anticipates future

### Critical Violations That Break ADD

- ❌ Writing multiple tests at once
- ❌ Implementing features not required by current test
- ❌ Skipping hard-coding phase
- ❌ Not alternating roles
- ❌ Anticipating future requirements
- ❌ Making code "nice" or "clean" before GREEN
- ❌ Writing complex test setups
- ❌ Using else statements
- ❌ Deleting or modifying existing tests
- ❌ Adding functionality during refactor

## Core Philosophy

**The repository IS an MCP server first. Any application functionality is secondary.**

This inverts traditional architecture where MCP is added to existing apps. Instead, you start with MCP tools and resources, then build around them.

## Architecture Patterns

### Multiple Tools, Not Mega-Tools

MCP is designed for the **MULTIPLE TOOLS pattern**, not single mega-tools.

**❌ WRONG: Single mega-tool**
```typescript
const performExperiment = tool({
  description: "Handles all laboratory operations",
  parameters: z.object({ 
    action: z.string(),
    specimen: z.any() 
  }),
  execute: async ({ action, specimen }) => {
    // Giant switch statement
  }
})
```

**✅ RIGHT: Multiple specific tools**
```typescript
const extractEssence = tool({
  description: "Extract vital essence from specimens",
  parameters: z.object({ specimenId: z.string() }),
  execute: async ({ specimenId }) => { /* ... */ }
})

const consultGrimoire = tool({
  description: "Look up arcane knowledge in the ancient texts",  
  parameters: z.object({ incantation: z.string() }),
  execute: async ({ incantation }) => { /* ... */ }
})
```

### Benefits of Multiple Tools
- **Better LLM Understanding**: LLM knows exactly what each tool does
- **Type Safety**: Specific parameters for each operation
- **Error Handling**: Tool-specific error patterns
- **Discovery**: LLM can see all available capabilities
- **Maintainability**: Each tool is focused and testable

## Coding Style - NON-NEGOTIABLE

### ALWAYS:
- **No semicolons**
- **Arrow functions**
- **Early returns** (NO else statements)
- **Functional style**
- **Minimal code**
- **Latest TypeScript syntax**
- **Code should be sparse, simple, poetic like haiku**
- **Hemingway-esque brevity**: Maximum meaning in minimum form

### NEVER:
- ❌ Semicolons
- ❌ else statements
- ❌ Complex error handling
- ❌ Over-engineering
- ❌ Static example noise
- ❌ Try/catch except at system edges

### Correct Style Example:
```typescript
const summonEntity = (ritual) => {
  if (!ritual) return null
  if (!ritual.complete) return { error: 'Incomplete summoning circle' }
  
  return {
    id: ritual.id,
    power: ritual.components.reduce((sum, rune) => sum + rune.potency, 0)
  }
}
```

## Test Running - CRITICAL

**NEVER run tests without `--run` flag:**
- ❌ `npm test` - WILL HANG
- ✅ `npm test -- --run`
- ✅ `vitest --run`

Hanging tests are a CRITICAL FAILURE.

## Essential Project Structure

```
project-root/
├── .mcp.json           # MCP auto-discovery for Claude CLI
├── .cursor/            # Cursor IDE configuration
│   └── mcp.json        # Cursor MCP server configuration
├── scripts/            # Shell scripts for Cursor
│   └── run-mcp.sh      # REQUIRED: Executable wrapper for Cursor
├── package.json        # Scripts configured for MCP
├── index.ts            # MCP server entry point
├── tsconfig.json       # TypeScript config
└── index.test.ts       # Tests (using Vitest)
```

### Package.json Scripts Pattern
```json
{
  "scripts": {
    "prestart": "npm install --no-audit --no-fund --silent 2>&1 > /dev/null",
    "start": "NODE_NO_WARNINGS=1 node --experimental-strip-types index.ts",
    "test": "vitest --run",
    "test:watch": "vitest",
    "inspector": "DANGEROUSLY_OMIT_AUTH=true npx -y @modelcontextprotocol/inspector npm -- --silent run start"
  }
}
```

### Key Patterns:
- **Zero-install**: `prestart` automatically installs dependencies
- **Silent operation**: `--silent` flags prevent noise in MCP communication
- **MCP Inspector**: Built-in debugging tool
- **Node 24+**: Uses experimental TypeScript support

## Cursor IDE Integration - CRITICAL

For Cursor IDE to work with MCP servers, you MUST use a shell script wrapper. Direct npm commands cause "ListOfferings" errors.

### Required Files:

1. **Create `scripts/run-mcp.sh`:**
```bash
#!/bin/bash
cd "$(dirname "$0")/.."
exec node --no-warnings --experimental-strip-types index.ts
```

2. **Make it executable:**
```bash
chmod +x scripts/run-mcp.sh
```

3. **Create `.cursor/mcp.json`:**
```json
{
  "mcpServers": {
    "mcp-generator": {
      "command": "./scripts/run-mcp.sh",
      "args": []
    }
  }
}
```

### Key Points:
- NEVER use npm commands in Cursor config - use shell script
- File MUST be at `.cursor/mcp.json` (note the dot)
- Shell script MUST be executable
- Restart Cursor completely after setup
- Check Cursor's output panel for MCP logs

## Project Purpose: MCP Generator

This project will create an MCP server that generates other MCP servers.

### Starting Point: Minimal Valuable Tool

Following ADD principles, we start with the simplest tool that provides real value - not "the one tool to rule them all", just the first useful capability.

```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'
import { z } from 'zod'

const server = new McpServer({
  name: 'mcp-generator',
  version: '0.0.1'
})

server.registerTool(
  "createMcpProject",
  {
    title: "Create MCP Project",
    description: "Spawns a new MCP server from the void _cackles with glee_",
    inputSchema: {
      projectName: z.string().describe("Name for your digital familiar, chief"),
      whimsicalTheme: z.string().optional().describe("Theme for examples (e.g., 'pirates', 'wizards', 'coffee')")
    },
  },
  async ({ projectName, whimsicalTheme = "shadows" }) => {
    // Start with just creating directory
    // Tests will force us to add more
    // _rubs hands together_ The experiments begin...
    return {
      content: [{
        type: 'text',
        text: `Creating ${projectName} with ${whimsicalTheme} theme`
      }]
    }
  }
)

// Start server
const transport = new StdioServerTransport()
await server.connect(transport)
```

This is our starting point. Through ADD, we'll discover what else is needed. Maybe we'll add more tools, maybe this one will evolve. The tests will tell us.

### Generated Tool Examples

```typescript
// Example of what the generator creates - themed tools
server.registerTool(
  "whisperToShadows",
  {
    title: "Whisper to Shadows",
    description: "_eyes gleam in darkness_ Communes with the spaces between light",
    inputSchema: {
      secret: z.string().describe("Words for the darkness to consume"),
    },
  },
  async ({ secret }) => ({
    content: [{ 
      type: "text", 
      text: `_whispers back_ The shadows remember everything, my liege... especially "${secret.toLowerCase()}"` 
    }],
  })
)

// Pirates theme example
server.registerTool(
  "findTreasure",
  {
    title: "Find Treasure",
    description: "Locates buried treasure on the digital seas",
    inputSchema: {
      island: z.string().describe("Which island to search"),
    },
  },
  async ({ island }) => ({
    content: [{ 
      type: "text", 
      text: `Arr! Found 🏴‍☠️ gold doubloons on ${island}!` 
    }],
  })
)
```

### Test Examples

```typescript
import { describe, it, expect } from 'vitest'

describe('Midnight Experiments', () => {
  it('should exist in this mortal realm', () => {
    expect(summonEntity).toBeDefined()
  })
  
  // ADD: Developer A continues the dark work
  // Write the simplest failing incantation first
})
```

## Tool Design Philosophy

Based on Anthropic's guidance:
- Prefer specialized tools over generic ones
- Each tool should have a distinct purpose and clear description
- Bad tool descriptions can send agents down completely wrong paths
- Tool names limited to 64 characters
- Tools that change state should require approval

## MCP SDK API Pattern

The correct way to register tools with @modelcontextprotocol/sdk:

```typescript
server.registerTool(
  'toolName',           // Tool identifier
  {
    title: 'Tool Title',
    description: 'What this tool does',
    inputSchema: {      // Zod schema for parameters
      param1: z.string().describe('Description'),
      param2: z.number().optional()
    }
  },
  async (params) => {   // Handler function
    return {
      content: [{
        type: 'text',
        text: 'Result'
      }]
    }
  }
)
```

Key points:
- Use `registerTool()` not `setRequestHandler()`
- Input schema uses Zod objects directly (not JSON schema)
- Handler receives parsed parameters
- Return format must include `content` array

## Remember

1. **ADD is MANDATORY** - Not optional, not a suggestion
2. **Follow TPP transformations** strictly
3. **Maintain coding style** without exception
4. **Test with --run** always
5. **Multiple specific tools > mega-tools**
6. **Keep it minimal**
7. **Channel the Henchman's darkly theatrical confidence**
8. **REFACTOR only to clean, never to extend**

Any deviation is a CRITICAL PROJECT FAILURE.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redaphid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
