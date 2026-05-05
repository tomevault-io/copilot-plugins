## soliditydiagram

> A VS Code extension that generates interactive Miro-style diagrams for Solidity smart contracts. When invoked on a function, it displays:

# Solidity Diagram Extension

## Project Overview
A VS Code extension that generates interactive Miro-style diagrams for Solidity smart contracts. When invoked on a function, it displays:
- The main function code
- Referenced data structures (structs, enums)
- State variable declarations
- Inner function calls with their implementations
- Visual arrows connecting references to definitions

## Architecture

```
src/
├── extension.ts              # VS Code extension entry point
├── parser/
│   ├── solidityParser.ts     # Solidity AST parsing using @solidity-parser/parser
│   └── astTraverser.ts       # AST walking utilities
├── analyzer/
│   ├── functionAnalyzer.ts   # Main analysis orchestrator
│   ├── typeResolver.ts       # Resolves struct/enum definitions
│   ├── callGraphBuilder.ts   # Builds function call graph
│   ├── stateVariableResolver.ts  # Resolves state variable declarations
│   ├── dataFlowAnalyzer.ts   # Data flow tracking for DeFi code review
│   └── inheritanceResolver.ts # Interface/inheritance resolution & library method tracking
├── renderer/
│   ├── webviewProvider.ts    # VS Code webview panel management
│   ├── diagramGenerator.ts   # Generates HTML diagram
│   ├── canvasController.ts   # Miro-style pan/zoom canvas
│   ├── draggableBlocks.ts    # Drag functionality for code blocks
│   ├── arrowManager.ts       # Dynamic arrow connections
│   ├── syntaxHighlight.ts    # Solidity syntax highlighting
│   ├── importManager.ts      # Dynamic Cmd+Click import functionality
│   ├── dataFlowVisualizer.ts # Data flow hover/click visualization
│   └── notesManager.ts       # Text annotations/notes on the canvas
├── types/index.ts            # TypeScript type definitions
└── utils/sourceMapper.ts     # Source code mapping utilities
```

## Key Technologies
- **@solidity-parser/parser**: Parses Solidity into AST
- **VS Code Webview API**: Renders interactive HTML diagrams
- **Vanilla JS Canvas**: Custom pan/zoom/drag implementation (no external libs)

## Commands
- `npm install` - Install dependencies
- `npm run compile` - Build TypeScript
- `npm run watch` - Watch mode for development
- Press **F5** in VS Code to debug the extension

## Usage
1. Open a `.sol` file
2. Right-click inside a function
3. Select "Generate Function Diagram"
4. Interact with the diagram:
   - **Pan**: Drag the dotted background
   - **Zoom**: Mouse wheel
   - **Move blocks**: Drag the header (title bar) of any block
   - **Scroll code**: Hover over code area and scroll
   - **Resize blocks**: Drag corner or edges to resize
   - **Navigate**: Click "Go to source" to jump to code
   - **Import**: Cmd+Click (Mac) or Ctrl+Click (Windows/Linux) on highlighted tokens to import definitions
   - **Remove**: Click the X button on non-main blocks to remove them

## What Gets Displayed
- **Main Function**: The selected function with full source code
- **Data Structures**: Structs and enums used anywhere in the function, resolved from ALL workspace files:
  - Variable declarations: `DepositPool memory pool_`
  - Struct instantiation: `DepositPool({...})`
  - Enum comparisons: `strategy_ == Strategy.NO_YIELD`
- **State Variables**: Contract state variables referenced in the function (importable via Cmd+Click)
- **Internal Calls**: Only functions with actual definitions in the workspace

## Interface & Library Resolution
The extension now resolves interface calls to actual implementations:

### Interface Calls
- **Pattern Detection**: Recognizes `IContractName(address).methodName()` patterns with nested parentheses
- **Multiple Implementations**: Shows picker dialog when multiple implementations exist
- **Workspace Search**: Searches all `.sol` files for contracts implementing the interface
- **Inheritance Tracking**: Follows `is InterfaceName` declarations to find implementations

### Library Extension Methods
- **Using Directives**: Parses `using LibraryName for Type` statements (global and contract-level)
- **Method Resolution**: Resolves calls like `token.safeApprove()` to `SafeERC20.safeApprove()`
- **Context-Aware**: Uses calling contract's scope to find correct library attachments
- **Workspace + Dependencies**: Searches both workspace and `node_modules`

### External Dependencies
The extension scans `node_modules` for common Solidity libraries:
- **OpenZeppelin Contracts**: `@openzeppelin/contracts/**/*.sol`
- **Solmate**: `solmate/src/**/*.sol`
- **Solady**: `solady/src/**/*.sol`
- **Forge-std**: `forge-std/src/**/*.sol`

### Implementation Picker
When multiple implementations exist for an interface call:
1. User Cmd+Clicks on the method name
2. Modal dialog shows all implementations with:
   - Contract name and file path
   - Contract kind badge (contract, abstract, library)
   - Location in workspace or dependencies
3. User selects desired implementation
4. Code block is imported and displayed

## What Gets Excluded (After Resolution Attempts)
- Interface calls without found implementations (shows signature only)
- Type casts: `address(0)`, `uint256(value)`
- Built-in calls: `require()`, `keccak256()`, `abi.encode()`
- External library static calls without definitions

## Dynamic Import Feature
Tokens in the code are highlighted as importable (dotted underline on hover):
- **Functions**: Click to import function definitions
- **Types**: Click to import struct/enum definitions
- **Variables**: Click to import the type definition of typed variables
- **State Variables**: Click to import state variable declarations
- **Interface Calls**: Click on method names in patterns like `IERC20(token).approve()` to import implementations

### Interaction Priority
The extension properly handles conflicting interactions:
- **Cmd+Click / Ctrl+Click**: Always triggers import functionality (takes precedence)
- **Regular Click**: Triggers data flow highlighting (if enabled)
- This allows both features to coexist without conflicts

## Data Flow Tracking (DeFi Code Review)
On-demand visualization of how data flows through functions. Designed for DeFi protocol auditing.

### Usage
- **Hover** on any variable to see where it's defined and used
- **Click** on a variable to lock the visualization for detailed inspection
- Press **ESC** or click elsewhere to dismiss

### What Gets Tracked
- **Definitions**: Function parameters, local variable declarations
- **Uses**: Where variables are read/referenced
- **Sinks**: External calls, state writes, return statements, event emissions
- **Flow edges**: How data moves from definitions to uses to sinks

### DeFi-Specific Patterns
Variables are tagged with visual indicators based on their semantic meaning:
- **Token Amounts** (orange dotted): `amount`, `value`, `shares`, `balance`, `fee`, `reward`, etc.
- **msg.value** (pink solid): Native ETH value
- **msg.sender** (green solid): Transaction caller
- **Address Targets** (cyan dotted): `to`, `recipient`, `pool`, `vault`, `router`, etc.
- **Balance Checks** (purple dotted): Variables involved in balance comparisons

### Tooltip Information
When hovering/clicking a variable, the tooltip shows:
1. Variable name and DeFi tag (if applicable)
2. Where it's defined (line numbers, parameter/local/state)
3. Where it's used (line numbers)
4. What sinks it flows to (external calls, state writes, returns)
5. DeFi risk hints (e.g., "Flows to external call - verify validation")

### Key Files
- `dataFlowAnalyzer.ts` - Core analysis: extracts definitions, uses, edges, sinks
- `dataFlowVisualizer.ts` - Client-side hover/click handlers and tooltip rendering
- `syntaxHighlight.ts` - Adds `data-var` and `data-defi-tag` attributes to tokens

## Annotations Feature
Add text annotations directly on the diagram canvas for code review.

### Notes (Sticky Notes)
- **Double-click** on empty canvas area to create a note
- **Ctrl/Cmd + N** to create a note at center of viewport
- Click the **"+ Note"** button in the controls
- **Drag** notes by their header
- **Resize** notes by dragging the textarea
- **Delete** notes with the × button
- **Color** notes using the color buttons (yellow, green, blue, pink)
- **Arrow to code**: Click ↗ button, then click a code line to connect

### Labels (Compact Text Annotations)
- **Ctrl/Cmd + L** to create a label at center of viewport
- Click the **"+ Label"** button in the controls
- Labels are compact one-line annotations
- **Drag** labels by the ⋮⋮ handle or body
- **Arrow to code**: Click ↗ button, then click a code line to connect
- **Background colors**: Blue, Red, Green, Orange
- **Text size**: A- / A+ buttons to adjust font size (8px - 32px)
- **Text color**: T buttons for White, Black, or Yellow text

### Annotation Arrows
- Arrows from notes/labels to code lines are **responsive to zoom/pan**
- Arrows automatically update position when canvas is transformed
- Dashed lines with colored arrowheads matching the annotation color

Both notes and labels can have arrows pointing to specific code lines.
Annotations are saved to localStorage and persist across sessions.

## Arrow Hover Effect
Arrows connecting code blocks have a **glitter/glow effect** on hover:
- Hover over any arrow to see it glow and pulse
- Makes it easier to trace connections between code blocks
- Color-coded by type (function=pink, struct=cyan, enum=green, statevar=orange)

## Code Style
- TypeScript with strict mode
- Inline JavaScript/CSS generation for webview (no external files in webview)
- Catppuccin/GitHub dark theme colors
- No external UI frameworks - pure DOM manipulation

## Key Files to Modify
- `canvasController.ts` - Canvas behavior, CSS styles, implementation picker UI
- `diagramGenerator.ts` - HTML structure, block layout
- `arrowManager.ts` - Arrow routing and positioning, glitter hover effect
- `syntaxHighlight.ts` - Code syntax colors, data flow token attributes, interface call detection
- `importManager.ts` - Dynamic import logic (client-side), interface call parsing, implementation picker
- `stateVariableResolver.ts` - State variable resolution
- `dataFlowAnalyzer.ts` - Data flow analysis, DeFi pattern detection
- `dataFlowVisualizer.ts` - Data flow tooltip and highlighting, interaction priority handling
- `notesManager.ts` - Text notes/annotations feature, responsive arrows
- `inheritanceResolver.ts` - Interface-to-implementation mapping, library method resolution, inheritance chains
- `webviewProvider.ts` - Webview communication, dependency scanning, implementation selection
- `solidityParser.ts` - AST parsing, `using` directive extraction, base contract tracking

## Color Palette
- Background: `#0d1117`
- Block background: `#161b22`
- Block header: `#21262d`
- Border: `#30363d`
- Primary blue: `#58a6ff`
- Function arrows: `#f38ba8` (pink)
- Struct arrows: `#89dceb` (cyan)
- Enum arrows: `#a6e3a1` (green)
- State variable arrows: `#fab387` (orange/peach)
- Keywords: `#cba6f7` (purple)

### Data Flow Colors
- Definition highlight: `#89b4fa` (blue)
- Use highlight: `#fab387` (orange)
- Sink highlight: `#f38ba8` (pink)
- Token amount tag: `#fab387` (orange)
- msg.value tag: `#f38ba8` (pink)
- msg.sender tag: `#a6e3a1` (green)
- Address target tag: `#89dceb` (cyan)
- Balance tag: `#cba6f7` (purple)

### Note Colors
- Yellow: `#fef3c7` → `#fde68a` gradient
- Green: `#d1fae5` → `#a7f3d0` gradient
- Blue: `#dbeafe` → `#bfdbfe` gradient
- Pink: `#fce7f3` → `#fbcfe8` gradient

## Technical Implementation Details

### Interface Call Parsing
The extension uses a character-by-character parser (not regex) to handle complex nested patterns:
- **Nested Parentheses**: `IERC20(address(token_)).approve(...)` 
- **String Literals**: Ignores parentheses inside strings
- **Multiple Nesting**: Handles arbitrary nesting depth
- **Type Casting Chains**: `IVault(address(pool)).getAsset()`

Algorithm:
1. Scan for capital-I prefix (interface naming convention)
2. Find matching parenthesis using stack-based approach
3. Extract interface name and method call
4. Ignore if method is a type cast or builtin

### Inheritance Resolution Algorithm
```
findAllImplementations(interfaceName, methodName, contextContract):
  1. Search libraries attached to interfaceName via 'using' in context
  2. Search contracts that directly inherit interfaceName
  3. Fallback: search all contracts that have methodName
  4. Return all found implementations
```

### Library Method Resolution
```
findLibraryMethods(typeName, methodName, contextContract):
  1. Get libraries attached to typeName in contextContract scope
  2. For each library, check if it defines methodName
  3. Return library functions that match
```

### Dependency Scanning
The extension scans `node_modules` with specific glob patterns:
- `**/node_modules/@openzeppelin/contracts/**/*.sol`
- `**/node_modules/solmate/src/**/*.sol`
- `**/node_modules/solady/src/**/*.sol`
- `**/node_modules/forge-std/src/**/*.sol`

This is done once per diagram generation and cached.

---
> Source: [0xrudra99/SolidityDiagram](https://github.com/0xrudra99/SolidityDiagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
