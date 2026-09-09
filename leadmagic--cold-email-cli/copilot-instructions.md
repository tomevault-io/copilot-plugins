## typescript-development

> - **[tsconfig.json](mdc:tsconfig.json)** - Strict TypeScript configuration with ES2020 target

# TypeScript Development Guidelines

## 🔧 Configuration

### TypeScript Setup
- **[tsconfig.json](mdc:tsconfig.json)** - Strict TypeScript configuration with ES2020 target
- **Strict Mode**: All strict TypeScript options enabled
- **Path Mapping**: Clean imports with aliases (`@core/*`, `@modules/*`, `@types/*`)
- **Declaration Files**: Full `.d.ts` generation for library usage

### Code Quality Tools
- **ESLint**: TypeScript-specific rules in [package.json](mdc:package.json)
- **Prettier**: Consistent formatting configuration
- **Build System**: Automated TypeScript compilation via [scripts/build.sh](mdc:scripts/build.sh)

## 📝 Development Best Practices

### Type Safety
```typescript
// ✅ Use strict interfaces from src/types/global.ts
import { ModuleName, CLIConfig, CLIModule } from '../types/global';

// ✅ Implement proper interfaces
export class MyModule implements CLIModule {
  public name: ModuleName = 'smartlead';
  // ... other required properties
}

// ✅ Use proper error handling
try {
  await module.executeCommand(command, args);
} catch (error) {
  console.log(this.theme.errorMessage(`Error: ${error}`));
}
```

### Import Patterns
```typescript
// ✅ Use relative imports for local files
import { ConfigManager } from './utils/config';
import { ThemeManager } from './utils/theme';

// ✅ Use absolute imports for types
import { ModuleName, CLIModule } from '../types/global';

// ✅ Proper external imports
import * as fs from 'fs';
import * as path from 'path';
```

### Environment Variables
```typescript
// ✅ Use bracket notation for strict TypeScript
process.env['SMARTLEAD_API_KEY']
process.env['NODE_ENV']

// ❌ Avoid dot notation (causes TS4111 errors)
process.env.SMARTLEAD_API_KEY
```

## 🏗️ Module Development

### Creating New Modules
1. **Interface**: Implement `CLIModule` interface from [src/types/global.ts](mdc:src/types/global.ts)
2. **Types**: Create module-specific types (see [src/modules/smartlead/types.ts](mdc:src/modules/smartlead/types.ts))
3. **Theme**: Add theme colors to [src/core/utils/theme.ts](mdc:src/core/utils/theme.ts)
4. **Registration**: Register in [src/core/module-selector.ts](mdc:src/core/module-selector.ts)

### Module Structure
```typescript
export default class MyModule implements CLIModule {
  public name: ModuleName = 'mymodule';
  public displayName = 'My Module';
  public description = 'Module description';
  public version = '1.0.0';
  public commands: Command[] = [];
  private theme: ThemeManager;

  constructor() {
    this.theme = new ThemeManager(this.name);
    this.setupCommands();
  }

  public async initialize(): Promise<void> {
    // Module initialization
  }

  public getCommands(): Command[] {
    return this.commands;
  }

  public async executeCommand(commandName: string, args: string[]): Promise<void> {
    // Command execution logic
  }
}
```

## 🧪 Testing Guidelines

### Test Structure
- **Location**: [tests/](mdc:tests/) directory with mirror structure
- **Setup**: [tests/setup.ts](mdc:tests/setup.ts) for test environment
- **Configuration**: [jest.config.js](mdc:jest.config.js) with TypeScript support

### Test Examples
```typescript
import { ConfigManager } from '../../../src/core/utils/config';
import { ModuleName } from '../../../src/types/global';

describe('ConfigManager', () => {
  let configManager: ConfigManager;

  beforeEach(() => {
    configManager = ConfigManager.getInstance();
  });

  it('should handle configuration correctly', () => {
    const config = { apiKey: 'test-key' };
    configManager.saveConfig(config);
    expect(configManager.loadConfig()).toEqual(config);
  });
});
```

### Test Commands
- **Run Tests**: `npm test`
- **Watch Mode**: `npm run test:watch`
- **Coverage**: `npm run test:coverage`

## 🔨 Build Process

### Development Workflow
```bash
npm run dev       # TypeScript watch mode with ts-node
npm run watch     # Nodemon with automatic recompilation
npm run build     # Full TypeScript compilation to dist/
npm run type-check # TypeScript validation without compilation
```

### Production Build
The [scripts/build.sh](mdc:scripts/build.sh) script handles:
1. TypeScript compilation to `dist/`
2. Binary creation for CLI commands
3. Package.json copying for version info
4. Executable permissions setup

### Code Quality
```bash
npm run lint      # ESLint checking
npm run lint:fix  # Auto-fix ESLint issues  
npm run format    # Prettier formatting
```

## 📦 Distribution

### Package Configuration
- **[package.json](mdc:package.json)** - Comprehensive npm configuration
- **Binary Commands**: `smartlead` and `sl` aliases
- **Files**: Only dist/, docs/, and essential files included

### Installation Methods
```bash
# Development
npm install && npm run build

# Global installation
npm run install-global

# Production (when published)
npm install -g smartlead-cli
```

---
> Source: [LeadMagic/cold-email-cli](https://github.com/LeadMagic/cold-email-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
