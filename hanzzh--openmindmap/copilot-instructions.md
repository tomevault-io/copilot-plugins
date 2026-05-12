## openmindmap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is **openMindMap** - a modular TypeScript-based community plugin for the Obsidian note-taking application that renders markdown files as interactive mind maps using D3.js. The plugin automatically detects files starting with `#mindmap` and replaces the standard markdown editor with a hierarchical D3-based visualization.

**Current Version**: 1.0.0
**Main Language**: TypeScript 5.9.3
**Target**: Obsidian API (minimum app version 0.15.0)
**Key Features**: AI-powered node suggestions, mobile support, AES-256 API key encryption

## Development Commands

### Core Development Workflow
```bash
npm run dev              # Development mode with watch for live reloading
npm run build            # Production build with TypeScript validation + esbuild bundling
npm run rebuild          # Clean rebuild using rebuild.sh script
npm run version          # Automated version bump (updates manifest.json and versions.json)
```

### Testing Commands
```bash
npm run build:core       # Build core modules only (for testing)
npm run test             # Run overlap verification tests (requires build:core)
npm run test:unit        # Run unit tests (requires build:core)
```

### Build Process
The build system uses a two-step process:
1. **TypeScript Validation**: `tsc -noEmit -skipLibCheck` for type checking
2. **Esbuild Bundling**: Bundles `src/main.ts` into `main.js` with proper externals

Key configuration files:
- `esbuild.config.mjs` - Main build configuration
- `esbuild.test.config.mjs` - Test build configuration
- `tsconfig.json` - TypeScript compiler configuration

### Development Notes
- Modular architecture with separated services, handlers, and renderers
- Development mode watches for changes and rebuilds automatically
- Built `main.js` is loaded directly by Obsidian during development
- Source maps are generated in development mode (disabled in production)
- Configuration system supports desktop and mobile devices through early branching pattern

## Architecture Overview

### Device Detection Strategy
The plugin implements an **early branching pattern** for device support:
1. Device detection happens once during plugin initialization (`main.ts:onload()`)
2. `ConfigManager` selects either `DesktopConfig` or `MobileConfig` based on device type
3. Configuration is immutable after creation - no runtime device checking
4. Ensures desktop and mobile code paths never mix
5. Respects user setting: can override auto-detection with explicit 'desktop' or 'mobile' choice

### Modular Plugin Structure
The plugin follows a modular service-oriented architecture with clear separation of concerns:

```
MindMapPlugin (extends Plugin)
├── Plugin lifecycle management
├── ConfigManager (device detection & config selection)
├── MindMapService (central coordination layer)
├── AIClient (AI-powered node suggestions)
├── View registration (MIND_MAP_VIEW_TYPE)
├── Command palette integration
└── Settings management with encryption

MindMapView (extends ItemView)
├── View state management (getState/setState)
├── RendererManager (desktop/mobile renderer selection)
├── InteractionManager (desktop/mobile interaction selection)
└── Theme integration

Configuration Layer
├── ConfigManager (central config access)
├── DesktopConfig (desktop-specific settings)
└── MobileConfig (mobile-specific settings)

Service Layer
├── MindMapService (central coordination)
├── D3FileHandler (file operations)
├── LayoutCalculator (position calculations)
└── StateHandler (view state management)

Handler Layer
├── D3FileHandler (file I/O and markdown parsing)
├── InteractionHandler (base interaction handling)
├── DesktopInteraction (desktop-specific interactions)
├── MobileInteraction (mobile-specific interactions)
└── StateHandler (state persistence)

Renderer Layer
├── RendererManager (renderer selection based on device)
├── DesktopTreeRenderer (desktop D3.js visualization)
├── MobileTreeRenderer (mobile D3.js visualization)
├── NodeRenderer (individual node rendering)
└── LayoutCalculator (position and layout calculations)

Utility Layer
├── AIClient (OpenAI-compatible API client)
├── EncryptionUtil (AES-256 API key encryption)
├── AIPrompts (AI prompt template management)
├── FontSizeManager (dynamic font sizing)
└── CoordinateSystem (spatial calculations)
```

### Core Components

#### 1. Plugin Class (`MindMapPlugin`)
Located in `src/main.ts`
- **Device Detection**: Single point of device type determination (respects user setting or auto-detects)
- **Configuration Management**: Creates `ConfigManager` with detected device type
- **Service Coordination**: Initializes `MindMapService` with config and AI client
- **AI Integration**: Manages `AIClient` with encrypted API key storage
- **File Detection**: Listens for `file-open` events to detect `#mindmap` identifier
- **View Replacement**: Uses `setViewState()` to replace markdown editor with mind map
- **Command Integration**: Ribbon icon and command palette access
- **Settings Management**: Handles plugin settings with API key encryption/decryption
- **Lifecycle Management**: Proper load/unload handling

#### 2. Configuration System (`ConfigManager`)
Located in `src/config/config-manager.ts`
- **Early Branching**: Device detection happens once at initialization
- **Immutable Config**: Configuration object is frozen after creation
- **Type Safety**: Provides strongly typed configuration access
- **Desktop/Mobile Separation**: `DesktopConfig` and `MobileConfig` classes are completely separate
- **Centralized Access**: Single source of truth for all device-specific settings

#### 3. Service Layer (`MindMapService`)
Located in `src/services/mindmap-service.ts`
- **Central API**: Provides high-level interface for all mind map operations
- **Module Coordination**: Coordinates between handlers, renderers, and utilities
- **File Operations**: Delegates to `D3FileHandler` for file I/O
- **AI Integration**: Provides AI-powered node suggestions through `AIClient`
- **Data Management**: Manages `MindMapData` structures and conversions

#### 4. AI System (`AIClient` + `EncryptionUtil`)
Located in `src/utils/ai-client.ts` and `src/utils/encryption.ts`
- **OpenAI-Compatible API**: Supports any OpenAI-compatible endpoint
- **Context-Aware Suggestions**: Analyzes node hierarchy, parent, siblings, and existing children
- **Customizable Prompts**: System message and prompt template are user-configurable
- **AES-256 Encryption**: API keys encrypted using Web Crypto API
- **Device-Bound Keys**: Encryption keys derived from device-specific info (PBKDF2)
- **Secure Storage**: Encrypted keys stored in `data.json`, never plain text

#### 5. View Class (`MindMapView`)
Located in `src/main.ts`
- **State Management**: Implements `getState()` and `setState()` for proper Obsidian integration
- **Renderer Selection**: Uses `RendererManager` to select desktop/mobile renderer
- **Interaction Selection**: Uses `InteractionManager` to select desktop/mobile interactions
- **Theme Integration**: Maintains Obsidian theme consistency

#### 6. Renderer System
- **RendererManager**: Selects appropriate renderer based on device config
- **DesktopTreeRenderer**: Desktop-optimized D3.js visualization with mouse interactions
- **MobileTreeRenderer**: Mobile-optimized D3.js visualization with touch interactions
- **NodeRenderer**: Handles individual node rendering and styling
- **LayoutCalculator**: Computes node positions and tree layout

#### 7. Interaction System
- **InteractionManager**: Selects appropriate interaction handler based on device config
- **DesktopInteraction**: Mouse-based interactions (click, drag, zoom, pan)
- **MobileInteraction**: Touch-based interactions (tap, pinch, scroll)

#### 8. File Processing Pipeline
```typescript
// File processing flow:
1. File detection via MindMapService.isMindMapFile()
2. Content loading through D3FileHandler.loadFileContent()
3. Markdown parsing using parseMarkdownContent() utility
4. Hierarchical MindMapData structure creation
5. Device-specific renderer selection via RendererManager
6. D3.js rendering via DesktopTreeRenderer or MobileTreeRenderer
```

### Key Technical Features

#### Device Support Architecture
- **Early Branching Pattern**: Device type determined once at initialization, never changes
- **Strategy Pattern**: Separate `DesktopConfig` and `MobileConfig` classes
- **Manager Pattern**: `RendererManager` and `InteractionManager` select appropriate implementations
- **No Runtime Checks**: No `if (isMobile)` checks scattered throughout code
- **User Override**: Setting allows forcing desktop/mobile mode regardless of actual device

#### AI-Powered Node Suggestions
- **OpenAI-Compatible**: Works with OpenAI, Anthropic, local models, or any compatible API
- **Context-Aware**: Analyzes node text, level, parent, siblings, and existing children
- **Template-Based**: User-customizable system message and prompt template
- **Smart Variables**: Supports `{nodeText}`, `{level}`, `{parentContext}`, `{siblingsContext}`, `{existingChildren}`, `{centralTopic}`
- **Flexible Adding**: Add individual suggestions or all at once

#### Security & Encryption
- **AES-GCM 256-bit**: Industry-standard encryption for API keys
- **PBKDF2 Key Derivation**: 100,000 iterations with SHA-256
- **Device-Bound Keys**: Encryption key derived from device-specific info (user agent + language)
- **Fixed Salt**: Ensures same key generated on same device
- **Secure Storage**: Only encrypted keys stored in `data.json`
- **Not Portable**: Encrypted data cannot be decrypted on other devices

#### D3.js Integration
- **Tree Visualization**: Interactive D3.js tree layout with smooth transitions
- **Zoom and Pan**: Full canvas navigation with mouse/touch support
- **Dynamic Layout**: Automatic node positioning and collision detection
- **Themed Rendering**: Adapts to Obsidian's color scheme automatically
- **Device-Optimized**: Separate renderers for desktop (mouse) and mobile (touch)

#### View State Management
The plugin implements proper Obsidian view state handling:
- **Multi-level fallback**: Instance variable → view state → active file detection
- **Lifecycle timing**: Handles asynchronous `setState()` calls after `onOpen()`
- **Error resilience**: Multiple backup mechanisms for file path resolution
- **State Persistence**: Saves node expansion states and view positions

#### Data Structure Design
- **Hierarchical Nodes**: Tree-based `MindMapNode` structure with parent-child relationships
- **Flat Indexing**: `allNodes` array for O(1) node lookup and manipulation
- **Type Safety**: Comprehensive TypeScript interfaces for all data structures
- **Serialization**: Markdown round-trip conversion with format preservation

## File Structure

```
openMindMap (obsidian-mindmap-plugin)/
├── src/
│   ├── main.ts                         # Main plugin entry point and lifecycle
│   ├── interfaces/
│   │   └── mindmap-interfaces.ts       # TypeScript interfaces and types
│   ├── services/
│   │   └── mindmap-service.ts          # Central coordination service
│   ├── handlers/
│   │   ├── file-handler.ts             # File I/O and markdown parsing
│   │   ├── interaction-handler.ts      # Base interaction handling
│   │   └── state-handler.ts            # View state management
│   ├── renderers/
│   │   ├── renderer-manager.ts         # Renderer selection strategy
│   │   ├── d3-tree-renderer.ts         # Base D3.js visualization (deprecated)
│   │   ├── desktop-tree-renderer.ts    # Desktop-specific renderer
│   │   ├── mobile-tree-renderer.ts     # Mobile-specific renderer
│   │   ├── node-renderer.ts            # Individual node rendering
│   │   └── layout-calculator.ts        # Position and layout calculations
│   ├── interactions/
│   │   ├── interaction-manager.ts      # Interaction selection strategy
│   │   ├── desktop-interaction.ts      # Desktop-specific interactions
│   │   └── mobile-interaction.ts       # Mobile-specific interactions
│   ├── config/
│   │   ├── types.ts                    # Configuration type definitions
│   │   ├── config-manager.ts           # Central config manager
│   │   ├── desktop-config.ts           # Desktop configuration
│   │   └── mobile-config.ts            # Mobile configuration
│   ├── utils/
│   │   ├── mindmap-utils.ts            # Utility functions and helpers
│   │   ├── ai-client.ts                # OpenAI-compatible API client
│   │   ├── encryption.ts               # AES-256 encryption utilities
│   │   ├── ai-prompts.ts               # AI prompt templates
│   │   ├── font-size-manager.ts        # Dynamic font sizing
│   │   └── coordinate-system.ts        # Spatial calculations
│   └── constants/
│       └── mindmap-constants.ts        # Configuration constants
├── test/
│   ├── test-data.md                    # Sample mindmap for testing
│   ├── test-connection-accuracy.html   # Connection accuracy test
│   ├── test-overlap.html               # Overlap verification test
│   ├── README.md                       # Testing documentation
│   └── run-tests.sh                    # Test runner script
├── main.js                             # Compiled bundle (generated)
├── styles.css                          # Plugin styles with theme variables
├── manifest.json                       # Plugin metadata and compatibility
├── package.json                        # Dependencies and build scripts
├── tsconfig.json                       # TypeScript configuration
├── esbuild.config.mjs                  # Main build configuration
├── esbuild.test.config.mjs             # Test build configuration
├── versions.json                       # Version compatibility mapping
├── rebuild.sh                          # Clean rebuild script
├── README.md                           # User documentation (Chinese)
├── TROUBLESHOOTING.md                  # Detailed debugging guide
└── CLAUDE.md                           # This file - development guidance
```

## Dependencies and Tech Stack

### Core Dependencies
- **TypeScript 5.9.3**: Primary development language with ES6 target
- **Obsidian API 1.11.0**: Plugin framework and view system
- **D3.js 7.9.0**: Data visualization library for interactive tree rendering
- **@types/d3 ^7.4.3**: TypeScript definitions for D3.js
- **tslib 2.8.1**: TypeScript runtime library

### Development Dependencies
- **Esbuild 0.27.2**: Fast bundling and compilation
- **@types/node ^25.0.3**: TypeScript definitions for Node.js APIs
- **builtin-modules ^5.0.0**: Node.js builtin modules for bundling

### Build Configuration
- **Bundling**: Esbuild with external Obsidian modules
- **Target**: ES2018 for compatibility
- **Format**: CommonJS for Obsidian compatibility
- **Externals**: Obsidian, Electron, CodeMirror, and Lezer modules
- **Source Maps**: Inline in development, disabled in production
- **TypeScript Config**: ESNext modules, ES6 target, strict mode

## Development Guidelines

### Code Organization Principles
- **Early Branching**: Device type determined once at initialization, never changes at runtime
- **Strategy Pattern**: Use `ConfigManager`, `RendererManager`, `InteractionManager` to select implementations
- **Service-Oriented Design**: `MindMapService` provides central coordination
- **Interface-Driven Development**: Comprehensive TypeScript interfaces
- **Separation of Concerns**: Clear boundaries between file I/O, rendering, and interaction
- **Event-driven Architecture**: File detection via workspace events

### Device Support Guidelines
When adding device-specific functionality:
1. **Never check device type at runtime** - use the config object
2. **Create separate classes** for desktop/mobile (e.g., `DesktopRenderer` vs `MobileRenderer`)
3. **Use manager classes** to select appropriate implementation based on `config.isMobile`
4. **Keep implementations completely separate** - no shared code paths
5. **Test both modes** even if you're only on one device (use the settings override)

Example:
```typescript
// ❌ WRONG - Runtime device checking
if (Platform.isMobile) {
    // mobile code
} else {
    // desktop code
}

// ✅ CORRECT - Early branching via manager
const renderer = new RendererManager(config).getRenderer();
renderer.render(); // Uses correct implementation automatically
```

### Module Interaction Patterns
- **Service Layer**: `MindMapService` coordinates all operations
- **Configuration**: Access device-specific settings via `ConfigManager`
- **Handler Pattern**: Specialized classes for file operations and interactions
- **Renderer Strategy**: Pluggable rendering system with desktop/mobile implementations
- **Manager Pattern**: Use managers to select appropriate device-specific implementations

### Plugin Development Best Practices
- **Service Coordination**: Use `MindMapService` as central API entry point
- **View State Management**: Always implement `getState()` and `setState()` for custom views
- **Async Handling**: Proper async/await patterns for file operations
- **Error Boundaries**: Comprehensive error handling with user feedback
- **Theme Integration**: Use Obsidian CSS variables for consistent styling
- **D3.js Patterns**: Follow D3.js data-binding and update patterns
- **Security**: Never store API keys in plain text - always use `EncryptionUtil`

### AI Integration Guidelines
- **OpenAI-Compatible**: Design for any OpenAI-compatible API, not just OpenAI
- **Error Handling**: Handle API errors gracefully with user-friendly messages
- **Context Awareness**: Provide rich context (node text, level, parent, siblings)
- **Customization**: Allow users to customize prompts and system messages
- **Validation**: Validate API responses structure before using

### Testing Approach
- **Visual Testing**: Test mindmap rendering with `test/test-data.md`
- **Connection Testing**: Use `test/test-connection-accuracy.html` for connection accuracy
- **Overlap Testing**: Use `test/test-overlap.html` to verify no node overlaps
- **Manual Testing**: Test both desktop and mobile modes (use settings override)
- **API Testing**: Use built-in "Test Connection" button for AI configuration

## Plugin Lifecycle

### Initialization Flow
1. **Load Settings**: Restore plugin configuration from `data.json` (decrypt API key if needed)
2. **Device Detection**: Determine device type (auto-detect or user override)
3. **ConfigManager Initialization**: Create `ConfigManager` with device type
4. **AIClient Initialization**: Create `AIClient` with API configuration
5. **Service Initialization**: Create `MindMapService` with config and AI client
6. **Register View**: Register custom `mind-map-view` type
7. **Add UI Elements**: Ribbon icon, status bar, commands
8. **Setup Event Listeners**: File-open detection via service layer
9. **Layout Ready Check**: Convert existing `#mindmap` files when workspace is ready

### Settings Management Flow
1. **Load on Startup**: Read settings from `data.json`
2. **Decrypt API Key**: If encrypted, use `EncryptionUtil.decrypt()` to retrieve API key
3. **Provide to AI Client**: Pass decrypted API key to `AIClient`
4. **User Updates**: When user changes settings, validate then encrypt before saving
5. **Test Connection**: Use `AIClient.testConnection()` to verify API configuration

### File Conversion Process
1. **Detection**: File-open event triggers `MindMapService.isMindMapFile()` check
2. **Service Coordination**: `replaceWithMindMapView()` called with file
3. **View Creation**: New `MindMapView` instance created
4. **State Setting**: File path passed via `setState()`
5. **Content Loading**: `MindMapService.loadMindMapData()` parses markdown
6. **Renderer Selection**: `RendererManager` selects desktop/mobile renderer
7. **Interaction Selection**: `InteractionManager` selects desktop/mobile interactions
8. **D3 Rendering**: Appropriate renderer creates interactive visualization

### AI Suggestion Flow
1. **Node Selection**: User selects a node in the mindmap
2. **Context Collection**: Gather node text, level, parent, siblings, existing children
3. **Prompt Construction**: Use `AIPrompts.buildSuggestionPrompt()` with context
4. **API Call**: `AIClient.getSuggestions()` sends request to configured API
5. **Response Validation**: Validate JSON response structure
6. **UI Display**: Show suggestions in modal/popup
7. **User Action**: Add individual or all suggestions as child nodes
8. **File Update**: Save updated mindmap to file

## Common Development Tasks

### Adding Device-Specific Features
1. Create separate implementations in `src/config/desktop-config.ts` and `src/config/mobile-config.ts`
2. Add to the `MindMapConfig` interface in `src/config/types.ts`
3. Access via `config.yourSetting` - no runtime checks needed
4. Document the setting in `README.md` if user-visible

Example:
```typescript
// src/config/types.ts
export interface MindMapConfig {
    isMobile: boolean;
    newNodePosition: { x: number; y: number };  // Device-specific
}

// src/config/desktop-config.ts
export class DesktopConfig {
    getConfig(): MindMapConfig {
        return {
            isMobile: false,
            newNodePosition: { x: 100, y: 100 }  // Desktop values
        };
    }
}

// src/config/mobile-config.ts
export class MobileConfig {
    getConfig(): MindMapConfig {
        return {
            isMobile: true,
            newNodePosition: { x: 50, y: 50 }   // Mobile values
        };
    }
}
```

### Adding New Features
- **Service Layer**: Add methods to `MindMapService` for new functionality
- **Rendering**: Create `src/renderers/desktop-yourfeature.ts` and `src/renderers/mobile-yourfeature.ts`
- **Interactions**: Create `src/interactions/desktop-yourfeature.ts` and `src/interactions/mobile-yourfeature.ts`
- **File Processing**: Extend `D3FileHandler` for new file formats
- **UI Elements**: Add commands in plugin `onload()` method
- **Interfaces**: Update TypeScript interfaces in `mindmap-interfaces.ts`
- **Styling**: Update CSS in `styles.css` for visual changes
- **Configuration**: Update `manifest.json` for API changes

### Testing Changes
```bash
npm run dev                          # Start development mode
# Test in Obsidian by opening a #mindmap file
# Test both desktop and mobile modes (use Device Type setting)
```

### Debugging Issues
- Enable browser developer tools in Obsidian (Ctrl+Shift+I)
- Check console for detailed error logging from all modules
- Use the Device Type setting to test both desktop and mobile modes
- Refer to `TROUBLESHOOTING.md` for common issues
- Check `data.json` for encrypted API key storage
- Test AI functionality with "Test Connection" button

### Version Management
- Use `npm run version` for automated version bumping
- Updates both `manifest.json` and `versions.json`
- Follow semantic versioning principles
- Update `README.md` for feature changes

## Current State and Future Development

### Implemented Features (v1.0.0)
- ✅ Modular service-oriented architecture
- ✅ D3.js interactive tree visualization
- ✅ Automatic `#mindmap` file detection
- ✅ Complete markdown editor replacement
- ✅ Interactive node expansion/collapse
- ✅ Theme-integrated styling with D3.js
- ✅ Command palette and ribbon integration
- ✅ Comprehensive error handling and debugging
- ✅ Zoom and pan navigation
- ✅ State persistence for node expansion
- ✅ **AI-powered node suggestions (OpenAI-compatible)**
- ✅ **AES-256 API key encryption**
- ✅ **Mobile device support with touch interactions**
- ✅ **Device-specific configuration system**
- ✅ **Customizable AI prompts**

### Architecture Strengths
- **Separation of Concerns**: Clear module boundaries
- **Device Support**: Early branching pattern ensures clean separation
- **Security**: Industry-standard encryption for sensitive data
- **Testability**: Individual modules can be tested in isolation
- **Extensibility**: New renderers and interactions easily added
- **Type Safety**: Comprehensive TypeScript coverage
- **Performance**: Efficient D3.js rendering and updates
- **AI Integration**: Flexible, customizable AI system

### File Format Requirements
- Files must start with exactly `#mindmap` (no preceding characters)
- Use standard markdown list syntax (`*`, `-`, or `+`)
- **4 spaces per indentation level** (critical for parsing)
- UTF-8 encoding required
- Does NOT support markdown heading syntax (`#`, `##`) for nodes

### Important Implementation Notes

#### Encryption & Security
- API keys are encrypted using AES-GCM 256-bit before storage
- Encryption keys are device-specific (derived from user agent + language)
- Encrypted data CANNOT be decrypted on other devices
- Uses PBKDF2 with 100,000 iterations for key derivation
- Always use `EncryptionUtil` for sensitive data, never store plain text

#### Device Support
- Device type is determined ONCE at plugin initialization
- Use the "Device Type" setting to override auto-detection
- Desktop and mobile implementations are completely separate
- Never use `Platform.isMobile` checks outside of initialization
- Always use `config.isMobile` to access device type

#### AI Integration
- AI client is OpenAI-compatible, not OpenAI-specific
- Supports any API that follows OpenAI's format
- API responses are validated for structure before use
- Prompt templates support rich context variables
- Error handling includes user-friendly messages

#### D3.js Considerations
- D3 selections are managed through renderer lifecycle
- Zoom and pan state is persisted in view state
- Node positions are calculated by `LayoutCalculator`
- Theme colors are extracted from Obsidian CSS variables
- Desktop and mobile use separate renderer implementations

### Future Development Opportunities
- 🎨 Enhanced styling and customization options
- ✏️ Inline node editing capabilities
- 📤 Export functionality (SVG, PNG formats)
- 🔗 Node link support for Obsidian connections
- 🔄 Real-time collaboration features
- 🎯 Advanced layout algorithms (force-directed, circular)
- 🌐 Multi-language AI prompt templates
- 📊 Usage analytics and metrics

## Important Notes

### Development Workflow
1. **Always use `npm run dev`** for development - it auto-rebuilds on changes
2. **Test both device modes** using the "Device Type" setting, even if you're only on one device
3. **Never store API keys in plain text** - always use `EncryptionUtil`
4. **Check the console** for detailed debug logging when troubleshooting
5. **Use the early branching pattern** for device-specific code

### Configuration Access
- Get config from `ConfigManager` instance (passed to most classes)
- Access settings via `this.config.yourSetting`
- Never check `Platform.isMobile` outside of `main.ts:onload()`
- Configuration is immutable after creation

### Service Integration
- All file operations go through `MindMapService` for consistency
- Handlers are injected with dependencies and coordinated through services
- Renderers are stateless and receive data through service layer
- AI operations go through `AIClient` with encrypted API keys
- State management is centralized through `StateHandler`

### Obsidian Integration
- Fully replaces markdown editor for detected files
- Maintains Obsidian's theme consistency through CSS variable extraction
- Integrates with workspace layout system
- Supports standard Obsidian view operations
- Proper view state management for workspace persistence
- Device-specific renderers selected via `RendererManager`

### Performance Considerations
- D3.js rendering is efficient for large datasets
- Service layer provides caching for repeated operations
- Memory usage scales with node count but is optimized
- Layout calculations use efficient tree algorithms
- Consider implementing virtualization for very large mind maps (>1000 nodes)
- Mobile devices may have stricter performance limitations

### Security Best Practices
- **Always encrypt** sensitive data before storing in `data.json`
- **Never log** API keys or other sensitive information
- **Validate** all user inputs before processing
- **Use HTTPS** for all API communications
- **Implement rate limiting** for API calls (user-configurable)

---
> Source: [Hanzzh/openMindMap](https://github.com/Hanzzh/openMindMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
