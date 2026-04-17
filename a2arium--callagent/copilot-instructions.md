## callagent

> This rule provides guidance on using the TaskContext (ctx) object that's passed to agent handlers. The TaskContext provides all the core functionality agents need to interact with users and the framework.

# Cursor Rule: TaskContext and Agent Development

## Purpose
This rule provides guidance on using the TaskContext (ctx) object that's passed to agent handlers. The TaskContext provides all the core functionality agents need to interact with users and the framework.

## Key TaskContext Features

### Progress Tracking

The framework supports two types of progress updates:

```typescript
// Percentage-based progress with message
ctx.progress(25, 'Processing data...');          // Shows: "Progress: 25% - Processing data..."
ctx.progress(50, 'Halfway done!');               // Shows: "Progress: 50% - Halfway done!"
ctx.progress(100);                                // Shows: "Progress: 100%"

// Status-based progress (more detailed)
ctx.progress({
    state: 'working',
    timestamp: new Date().toISOString(),
    message: {
        role: 'agent',
        parts: [{ type: 'text', text: 'Calling external API...' }]
    }
});  // Shows: "Progress: Calling external API..."

// Mixed approach - you can use both in the same agent
ctx.progress(10, 'Starting...');
ctx.progress({ state: 'working', message: { role: 'agent', parts: [{ type: 'text', text: 'Complex operation...' }] } });
ctx.progress(90, 'Almost done...');
```

### Replying to Users

```typescript
// Simple text reply
await ctx.reply("Hello, user!");

// Multiple text parts
await ctx.reply(["First line", "Second line"]);

// Structured message parts
await ctx.reply([
    { type: 'text', text: 'Here is the result:' },
    { type: 'text', text: JSON.stringify(data, null, 2) }
]);

// With artifact options (for streaming)
await ctx.reply("Partial response...", { 
    artifactName: 'analysis',
    append: true,
    lastChunk: false 
});
```

### Task Completion

```typescript
// Simple completion
ctx.complete();

// With status message
ctx.complete(100, 'Task completed successfully');

// Automatic completion happens if you don't call complete() explicitly
```

### Error Handling

```typescript
// Structured error throwing
ctx.throw('INVALID_INPUT', 'The provided data is invalid', { 
    field: 'email',
    value: userInput.email 
});

// Failing the task
await ctx.fail(new Error('Something went wrong'));
```

### Logging

```typescript
// Import the centralized logger
import { logger } from '@a2arium/callagent-utils';

// Use logger directly - context is automatic
logger.info('Processing started', { userId: ctx.task.input.userId });
logger.warn('Rate limit approaching', { remaining: 10 });
logger.error('Database connection failed', error, { retryCount: 3 });
```

### Memory Operations

```typescript
// Semantic memory
await ctx.memory.semantic.store('user-preferences', { theme: 'dark', lang: 'en' });
const prefs = await ctx.memory.semantic.retrieve('user-preferences');

// Search memory
const results = await ctx.memory.semantic.search('user settings', { limit: 10 });
```

### Environment Variables

```typescript
// Get environment variables with optional defaults
const apiKey = ctx.getEnv('API_KEY');
const timeout = ctx.getEnv('TIMEOUT', '30000');
```

### Usage Tracking

```typescript
// Record usage (usually automatic for LLMs)
ctx.recordUsage({ cost: 0.05 });
ctx.recordUsage(0.05); // Simple number format
```

## Best Practices

1. **Always await async operations** like `ctx.reply()` and `ctx.fail()`
2. **Use progress tracking** for long-running operations to keep users informed
3. **Use structured logging** with context data for better debugging
4. **Handle errors gracefully** using `ctx.fail()` or `ctx.throw()`
5. **Complete tasks explicitly** when you want to control the completion message

## Related Documents
* `.cursor/rules/llm_factory.mdc` - LLM configuration and usage
* `.cursor/rules/a2a_high_level_summary.mdc` - A2A protocol overview

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/a2arium) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
