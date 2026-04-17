## node-dynamixel

> This is a Node.js library for controlling DYNAMIXEL servo motors via U2D2 interface with Protocol 2.0 support. The library provides both ESM and CommonJS builds, TypeScript definitions, and comprehensive Electron integration support.

# Cursor Rules for node-dynamixel

## Project Overview
This is a Node.js library for controlling DYNAMIXEL servo motors via U2D2 interface with Protocol 2.0 support. The library provides both ESM and CommonJS builds, TypeScript definitions, and comprehensive Electron integration support.

## Architecture & Code Organization

### Directory Structure
- `src/` - Source code (ES modules)
  - `dynamixel/` - Core DYNAMIXEL classes (Protocol2, DynamixelDevice, etc.)
  - `transport/` - Connection classes (SerialConnection, U2D2Connection, WebSerialConnection)
  - `utils/` - Utility classes (Logger)
  - `DynamixelController.js` - Main controller class
- `examples/` - Example scripts and usage demonstrations
- `tests/` - Test suite (unit and integration tests)
- `dist/` - Built files (generated, not in git)

### Key Classes & Patterns
- **DynamixelController**: Main entry point, handles device discovery and management
- **DynamixelDevice**: Individual motor control and monitoring
- **Protocol2**: DYNAMIXEL Protocol 2.0 implementation with CRC calculation
- **Transport Classes**: Connection abstractions (Serial, USB, WebSerial)
- **Event-driven**: Use EventEmitter patterns for connection state and errors

## Coding Standards

### JavaScript/ES Modules
- Use ES6+ features and ES modules (`import`/`export`)
- Prefer `const` over `let`, avoid `var`
- Use arrow functions for callbacks and short functions
- Use template literals for string interpolation
- Always use semicolons
- Use JSDoc comments for all public methods and classes

### Error Handling
- Always use try/catch for async operations
- Emit 'error' events on EventEmitter classes
- Provide descriptive error messages with context
- Use Protocol2.getErrorDescription() for DYNAMIXEL error codes

### Async Patterns
- Use async/await over Promise chains
- Always handle Promise rejections
- Use timeouts for hardware communication
- Clean up resources in finally blocks

### Buffer & Protocol Handling
- Always validate buffer lengths before parsing
- Use Protocol2.parseStatusPacket() for parsing raw responses
- Calculate CRC using Protocol2.calculateCRC()
- Handle both little-endian and big-endian data properly

## Development Workflow & Quality Assurance

### **MANDATORY Before Any Commit or Pull Request**
**Always run these commands in sequence - NO EXCEPTIONS:**

1. **Linting**: `npm run lint` (or `npm run lint:fix` to auto-fix)
2. **Building**: `npm run build` (verify TypeScript generation)
3. **Testing**: `npm test` (full test suite with --detectOpenHandles)
4. **Package Verification**: `npm pack --dry-run` (ensure package is valid)

### **Development Commands Reference**
```bash
# Primary workflow (run ALL of these before committing):
npm run lint          # Check code style and catch errors
npm run lint:fix       # Auto-fix linting issues where possible
npm run build          # Build ESM/CommonJS + TypeScript definitions
npm test               # Run all 284 tests with handle detection
npm pack --dry-run     # Verify package contents

# Individual test commands:
npm run test:unit      # Unit tests only
npm run test:integration  # Integration tests only
npm run test:coverage  # Tests with coverage report
npm run test:watch     # Watch mode for development

# Development helpers:
npm run build:watch    # Watch mode for building
npm run discovery      # Test device discovery
npm run diagnostics    # USB diagnostics
```

### **Code Quality Standards**
- **Zero linting errors** - Fix ALL ESLint warnings and errors
- **All tests must pass** - No exceptions, including integration tests
- **Clean builds** - TypeScript definitions must generate without errors
- **No hanging handles** - Jest must exit cleanly (--detectOpenHandles)
- **Package integrity** - npm pack must succeed without warnings

### **Pre-Commit Checklist**
- [ ] `npm run lint` passes with zero errors
- [ ] `npm run build` completes successfully
- [ ] `npm test` shows "284 passed" with no hanging handles
- [ ] `npm pack --dry-run` shows expected package contents
- [ ] `node examples/separated-discovery.js` works with real hardware (if available)
- [ ] Git status is clean (no untracked build artifacts)

## Testing Guidelines

### Test Structure
- Unit tests in `tests/unit/` for individual classes
- Integration tests in `tests/integration/` for real hardware scenarios
- Use Jest with ES modules support
- Mock hardware connections for unit tests

### **Testing + Linting Workflow**
**CRITICAL**: Always run linting before testing to catch code quality issues early:
```bash
# Correct workflow:
npm run lint           # Fix any linting errors first
npm run build          # Ensure build works
npm test               # Then run full test suite

# NEVER skip linting - it catches critical issues that tests might miss
```

### Test Patterns
- Use `createStatusPacketBuffer()` helper for mock DYNAMIXEL responses
- Always clean up connections in `afterEach` blocks
- Use `--detectOpenHandles` to ensure no resource leaks
- Test both success and error conditions

### Mock Guidelines
- Mock `sendAndWaitForResponse` to return proper status packet buffers
- Use Protocol2 to generate valid CRC checksums in test data
- Mock USB and serial port operations to avoid hardware dependencies

## Hardware Communication

### DYNAMIXEL Protocol 2.0
- All packets must have valid CRC-16 checksums
- Use proper packet structure: Header + ID + Length + Instruction + Data + CRC
- Handle status packets with error codes correctly
- Implement proper timeout handling (default 1000ms)

### Connection Management
- Support deferred connections for Electron apps
- Prioritize serial connections over USB connections
- Handle connection state events (connected, disconnected, error)
- Clean up resources on disconnect

### Device Discovery
- Use separated discovery pattern for UI applications
- Support both quick scan (IDs 1-20) and full scan (IDs 1-252)
- Handle device timeouts gracefully during discovery
- Return proper device information (model, firmware, etc.)

## Build & Distribution

### Dual Module Support
- Maintain both ESM (`index.esm.js`) and CommonJS (`index.cjs.js`) builds
- Use Rollup for building with proper externals
- Generate TypeScript declarations from JSDoc
- Test both module formats

### Package Management
- Keep dependencies minimal (only serialport required, usb optional)
- Use engines field to specify Node.js 18+ requirement
- Include proper files array in package.json
- Maintain semantic versioning

## Documentation

### JSDoc Standards
```javascript
/**
 * Brief description of the method
 * @param {type} paramName - Parameter description
 * @param {Object} options - Options object
 * @param {boolean} [options.optional=false] - Optional parameter with default
 * @returns {Promise<type>} - Return value description
 * @throws {Error} - When this error occurs
 * @example
 * const result = await method(param, { optional: true });
 */
```

### Code Comments
- Explain complex protocol logic and bit manipulations
- Document hardware-specific behaviors and limitations
- Add TODO comments for future improvements
- Use inline comments for non-obvious code sections

## Common Pitfalls to Avoid

### **Development Workflow Issues**
- **DON'T commit without linting** - Always run `npm run lint` first
- **DON'T skip the build step** - TypeScript definitions must be current
- **DON'T ignore test failures** - All 284 tests must pass
- **DON'T commit hanging tests** - Use --detectOpenHandles to verify clean exit

### Protocol Issues
- Don't access `response.error` on raw buffers - parse first with Protocol2.parseStatusPacket()
- Don't assume packet lengths - always validate with Protocol2.getCompletePacketLength()
- Don't forget CRC validation on received packets
- Don't mix up little-endian vs big-endian data

### Resource Management
- Always disconnect connections in cleanup code
- Don't leak USB device handles in tests
- Clear timeouts to prevent hanging
- Remove event listeners to prevent memory leaks

### Testing Issues
- Don't use real hardware connections in unit tests
- Don't forget to mock all async dependencies
- Don't skip error condition testing
- Don't ignore Jest open handle warnings

## Project-Specific Conventions

### File Naming
- Use PascalCase for class files (e.g., `DynamixelDevice.js`)
- Use kebab-case for example files (e.g., `separated-discovery.js`)
- Use descriptive names that match the main export

### Import/Export Patterns
```javascript
// Named exports for classes
export class DynamixelController extends EventEmitter { }

// Default export for utilities
export default class Logger { }

// Re-exports in index files
export { DynamixelController } from './DynamixelController.js';
```

### Event Naming
- Use lowercase with hyphens: 'device-discovered', 'connection-lost'
- Always document event payloads in JSDoc
- Emit events consistently across similar classes

## Version 0.0.5 Specific Notes
- All transport classes now properly handle buffer processing with safety limits
- Motor discovery uses proper packet parsing for reliable property reading
- Separated discovery pattern is the preferred approach for Electron apps
- TypeScript definitions are automatically generated and should match JSDoc exactly
- Test suite is comprehensive with 284 tests covering all major functionality

**REMEMBER**: Always run `npm run lint && npm run build && npm test` before committing. When making changes, ensure the separated-discovery example works correctly with real hardware.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andyshinn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
