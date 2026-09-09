## module-system

> The SmartLead CLI uses a **modular architecture** where each email marketing platform is implemented as a separate module. This allows for:

# Module System Architecture

## 🧩 Core Concept

The SmartLead CLI uses a **modular architecture** where each email marketing platform is implemented as a separate module. This allows for:
- **Platform Independence**: Each module operates independently
- **Branded Experience**: Module-specific themes and UX
- **Easy Extension**: Add new platforms without affecting existing ones
- **Clean Separation**: Clear boundaries between platform implementations

## 📁 Module Structure

### Current Modules
- **[src/modules/smartlead/](mdc:src/modules/smartlead/)** - SmartLead platform (✅ Available, 80+ commands)
- **[src/modules/instantly/index.ts](mdc:src/modules/instantly/index.ts)** - Instantly platform (🟡 Coming Q2 2024)

### Module Interface
All modules implement the `CLIModule` interface defined in [src/types/global.ts](mdc:src/types/global.ts):

```typescript
export interface CLIModule {
  name: ModuleName;
  displayName: string;
  description: string;
  version: string;
  commands: Command[];
  initialize(): Promise<void>;
  getCommands(): Command[];
  executeCommand(commandName: string, args: string[]): Promise<void>;
}
```

## 🔧 Module Management

### Module Selector
The **[src/core/module-selector.ts](mdc:src/core/module-selector.ts)** handles:
- **Module Registration**: Registering available modules
- **Interactive Selection**: UI for switching between modules
- **Module Loading**: Dynamic module loading and initialization
- **Module Information**: Display module status and capabilities

### Configuration per Module
Each module can have its own configuration:
- **Global Config**: `~/.smartlead-cli/config.json` (active module, shared settings)
- **Module Config**: `~/.smartlead-cli/{module-name}.json` (API keys, module settings)
- **Environment Variables**: Module-specific environment variables

Example configuration in [src/core/utils/config.ts](mdc:src/core/utils/config.ts):
```typescript
public getModuleConfig(moduleName: ModuleName): ModuleConfig | null {
  const moduleConfigFile = path.join(this.configDir, `${moduleName}.json`);
  // Load module-specific configuration
}
```

## 🎨 Theme System per Module

### Theme Management
The **[src/core/utils/theme.ts](mdc:src/core/utils/theme.ts)** provides module-specific themes:

```typescript
export const themes: Record<ModuleName, ThemeColors> = {
  smartlead: {
    primary: '#2563eb',    // SmartLead Blue
    secondary: '#0ea5e9',  // Light Blue  
    accent: '#06b6d4',     // Cyan
    // ... other colors
  },
  instantly: {
    primary: '#7c3aed',    // Purple
    secondary: '#a855f7',  // Light Purple
    accent: '#ec4899',     // Pink
    // ... other colors
  }
};
```

### Using Themes in Modules
```typescript
export default class MyModule implements CLIModule {
  private theme: ThemeManager;

  constructor() {
    this.theme = new ThemeManager(this.name);
  }

  private showMessage() {
    console.log(this.theme.primary('Welcome to MyModule!'));
    console.log(this.theme.success('✅ Operation completed'));
  }
}
```

## 📝 Creating New Modules

### Step 1: Create Module File
Create `src/modules/newmodule/index.ts`:
```typescript
import { CLIModule, ModuleName, Command } from '../../types/global';
import { ThemeManager } from '../../core/utils/theme';

export default class NewModule implements CLIModule {
  public name: ModuleName = 'newmodule';
  public displayName = 'New Module';
  public description = 'Description of the new module';
  public version = '1.0.0';
  public commands: Command[] = [];
  private theme: ThemeManager;

  constructor() {
    this.theme = new ThemeManager(this.name);
    this.setupCommands();
  }

  private setupCommands(): void {
    this.commands = [
      {
        name: 'example',
        description: 'Example command',
        usage: 'newmodule example',
        examples: ['newmodule example --option value']
      }
    ];
  }

  public async initialize(): Promise<void> {
    this.theme.setModule(this.name);
  }

  public getCommands(): Command[] {
    return this.commands;
  }

  public async executeCommand(commandName: string, args: string[]): Promise<void> {
    switch (commandName) {
      case 'example':
        await this.handleExample(args);
        break;
      default:
        console.log(this.theme.errorMessage(`Unknown command: ${commandName}`));
    }
  }

  private async handleExample(args: string[]): Promise<void> {
    console.log(this.theme.success('Example command executed!'));
  }
}
```

### Step 2: Add Types
Update [src/types/global.ts](mdc:src/types/global.ts):
```typescript
export type ModuleName = 'smartlead' | 'instantly' | 'newmodule';
```

### Step 3: Add Theme
Update [src/core/utils/theme.ts](mdc:src/core/utils/theme.ts):
```typescript
export const themes: Record<ModuleName, ThemeColors> = {
  // ... existing themes
  newmodule: {
    primary: '#your-color',
    secondary: '#your-color',
    // ... other colors
  }
};
```

### Step 4: Register Module
Update [src/core/module-selector.ts](mdc:src/core/module-selector.ts):
```typescript
private registerModules(): void {
  // ... existing modules
  this.modules.set('newmodule', {
    name: 'newmodule',
    displayName: 'New Module',
    description: 'Description of the new module',
    version: '1.0.0',
    available: true
  });
}
```

### Step 5: Create Module-Specific Types
Create `src/modules/newmodule/types.ts` for module-specific interfaces:
```typescript
export interface NewModuleConfig {
  apiKey: string;
  baseUrl: string;
  // ... module-specific configuration
}

export interface NewModuleCommand {
  id: string;
  name: string;
  // ... module-specific command structure
}
```

## 🔄 Module Lifecycle

### Module Loading Process
1. **User Selection**: User selects module via [src/core/module-selector.ts](mdc:src/core/module-selector.ts)
2. **Dynamic Import**: Module is dynamically loaded
3. **Initialization**: `initialize()` method called
4. **Theme Setup**: Module-specific theme applied
5. **Command Execution**: Commands routed to module

### Module State Management
- **Active Module**: Stored in global configuration
- **Module Config**: Separate configuration per module
- **Theme State**: Theme manager tracks current module theme
- **Command Context**: Commands execute within module context

## 🧪 Testing Modules

### Module Tests
Create tests in `tests/modules/newmodule/`:
```typescript
import NewModule from '../../../src/modules/newmodule/index';

describe('NewModule', () => {
  let module: NewModule;

  beforeEach(() => {
    module = new NewModule();
  });

  it('should initialize correctly', async () => {
    await module.initialize();
    expect(module.name).toBe('newmodule');
  });

  it('should handle commands', async () => {
    await module.executeCommand('example', []);
    // Test command execution
  });
});
```

### Integration Tests
Test module integration with the core system:
```typescript
import { ModuleSelector } from '../../../src/core/module-selector';

describe('Module Integration', () => {
  it('should load and switch modules', async () => {
    const selector = new ModuleSelector();
    const module = await selector.loadModule('newmodule');
    expect(module).toBeDefined();
  });
});
```

## 📚 Module Documentation

### Module README
Each module should include documentation:
- **API Integration**: How the module connects to its platform
- **Command Reference**: Complete list of available commands
- **Configuration**: Module-specific configuration options
- **Examples**: Usage examples and common workflows

### Command Documentation
Document commands in the module implementation:
```typescript
{
  name: 'example',
  description: 'Detailed description of what this command does',
  usage: 'module example [options]',
  options: [
    {
      flag: '--option',
      description: 'Option description',
      required: false,
      default: 'default-value'
    }
  ],
  examples: [
    'module example --option value',
    'module example --help'
  ]
}
```

---
> Source: [LeadMagic/cold-email-cli](https://github.com/LeadMagic/cold-email-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
