## prompt-or-die-architect

> Development best practices guide for working with ElizaOS core package


# ElizaOS Development Best Practices

## Core Development Principles

### Type Safety First

```typescript
// ✅ DO: Use strict TypeScript types throughout
import type {
  IAgentRuntime,
  Memory,
  Character,
  Action,
  Provider,
  State
} from '@elizaos/core';

// Avoid any types
function processMemory(memory: Memory): string {
  // TypeScript knows memory.content.text exists
  return memory.content.text || 'No content';
}

// ❌ DON'T: Use any or generic types when specific types exist
function badProcessMemory(memory: any): any {
  return memory.content?.text; // No type safety
}
```

### Error Handling Patterns

```typescript
// ✅ DO: Use structured error handling
export class ServiceError extends Error {
  constructor(
    message: string,
    public code: string,
    public context?: Record<string, any>
  ) {
    super(message);
    this.name = 'ServiceError';
  }
}

async function safeOperation(runtime: IAgentRuntime): Promise<void> {
  try {
    await runtime.createMemory({
      entityId: runtime.agentId,
      roomId: 'system-room',
      content: { text: 'Operation started' }
    });
  } catch (error) {
    if (error instanceof ServiceError) {
      await runtime.logger.error('Service operation failed', {
        code: error.code,
        context: error.context
      });
    }
    throw error;
  }
}

// ✅ DO: Use error boundaries for async operations
async function withErrorBoundary<T>(
  operation: () => Promise<T>,
  fallback: T
): Promise<T> {
  try {
    return await operation();
  } catch (error) {
    console.error('Operation failed, using fallback:', error);
    return fallback;
  }
}
```

### Resource Management

```typescript
// ✅ DO: Properly manage resources with cleanup
class ResourceManager {
  private resources = new Map<string, any>();

  async acquire<T>(key: string, factory: () => Promise<T>): Promise<T> {
    if (this.resources.has(key)) {
      return this.resources.get(key);
    }

    const resource = await factory();
    this.resources.set(key, resource);
    return resource;
  }

  release(key: string): void {
    const resource = this.resources.get(key);
    if (resource && typeof resource.close === 'function') {
      resource.close();
    }
    this.resources.delete(key);
  }

  async cleanup(): Promise<void> {
    for (const key of this.resources.keys()) {
      this.release(key);
    }
  }
}

// ✅ DO: Use resource managers in services
class DatabaseService extends Service {
  private resourceManager = new ResourceManager();

  async query(sql: string): Promise<any[]> {
    const connection = await this.resourceManager.acquire(
      'db-connection',
      () => createDatabaseConnection()
    );

    try {
      return await connection.query(sql);
    } finally {
      // Connection pooling handles cleanup automatically
    }
  }

  async stop(): Promise<void> {
    await this.resourceManager.cleanup();
  }
}
```

## Memory Management

### Efficient Memory Operations

```typescript
// ✅ DO: Use batch operations for multiple memories
async function createMultipleMemories(
  runtime: IAgentRuntime,
  memories: Omit<Memory, 'id' | 'createdAt'>[]
): Promise<UUID[]> {
  const memoryIds: UUID[] = [];

  // Use Promise.all for concurrent operations
  const results = await Promise.all(
    memories.map(memory =>
      runtime.createMemory({
        ...memory,
        createdAt: Date.now()
      })
    )
  );

  return results;
}

// ✅ DO: Implement memory pagination
async function getMemoriesPaginated(
  runtime: IAgentRuntime,
  roomId: UUID,
  page: number = 1,
  limit: number = 50
): Promise<{ memories: Memory[]; hasMore: boolean }> {
  const offset = (page - 1) * limit;

  const memories = await runtime.getMemories({
    roomId,
    count: limit + 1, // Get one extra to check if there are more
    // Add any additional filters
  });

  const hasMore = memories.length > limit;
  const pageMemories = hasMore ? memories.slice(0, limit) : memories;

  return { memories: pageMemories, hasMore };
}
```

### Memory Search Optimization

```typescript
// ✅ DO: Use efficient search patterns
async function searchWithFallback(
  runtime: IAgentRuntime,
  query: string,
  options: {
    roomId?: UUID;
    threshold?: number;
    limit?: number;
  }
): Promise<Memory[]> {
  const {
    roomId,
    threshold = 0.8,
    limit = 10
  } = options;

  // Try semantic search first
  const semanticResults = await runtime.searchMemories({
    query,
    roomId,
    match_threshold: threshold,
    count: limit
  });

  if (semanticResults.length > 0) {
    return semanticResults;
  }

  // Fallback to text-based search
  return await runtime.getMemories({
    roomId,
    count: limit,
    // Filter by text content containing query terms
  });
}
```

## State Management

### State Composition Best Practices

```typescript
// ✅ DO: Create focused state composition
async function composeFocusedState(
  runtime: IAgentRuntime,
  message: Memory,
  requiredProviders: string[]
): Promise<State> {
  // Only include necessary providers
  const state = await runtime.composeState(
    message,
    requiredProviders,
    true, // onlyInclude
    false // skipCache for fresh data
  );

  // Add computed properties
  state.values.messageLength = message.content.text?.length || 0;
  state.values.hasAttachments = (message.content.attachments?.length || 0) > 0;

  return state;
}

// ✅ DO: Implement state validation
function validateState(state: State): boolean {
  // Check required properties
  if (!state.text || state.text.trim().length === 0) {
    return false;
  }

  // Check data consistency
  if (state.values.userName && typeof state.values.userName !== 'string') {
    return false;
  }

  return true;
}

// ✅ DO: Use state caching strategically
class StateCache {
  private cache = new Map<string, { state: State; timestamp: number }>();
  private readonly ttl = 5 * 60 * 1000; // 5 minutes

  get(key: string): State | null {
    const cached = this.cache.get(key);
    if (!cached) return null;

    if (Date.now() - cached.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }

    return JSON.parse(JSON.stringify(cached.state)); // Deep clone
  }

  set(key: string, state: State): void {
    this.cache.set(key, {
      state: JSON.parse(JSON.stringify(state)), // Deep clone
      timestamp: Date.now()
    });
  }
}
```

## Component Development

### Action Development Patterns

```typescript
// ✅ DO: Create robust actions with proper validation
const robustAction: Action = {
  name: 'PROCESS_DATA',
  description: 'Process incoming data with validation and error handling',

  validate: async (runtime: IAgentRuntime, message: Memory): Promise<boolean> => {
    // Fast validation
    if (!message.content.text) return false;

    // Check if required services are available
    return runtime.hasService('data-processor');
  },

  handler: async (runtime: IAgentRuntime, message: Memory, state?: State): Promise<ActionResult> => {
    try {
      // Input validation
      const data = parseData(message.content.text);
      if (!isValidData(data)) {
        return {
          success: false,
          error: 'Invalid data format',
          text: 'The provided data is not in the expected format.'
        };
      }

      // Process the data
      const result = await processData(runtime, data);

      // Store the result
      await runtime.createMemory({
        entityId: message.entityId,
        roomId: message.roomId,
        content: {
          text: `Data processed successfully: ${result.summary}`,
          action: 'PROCESS_DATA',
          data: result
        }
      });

      return {
        success: true,
        text: `Successfully processed data. Result: ${result.summary}`,
        data: result
      };

    } catch (error) {
      // Log the error
      await runtime.logger.error('PROCESS_DATA action failed', {
        error: error.message,
        messageId: message.id
      });

      return {
        success: false,
        error: error.message,
        text: 'Failed to process the data. Please try again.'
      };
    }
  }
};
```

### Provider Development Patterns

```typescript
// ✅ DO: Create efficient providers with caching
const cachedWeatherProvider: Provider = {
  name: 'weather',
  description: 'Provides current weather information',
  dynamic: true,

  get: async (runtime: IAgentRuntime, message: Memory, state: State): Promise<ProviderResult> => {
    // Check cache first
    const cacheKey = `weather_${message.roomId}`;
    const cached = state.weatherCache?.[cacheKey];

    if (cached && Date.now() - cached.timestamp < 10 * 60 * 1000) { // 10 minutes
      return cached.data;
    }

    // Fetch fresh data
    const location = await extractLocationFromMessage(message);
    const weatherData = await fetchWeatherData(location);

    const result = {
      text: `Current weather in ${location}: ${weatherData.temperature}°F, ${weatherData.condition}`,
      data: { weather: weatherData, location },
      values: {
        temperature: weatherData.temperature,
        condition: weatherData.condition,
        location
      }
    };

    // Update cache
    if (!state.weatherCache) state.weatherCache = {};
    state.weatherCache[cacheKey] = {
      timestamp: Date.now(),
      data: result
    };

    return result;
  }
};
```

## Service Development

### Service Lifecycle Management

```typescript
// ✅ DO: Implement proper service lifecycle
class LifecycleManagedService extends Service {
  static serviceType = 'lifecycle-managed';
  capabilityDescription = 'Service with proper lifecycle management';

  private connections: Connection[] = [];
  private isShuttingDown = false;

  static async start(runtime: IAgentRuntime): Promise<LifecycleManagedService> {
    const service = new LifecycleManagedService(runtime);

    // Initialize connections
    service.connections = await Promise.all([
      createDatabaseConnection(runtime.getSetting('DB_URL')),
      createCacheConnection(runtime.getSetting('CACHE_URL')),
      createQueueConnection(runtime.getSetting('QUEUE_URL'))
    ]);

    // Set up health monitoring
    service.setupHealthChecks();

    return service;
  }

  async stop(): Promise<void> {
    if (this.isShuttingDown) return;
    this.isShuttingDown = true;

    // Graceful shutdown with timeout
    const shutdownTimeout = setTimeout(() => {
      console.warn('Service shutdown timed out, forcing close');
      this.forceShutdown();
    }, 30000);

    try {
      await Promise.all(
        this.connections.map(conn => conn.close())
      );
    } catch (error) {
      console.error('Error during service shutdown:', error);
    } finally {
      clearTimeout(shutdownTimeout);
    }
  }

  private setupHealthChecks(): void {
    setInterval(async () => {
      if (this.isShuttingDown) return;

      const health = await this.checkHealth();
      if (!health.healthy) {
        await this.runtime.logger.warn('Service health degraded', health);
      }
    }, 60000); // Check every minute
  }

  private async checkHealth(): Promise<{ healthy: boolean; details?: any }> {
    // Implement health checks for all connections
    return { healthy: true };
  }

  private forceShutdown(): void {
    // Force close all connections
    this.connections.forEach(conn => {
      try { conn.destroy(); } catch {}
    });
    this.connections = [];
  }
}
```

### Service Builder Pattern

```typescript
// ✅ DO: Use service builders for complex service setup
const complexService = createService<ComplexService>('complex-service')
  .withDescription('A complex service with multiple dependencies')
  .withStart(async (runtime: IAgentRuntime) => {
    const service = new ComplexService(runtime);

    // Initialize multiple components
    await Promise.all([
      service.initializeDatabase(),
      service.initializeCache(),
      service.initializeQueue(),
      service.initializeMetrics()
    ]);

    // Set up event handlers
    service.setupEventHandlers();

    return service;
  })
  .withStop(async () => {
    await service.shutdown();
  })
  .build();
```

## Plugin Development

### Plugin Structure and Organization

```typescript
// ✅ DO: Organize plugins with clear structure
const wellOrganizedPlugin: Plugin = {
  name: 'well-organized-plugin',
  description: 'A plugin with clear organization and best practices',

  dependencies: ['@elizaos/plugin-bootstrap'],

  init: async (config: Record<string, string>, runtime: IAgentRuntime): Promise<void> => {
    // Validate configuration
    const requiredSettings = ['API_KEY', 'BASE_URL'];
    const missingSettings = requiredSettings.filter(key => !config[key]);

    if (missingSettings.length > 0) {
      throw new Error(`Missing required settings: ${missingSettings.join(', ')}`);
    }

    // Initialize plugin resources
    await initializePluginResources(runtime, config);

    // Register event handlers
    setupPluginEventHandlers(runtime);

    runtime.logger.info(`${this.name} initialized successfully`);
  },

  // Organize components logically
  actions: [userAction, adminAction, utilityAction],
  providers: [userProvider, systemProvider, contextProvider],
  evaluators: [qualityEvaluator, securityEvaluator],
  services: [APIService, CacheService],

  // Use configuration for customization
  config: {
    apiTimeout: 5000,
    maxRetries: 3,
    enableCaching: true
  }
};

// ✅ DO: Separate plugin components into logical modules
// actions/user-actions.ts
export const userAction: Action = { /* ... */ };

// providers/user-providers.ts
export const userProvider: Provider = { /* ... */ };

// services/api-service.ts
export class APIService extends Service { /* ... */ };
```

### Plugin Configuration Management

```typescript
// ✅ DO: Use Zod for plugin configuration validation
import { z } from 'zod';

const pluginConfigSchema = z.object({
  apiKey: z.string().min(1, 'API key is required'),
  baseUrl: z.string().url('Base URL must be a valid URL'),
  timeout: z.number().min(1000).max(30000).default(5000),
  maxRetries: z.number().min(0).max(10).default(3),
  enableCaching: z.boolean().default(true),
  cacheTtl: z.number().min(0).default(300000), // 5 minutes
});

type PluginConfig = z.infer<typeof pluginConfigSchema>;

const validatedConfig = pluginConfigSchema.parse(rawConfig);
```

## Testing Practices

### Unit Testing Patterns

```typescript
// ✅ DO: Write comprehensive unit tests
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { createMockRuntime } from './test-utils';
import { myAction } from '../src/actions/my-action';

describe('myAction', () => {
  let mockRuntime: IAgentRuntime;
  let mockMessage: Memory;

  beforeEach(() => {
    mockRuntime = createMockRuntime();
    mockMessage = {
      id: 'test-message-id',
      entityId: 'test-entity-id',
      roomId: 'test-room-id',
      content: { text: 'Test message' },
      createdAt: Date.now()
    };
  });

  afterEach(() => {
    vi.clearAllMocks();
  });

  describe('validate', () => {
    it('should return true for valid messages', async () => {
      const isValid = await myAction.validate(mockRuntime, mockMessage);
      expect(isValid).toBe(true);
    });

    it('should return false for invalid messages', async () => {
      const invalidMessage = { ...mockMessage, content: { text: '' } };
      const isValid = await myAction.validate(mockRuntime, invalidMessage);
      expect(isValid).toBe(false);
    });
  });

  describe('handler', () => {
    it('should process valid messages successfully', async () => {
      const result = await myAction.handler(mockRuntime, mockMessage);

      expect(result).toEqual({
        success: true,
        text: expect.any(String)
      });
      expect(mockRuntime.createMemory).toHaveBeenCalled();
    });

    it('should handle errors gracefully', async () => {
      mockRuntime.createMemory.mockRejectedValue(new Error('Database error'));

      const result = await myAction.handler(mockRuntime, mockMessage);

      expect(result.success).toBe(false);
      expect(result.error).toBeDefined();
    });
  });
});
```

### Integration Testing

```typescript
// ✅ DO: Write integration tests for component interactions
describe('Plugin Integration', () => {
  let runtime: IAgentRuntime;

  beforeEach(async () => {
    runtime = await TestRuntimeFactory.createTestRuntime({}, {
      plugins: [testPlugin]
    });
  });

  afterEach(async () => {
    await TestRuntimeFactory.cleanup();
  });

  it('should handle end-to-end message processing', async () => {
    const message = createTestMessage({
      text: 'Process this message'
    });

    // Process the message through the runtime
    await runtime.processActions(message, [], {});

    // Verify the results
    const memories = await runtime.getMemories({
      roomId: message.roomId,
      count: 10
    });

    expect(memories.length).toBeGreaterThan(1);
    const response = memories.find(m => m.agentId === runtime.agentId);
    expect(response).toBeDefined();
    expect(response?.content.text).toContain('processed');
  });
});
```

## Performance Optimization

### Memory Usage Optimization

```typescript
// ✅ DO: Implement memory-efficient patterns
class MemoryEfficientService extends Service {
  private memoryCache = new WeakMap<object, any>();

  // Use WeakMap for automatic garbage collection
  getCachedData(key: object): any {
    return this.memoryCache.get(key);
  }

  setCachedData(key: object, value: any): void {
    this.memoryCache.set(key, value);
  }

  // Clean up on service stop
  async stop(): Promise<void> {
    // WeakMap cleans up automatically, but we can clear references
    this.memoryCache = new WeakMap();
  }
}

// ✅ DO: Use streaming for large data processing
async function processLargeDataset(
  runtime: IAgentRuntime,
  dataStream: ReadableStream
): Promise<void> {
  const reader = dataStream.getReader();
  let processedCount = 0;

  try {
    while (true) {
      const { done, value } = await reader.read();

      if (done) break;

      // Process data in chunks
      await processDataChunk(runtime, value);
      processedCount += value.length;

      // Periodic cleanup to prevent memory buildup
      if (processedCount % 1000 === 0) {
        if (global.gc) global.gc(); // Manual GC if available
      }
    }
  } finally {
    reader.releaseLock();
  }
}
```

### Database Optimization

```typescript
// ✅ DO: Use database connection pooling
class OptimizedDatabaseService extends Service {
  private pool: ConnectionPool;

  static async start(runtime: IAgentRuntime): Promise<OptimizedDatabaseService> {
    const service = new OptimizedDatabaseService(runtime);

    service.pool = new ConnectionPool({
      min: 2,
      max: 10,
      idleTimeoutMillis: 30000,
      acquireTimeoutMillis: 60000,
    });

    return service;
  }

  async query(sql: string, params?: any[]): Promise<any[]> {
    const connection = await this.pool.getConnection();

    try {
      return await connection.query(sql, params);
    } finally {
      connection.release(); // Always release connection
    }
  }

  async stop(): Promise<void> {
    if (this.pool) {
      await this.pool.close();
    }
  }
}

// ✅ DO: Implement database query optimization
async function optimizedQuery(
  runtime: IAgentRuntime,
  query: string,
  options: {
    useIndex?: boolean;
    limit?: number;
    offset?: number;
  }
): Promise<any[]> {
  const dbService = runtime.getService<OptimizedDatabaseService>('database');

  // Add query optimizations
  let optimizedQuery = query;

  if (options.useIndex) {
    optimizedQuery = `/*+ INDEX(table_name index_name) */ ${query}`;
  }

  if (options.limit) {
    optimizedQuery += ` LIMIT ${options.limit}`;
  }

  if (options.offset) {
    optimizedQuery += ` OFFSET ${options.offset}`;
  }

  return await dbService.query(optimizedQuery);
}
```

## Code Organization

### File Structure Best Practices

```
packages/core/src/
├── types/                 # All type definitions
│   ├── index.ts          # Main type exports
│   ├── primitives.ts     # Basic types (UUID, Content)
│   ├── components.ts     # Action, Provider, Evaluator
│   ├── runtime.ts        # Runtime interface
│   └── ...
├── utils/                # Utility functions
│   ├── index.ts          # Utility exports
│   ├── environment.ts    # Environment utilities
│   ├── buffer.ts         # Buffer utilities
│   └── ...
├── actions/              # Action implementations
├── providers/            # Provider implementations
├── services/             # Service implementations
├── runtime.ts            # Runtime implementation
├── logger.ts             # Logging utilities
└── ...
```

### Import Organization

```typescript
// ✅ DO: Organize imports logically
// 1. External dependencies
import { z } from 'zod';
import { v4 as uuidv4 } from 'uuid';

// 2. Internal types and interfaces
import type {
  IAgentRuntime,
  Memory,
  Character,
  Action,
  Provider
} from './types';

// 3. Internal utilities
import { validateCharacter } from './schemas/character';
import { createLogger } from './logger';

// 4. Local imports
import { processMemory } from './utils/memory';
```

## References

- [TypeScript Best Practices](mdc:https:/www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
- [Node.js Performance Best Practices](mdc:https:/nodejs.org/en/docs/guides/dont-block-the-event-loop)
- [Database Connection Pooling](mdc:https:/github.com/vitaly-t/pg-promise#connection-pools)
- [Memory Management in Node.js](mdc:https:/nodejs.org/en/docs/guides/memory-usage)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dexploarer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
