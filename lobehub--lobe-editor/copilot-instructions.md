## debug-logging

> Debug logging standards and usage guidelines for LobeHub Editor


# Debug Logging Standards

## Debug Package Integration

LobeHub Editor uses the [debug](https://www.npmjs.com/package/debug) package for structured logging. All utilities are centralized in [src/utils/debug.ts](mdc:src/utils/debug.ts).

### Usage Pattern

```typescript
import { createDebugLogger } from '@/utils/debug';

class MyComponent {
  private logger = createDebugLogger('kernel'); // or 'plugin', 'service', etc.
  
  someMethod() {
    this.logger.info('🚀 Component initialized');    // Major events
    this.logger.debug('🔍 Processing data');         // Detailed tracing (sparingly)
    this.logger.warn('⚠️ Deprecated feature used');  // Non-critical issues
    this.logger.error('❌ Operation failed:', error); // Critical errors
  }
}
```

**Note**: `logger.error/warn` use native console methods for proper browser dev tool categorization.

### Category-Specific Usage

```typescript
// Plugins
export const MyPlugin = class extends KernelPlugin {
  private logger = createDebugLogger('plugin', 'my-plugin-name');
}

// Services
export class MyService implements IMyService {
  private logger = createDebugLogger('service', 'my-service');
}

// Demo files - development only
import { devConsole } from '@/utils/debug';
devConsole.log('Development only logging');
```

## Environment Configuration

```bash
# Enable all debug messages
DEBUG=lobe-editor:*

# Enable specific categories
DEBUG=lobe-editor:kernel,lobe-editor:plugin:*

# Enable with exclusions (reduce noise)
DEBUG=lobe-editor:*,-lobe-editor:demo

# Enable only critical logs
DEBUG=lobe-editor:*:error,lobe-editor:*:warn
```

**Environment Usage:**

- **Development**: Use `devConsole` for temporary debugging
- **Production**: Use debug loggers that respect environment variables

## Available Namespaces

- `lobe-editor:kernel` - Core editor functionality
- `lobe-editor:plugin:*` - Plugin-specific debugging
- `lobe-editor:service:*` - Service layer debugging
- `lobe-editor:upload` - File upload operations
- `lobe-editor:react` - React components
- `lobe-editor:demo` - Demo components

## Migration from console.\*

```typescript
// ❌ Don't use console directly
console.log('Debug info');

// ✅ Use debug loggers instead
this.logger.debug('Debug info');

// ✅ For demo files, use devConsole
devConsole.log('Demo debug info');
```

## Best Practices

1. **Use appropriate log levels**: `info` for major events, `warn` for non-critical issues, `error` for failures, `debug` sparingly
2. **Visual distinction with emojis**: 🚀 startup, ✅ success, ⚠️ warnings, ❌ errors
3. **Reduce log noise**: Group similar operations, skip repetitive operations
4. **Never use console.* directly*\* - Always use debug loggers
5. **Meaningful, concise messages** - Clear but brief descriptions
6. **Respect environment variables** - All logs should be controllable via DEBUG

---
> Source: [lobehub/lobe-editor](https://github.com/lobehub/lobe-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
