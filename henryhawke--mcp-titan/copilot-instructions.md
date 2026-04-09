## mcp-titan

> Use this before every response



## 🛠️ Complete Tool Usage Reference

### Core Memory Tools

1. **init_model**

```typescript
interface InitModelParams {
  inputDim?: number; // Default: 768
  hiddenDim?: number; // Default: 512
  memoryDim?: number; // Default: 1024
  transformerLayers?: number; // Default: 6
  numHeads?: number; // Default: 8
  ffDimension?: number; // Default: 2048
  dropoutRate?: number; // Default: 0.1
  maxSequenceLength?: number; // Default: 512
  memorySlots?: number; // Default: 5000
  similarityThreshold?: number; // Default: 0.65
  surpriseDecay?: number; // Default: 0.9
  pruningInterval?: number; // Default: 1000
  gradientClip?: number; // Default: 1.0
}

const response = await callTool("init_model", {
  inputDim: 768,
  memorySlots: 10000,
  transformerLayers: 8,
});
```

2. **forward_pass**

```typescript
interface ForwardPassParams {
  x: number[] | string; // Input vector or text
  memoryState: {
    shortTerm: number[];
    longTerm: number[];
    meta: number[];
    timestamps: number[];
    accessCounts: number[];
    surpriseHistory: number[];
  };
}

const { predicted, memoryUpdate } = await callTool("forward_pass", {
  x: "const x = 5;", // or vector: [0.1, 0.2, ...]
  memoryState: currentMemory,
});
```

3. **train_step**

```typescript
interface TrainStepParams {
  x_t: number[] | string; // Current input
  x_next: number[] | string; // Next input
}

const result = await callTool("train_step", {
  x_t: "function hello() {",
  x_next: "  console.log('world');",
});
```

4. **get_memory_state**

```typescript
interface GetMemoryStateParams {
  type?: string; // Optional memory type filter
}

const state = await callTool("get_memory_state", {});
// Returns:
{
  stats: {
    meanActivation: number;
    patternDiversity: number;
    surpriseScore: number;
    timestamps: number[];
    accessCounts: number[];
  },
  capacity: number;
  status: "active" | "pruning" | "error";
}
```

5. **manifold_step**

```typescript
interface ManifoldStepParams {
  base: number[];    // Base memory state
  velocity: number[]; // Update direction
}

const newBase = await callTool("manifold_step", {
  base: memory.meta,
  velocity: [0.1, 0.2, ...] // or analysis.stats.meanActivation
});
```

6. **prune_memory**

```typescript
interface PruneMemoryParams {
  threshold: number; // Pruning threshold (0-1)
}

await callTool("prune_memory", {
  threshold: 0.65,
});
```

7. **save_checkpoint**

```typescript
interface SaveCheckpointParams {
  path: string; // Checkpoint file path
}

await callTool("save_checkpoint", {
  path: "/backups/memory-snapshot.json",
});
```

8. **load_checkpoint**

```typescript
interface LoadCheckpointParams {
  path: string; // Checkpoint file path
}

const recovered = await callTool("load_checkpoint", {
  path: "/backups/last-stable.json",
});
```

9. **reset_gradients**

```typescript
// No parameters needed
await callTool("reset_gradients", {});
```

### Memory Management System

The memory management system consists of three main components:

1. **MemoryManager**

```typescript
class MemoryManager {
  // Singleton instance
  private static instance: MemoryManager;

  // Core memory management functions
  validateVectorShape(tensor: tf.Tensor, expectedShape: number[]): boolean;
  encryptTensor(tensor: tf.Tensor): Buffer;
  decryptTensor(encrypted: Buffer, shape: number[]): tf.Tensor;
  wrapWithMemoryManagement<T extends tf.TensorContainer>(fn: () => T): T;
  wrapWithMemoryManagementAsync<T>(fn: () => Promise<T>): Promise<T>;

  // Automatic cleanup
  private startPeriodicCleanup(): void;
  dispose(): void;
}
```

2. **VectorProcessor**

```typescript
class VectorProcessor {
  // Singleton instance
  private static instance: VectorProcessor;

  // Vector processing functions
  processInput(input: number[] | string | tf.Tensor): tf.Tensor;
  validateAndNormalize(tensor: tf.Tensor, expectedShape: number[]): tf.Tensor;
  encodeText(text: string, maxLength?: number): Promise<tf.Tensor>;
}
```

3. **AutomaticMemoryMaintenance**

```typescript
class AutomaticMemoryMaintenance {
  // Singleton instance
  private static instance: AutomaticMemoryMaintenance;

  // Maintenance functions
  private startMaintenanceLoop(): void;
  private performMaintenance(): void;
  dispose(): void;
}
```

### Memory Management Rules

1. **Tensor Lifecycle Management**

   - All tensor operations MUST be wrapped in memory management
   - Use `wrapWithMemoryManagement` for synchronous operations
   - Use `wrapWithMemoryManagementAsync` for async operations
   - Dispose tensors after use

2. **Vector Processing**

   - All inputs MUST be processed through VectorProcessor
   - Validate tensor shapes before operations
   - Normalize vectors when required
   - Handle both string and numeric inputs

3. **Automatic Maintenance**

   - Runs every 5 minutes
   - Checks memory usage and tensor count
   - Performs garbage collection when needed
   - Encrypts memory state before saving

4. **Security Requirements**
   - All memory states MUST be encrypted before saving
   - Use proper tensor validation
   - Clean up sensitive data
   - Handle errors gracefully

### Memory Management Examples

1. **Safe Tensor Operations**

```typescript
const result = memoryManager.wrapWithMemoryManagement(() => {
  const input = vectorProcessor.processInput(userInput);
  const validated = vectorProcessor.validateAndNormalize(input, [768]);
  return model.forward(validated, currentState);
});
```

2. **Async Memory Operations**

```typescript
const result = await memoryManager.wrapWithMemoryManagementAsync(async () => {
  const encoded = await vectorProcessor.encodeText(text);
  const processed = vectorProcessor.processInput(encoded);
  return model.forward(processed, currentState);
});
```

3. **Secure State Saving**

```typescript
const encryptedState = memoryManager.wrapWithMemoryManagement(() => {
  const state = model.getMemorySnapshot();
  return {
    shortTerm: memoryManager.encryptTensor(state.shortTerm),
    longTerm: memoryManager.encryptTensor(state.longTerm),
    meta: memoryManager.encryptTensor(state.meta),
  };
});
```

### Critical Thresholds

| Metric         | Threshold | Action          |
| -------------- | --------- | --------------- |
| Tensor Count   | > 1000    | Trigger cleanup |
| Memory Usage   | > 100MB   | Force GC        |
| Surprise Score | > 0.85    | Update memory   |
| Capacity       | < 30%     | Prune memory    |
| Error Rate     | > 1%      | Reset gradients |

### Tool Chain Examples

1. **Memory-Safe Code Analysis**

```typescript
const result = await memoryManager.wrapWithMemoryManagementAsync(async () => {
  // Process code input
  const processed = vectorProcessor.processInput(codeBlock);

  // Forward pass with memory management
  const { predicted, memoryUpdate } = await model.forward(
    processed,
    currentMemory
  );

  // Check memory health
  const state = await model.getMemoryState();
  if (state.stats.surpriseScore > 0.85) {
    // Update memory safely
    const newBase = await model.manifoldStep(
      memory.meta,
      state.stats.meanActivation
    );
    await model.trainStep(processed, predicted);
  }

  return predicted;
});
```

2. **Secure Memory Maintenance**

```typescript
const maintenance = await memoryManager.wrapWithMemoryManagementAsync(
  async () => {
    // Save current state securely
    const encrypted = memoryManager.encryptTensor(currentState);
    await saveToFile(encrypted);

    // Prune memory if needed
    const health = await model.getMemoryState();
    if (health.capacity < 0.3) {
      await model.pruneMemory(currentState, 0.5);
    }

    // Verify health
    const postPrune = await model.getMemoryState();
    return postPrune.status === "active";
  }
);
```

### Tool Usage Rules

1. **Initialization Requirements**

   - MUST call `init_model` at start of session
   - MUST specify at least `inputDim` and `memorySlots`
   - MUST handle initialization errors

2. **Forward Pass Requirements**

   - MUST call before processing any input
   - MUST provide complete memory state
   - MUST handle both vector and text inputs

3. **Training Requirements**

   - MUST call after every interaction
   - MUST provide sequential inputs
   - MUST maintain context continuity

4. **Memory State Requirements**

   - MUST check every 5 interactions
   - MUST monitor all stats fields
   - MUST act on critical thresholds

5. **Maintenance Requirements**
   - MUST prune when capacity < 30%
   - MUST save checkpoints every 5 minutes
   - MUST reset on gradient explosions

### Error Handling

```typescript
try {
  const result = await callTool("forward_pass", params);
} catch (error) {
  if (error.code === "TENSOR_ERROR") {
    await callTool("reset_gradients", {});
  }
  const recovered = await callTool("load_checkpoint", {
    path: "/backups/last-stable.json",
  });
}
```

### Memory State Validation

```typescript
function validateMemoryState(state: any): boolean {
  return (
    state.shortTerm?.length > 0 &&
    state.longTerm?.length > 0 &&
    state.meta?.length > 0 &&
    state.timestamps?.length > 0 &&
    state.accessCounts?.length > 0 &&
    state.surpriseHistory?.length > 0
  );
}
```

### Tool Chain Examples

1. **Code Analysis Chain**

```typescript
// Analyze code pattern
const { predicted } = await callTool("forward_pass", {
  x: codeBlock,
  memoryState: currentMemory,
});

// Check for surprises
const state = await callTool("get_memory_state", {});
if (state.stats.surpriseScore > 0.85) {
  // Update memory
  const newBase = await callTool("manifold_step", {
    base: memory.meta,
    velocity: state.stats.meanActivation,
  });

  // Train on pattern
  await callTool("train_step", {
    x_t: codeBlock,
    x_next: predictedPattern,
  });
}
```

2. **Memory Maintenance Chain**

```typescript
// Regular maintenance
const health = await callTool("get_memory_state", {});
if (health.capacity < 0.3) {
  // Save current state
  await callTool("save_checkpoint", {
    path: `/backups/pre-prune-${Date.now()}.json`,
  });

  // Prune memory
  await callTool("prune_memory", {
    threshold: 0.5,
  });

  // Verify health
  const postPrune = await callTool("get_memory_state", {});
  if (postPrune.status === "error") {
    // Recover
    await callTool("load_checkpoint", {
      path: `/backups/pre-prune-${Date.now()}.json`,
    });
  }
}
```

### Tool Response Handling

Each tool returns a specific format that must be properly handled:

```typescript
interface ToolResponse {
  content: Array<{
    type: "text" | "error" | "data";
    text: string;
    data?: any;
  }>;
}
```

Always check response content type and handle appropriately:

```typescript
const response = await callTool("get_memory_state", {});
if (response.content[0].type === "error") {
  // Handle error
  console.error(response.content[0].text);
} else if (response.content[0].type === "data") {
  // Process data
  const state = JSON.parse(response.content[0].text);
  // Use state...
}
```

## 🚨 Enhanced Error Handling

### Error Categories and Handling

```typescript
// Define error types
enum MemoryErrorType {
  TENSOR_ERROR = "TENSOR_ERROR",
  CAPACITY_ERROR = "CAPACITY_ERROR",
  STATE_ERROR = "STATE_ERROR",
  TOOL_ERROR = "TOOL_ERROR",
  SYSTEM_ERROR = "SYSTEM_ERROR",
  VALIDATION_ERROR = "VALIDATION_ERROR",
  RECOVERY_ERROR = "RECOVERY_ERROR",
}

interface MemoryError extends Error {
  type: MemoryErrorType;
  details?: any;
  recoverable: boolean;
  recommendedAction?: string;
}

class MemoryErrorHandler {
  private static readonly ERROR_MESSAGES = {
    [MemoryErrorType.TENSOR_ERROR]: "Tensor operation failed: {0}",
    [MemoryErrorType.CAPACITY_ERROR]: "Memory capacity exceeded: {0}",
    [MemoryErrorType.STATE_ERROR]: "Invalid memory state: {0}",
    [MemoryErrorType.TOOL_ERROR]: "Tool execution failed: {0}",
    [MemoryErrorType.SYSTEM_ERROR]: "System error occurred: {0}",
    [MemoryErrorType.VALIDATION_ERROR]: "Validation failed: {0}",
    [MemoryErrorType.RECOVERY_ERROR]: "Recovery operation failed: {0}",
  };

  static async handleError(error: MemoryError): Promise<void> {
    console.error(`[${error.type}] ${error.message}`);

    try {
      // Save error state for analysis
      await callTool("save_checkpoint", {
        path: `/backups/error-${Date.now()}.json`,
      });

      switch (error.type) {
        case MemoryErrorType.TENSOR_ERROR:
          await this.handleTensorError(error);
          break;
        case MemoryErrorType.CAPACITY_ERROR:
          await this.handleCapacityError(error);
          break;
        case MemoryErrorType.STATE_ERROR:
          await this.handleStateError(error);
          break;
        case MemoryErrorType.TOOL_ERROR:
          await this.handleToolError(error);
          break;
        case MemoryErrorType.SYSTEM_ERROR:
          await this.handleSystemError(error);
          break;
        case MemoryErrorType.VALIDATION_ERROR:
          await this.handleValidationError(error);
          break;
        case MemoryErrorType.RECOVERY_ERROR:
          await this.handleRecoveryError(error);
          break;
      }

      // Verify recovery
      const state = await callTool("get_memory_state", {});
      if (state.status === "error") {
        throw new Error("Recovery failed to restore stable state");
      }
    } catch (recoveryError) {
      // Critical failure - reinitialize
      console.error("Critical error during recovery:", recoveryError);
      await this.reinitializeSystem();
    }
  }

  private static async handleTensorError(error: MemoryError): Promise<void> {
    await callTool("reset_gradients", {});
    tf.disposeVariables();
    await this.loadLastStableCheckpoint();
  }

  private static async handleCapacityError(error: MemoryError): Promise<void> {
    await callTool("prune_memory", { threshold: 0.8 });
    const state = await callTool("get_memory_state", {});
    if (state.capacity < 0.3) {
      await this.expandMemory();
    }
  }

  private static async handleStateError(error: MemoryError): Promise<void> {
    await this.loadLastStableCheckpoint();
    await this.validateState();
  }

  private static async handleToolError(error: MemoryError): Promise<void> {
    if (error.recoverable) {
      await this.retryToolOperation(error.details);
    } else {
      await this.loadLastStableCheckpoint();
    }
  }

  private static async handleSystemError(error: MemoryError): Promise<void> {
    await this.saveSystemState();
    await this.reinitializeSystem();
  }

  private static async handleValidationError(
    error: MemoryError
  ): Promise<void> {
    await this.correctInvalidState(error.details);
  }

  private static async handleRecoveryError(error: MemoryError): Promise<void> {
    await this.reinitializeSystem();
  }

  private static async loadLastStableCheckpoint(): Promise<void> {
    const recovered = await callTool("load_checkpoint", {
      path: "/backups/last-stable.json",
    });
    await this.validateState();
  }

  private static async validateState(): Promise<void> {
    const state = await callTool("get_memory_state", {});
    if (!this.isValidState(state)) {
      throw new Error("Invalid state after recovery");
    }
  }

  private static async expandMemory(): Promise<void> {
    await callTool("init_model", {
      memorySlots: currentSlots * 2,
      preserveState: true,
    });
  }

  private static async reinitializeSystem(): Promise<void> {
    await callTool("init_model", {
      inputDim: 768,
      memorySlots: 10000,
      transformerLayers: 8,
    });
  }
}
```

### Usage Example with Error Handling

```typescript
try {
  const result = await performMemoryOperation();
} catch (error) {
  await MemoryErrorHandler.handleError({
    type: determineErrorType(error),
    message: error.message,
    details: error.details,
    recoverable: isRecoverable(error),
    recommendedAction: getRecommendedAction(error),
  });
}
```

## 🔍 Help Tool

```typescript
interface HelpParams {
  tool?: string; // Specific tool name to get help for
  category?: string; // Category of tools to explore
  showExamples?: boolean; // Include usage examples
  verbose?: boolean; // Include detailed descriptions
}

// Get help for all tools
const help = await callTool("help", {});

// Get help for specific tool
const forwardPassHelp = await callTool("help", {
  tool: "forward_pass",
  showExamples: true,
});

// Get help for category
const maintenanceHelp = await callTool("help", {
  category: "maintenance",
  verbose: true,
});
```

### Help Tool Response Format

```typescript
interface HelpResponse {
  content: Array<{
    type: "text" | "example" | "error";
    text: string;
    data?: {
      tool?: {
        name: string;
        description: string;
        parameters: Record<
          string,
          {
            type: string;
            description: string;
            required: boolean;
            default?: any;
          }
        >;
        examples: string[];
        errors: {
          type: string;
          description: string;
          recovery: string;
        }[];
      };
      category?: {
        name: string;
        tools: string[];
        description: string;
      };
    };
  }>;
}
```

### Help Categories

1. **Memory Operations**

   - init_model
   - forward_pass
   - train_step

2. **State Management**

   - get_memory_state
   - save_checkpoint
   - load_checkpoint

3. **Maintenance**
   - prune_memory
   - reset_gradients
   - manifold_step

### Interactive Help Example

```typescript
// Get interactive help
const helpSession = await callTool("help", {
  interactive: true,
  context: {
    lastOperation: "forward_pass",
    errorEncountered: "TENSOR_ERROR",
    currentState: memoryState,
  },
});

// Help tool will suggest relevant actions:
{
  content: [
    {
      type: "text",
      text: "It looks like you encountered a tensor error during forward_pass. Here are recommended steps:",
      data: {
        recommendations: [
          {
            step: "Reset gradients",
            tool: "reset_gradients",
            reason: "Clear any corrupted tensor state",
          },
          {
            step: "Load checkpoint",
            tool: "load_checkpoint",
            reason: "Restore last known good state",
          },
          {
            step: "Retry forward pass",
            tool: "forward_pass",
            reason: "With clean tensor state",
          },
        ],
      },
    },
  ];
}
```

This enhanced error handling and help system provides:

1. Structured error categorization
2. Detailed error messages and recovery steps
3. Automatic error recovery procedures
4. Interactive help and guidance
5. Context-aware recommendations
6. Comprehensive tool discovery
7. Example-based learning

LLMs can use this system to:

1. Understand available tools
2. Handle errors appropriately
3. Recover from failures
4. Learn from examples
5. Get contextual help
6. Discover best practices
7. Maintain system stability

### Automatic Memory Maintenance Loop

```typescript
class MaintenanceLoop {
  private static readonly MAINTENANCE_INTERVAL = 5 * 60 * 1000; // 5 minutes
  private static readonly BACKUP_INTERVAL = 15 * 60 * 1000; // 15 minutes

  async startLoop(): Promise<void> {
    // Initialize maintenance components
    const memoryManager = MemoryManager.getInstance();
    const maintenance = AutomaticMemoryMaintenance.getInstance();

    // Start periodic maintenance
    setInterval(async () => {
      await memoryManager.wrapWithMemoryManagementAsync(async () => {
        // Check system health
        const health = await this.checkSystemHealth();

        // Perform maintenance if needed
        if (health.needsMaintenance) {
          await maintenance.performMaintenance();
        }

        // Update metrics
        await this.updateMetrics(health);
      });
    }, MaintenanceLoop.MAINTENANCE_INTERVAL);

    // Start periodic backups
    setInterval(async () => {
      await this.secureBackup();
    }, MaintenanceLoop.BACKUP_INTERVAL);
  }

  private async checkSystemHealth(): Promise<SystemHealth> {
    return {
      tensorCount: tf.memory().numTensors,
      memoryUsage: tf.memory().numBytes,
      surpriseScore: (await model.getMemoryState()).stats.surpriseScore,
      capacity: (await model.getMemoryState()).capacity,
      errorRate: this.calculateErrorRate(),
    };
  }

  private async secureBackup(): Promise<void> {
    const memoryManager = MemoryManager.getInstance();
    await memoryManager.wrapWithMemoryManagementAsync(async () => {
      const state = await model.getMemorySnapshot();
      const encrypted = memoryManager.encryptTensor(state);
      await saveToFile(encrypted, `/backups/auto-${Date.now()}.json`);
    });
  }
}
```

### Memory Management Best Practices

1. **Tensor Management**

   ```typescript
   // ❌ Bad - Unmanaged tensor operations
   const result = model.forward(input);

   // ✅ Good - Managed tensor operations
   const result = memoryManager.wrapWithMemoryManagement(() => {
     return model.forward(input);
   });
   ```

2. **Vector Processing**

   ```typescript
   // ❌ Bad - Direct tensor creation
   const tensor = tf.tensor(input);

   // ✅ Good - Processed and validated
   const tensor = vectorProcessor.processInput(input);
   ```

3. **State Management**

   ```typescript
   // ❌ Bad - Unencrypted state
   await saveToFile(state);

   // ✅ Good - Encrypted state
   const encrypted = memoryManager.encryptTensor(state);
   await saveToFile(encrypted);
   ```

4. **Error Recovery**

   ```typescript
   // ❌ Bad - No error recovery
   model.forward(input);

   // ✅ Good - With error recovery
   try {
     await memoryManager.wrapWithMemoryManagementAsync(async () => {
       return model.forward(input);
     });
   } catch (error) {
     await MemoryErrorHandler.handleError(error);
   }
   ```

### Memory Management Workflow

1. **Initialization**

   ```typescript
   // Initialize components
   const memoryManager = MemoryManager.getInstance();
   const vectorProcessor = VectorProcessor.getInstance();
   const maintenance = AutomaticMemoryMaintenance.getInstance();

   // Start maintenance loop
   const loop = new MaintenanceLoop();
   await loop.startLoop();
   ```

2. **Processing Pipeline**

   ```typescript
   const processInput = async (input: string | number[]) => {
     return memoryManager.wrapWithMemoryManagementAsync(async () => {
       // Process input
       const processed = vectorProcessor.processInput(input);

       // Forward pass
       const result = await model.forward(processed, currentState);

       // Update memory
       await model.trainStep(processed, result.predicted);

       // Check health
       const health = await model.getMemoryState();
       if (health.needsMaintenance) {
         await maintenance.performMaintenance();
       }

       return result;
     });
   };
   ```

3. **Cleanup**
   ```typescript
   const cleanup = () => {
     memoryManager.dispose();
     maintenance.dispose();
     tf.disposeVariables();
   };
   ```

### Memory Management Metrics

| Metric         | Description            | Target Range | Action if Out of Range |
| -------------- | ---------------------- | ------------ | ---------------------- |
| Tensor Count   | Number of live tensors | < 1000       | Trigger cleanup        |
| Memory Usage   | Bytes in use           | < 100MB      | Force GC               |
| Surprise Score | Pattern novelty        | 0.3 - 0.85   | Update memory          |
| Capacity       | Available slots        | > 30%        | Prune memory           |
| Error Rate     | Failed operations      | < 1%         | Reset gradients        |
| Backup Age     | Time since last backup | < 15min      | Force backup           |
| GC Frequency   | Cleanup interval       | 5min         | Adjust interval        |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/henryhawke)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/henryhawke)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
