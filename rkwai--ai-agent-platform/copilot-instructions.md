## ai-agent-platform

> // ALWAYS structure code around events

# AI Agent Platform Development Guidelines

## Core Development Principles

### 1. Event-First Architecture
```typescript
// ALWAYS structure code around events
interface AgentEvent {
  id: string;
  timestamp: Date;
  agentId: string;
  type: EventType;
  data: Record<string, unknown>;
  metadata: EventMetadata;
}

// NEVER mutate state directly
// GOOD
await eventStore.append(new AgentEvent({
  type: 'TaskStarted',
  data: { taskId, parameters }
}));

// BAD
agent.status = 'running';
await agent.save();
```

### 2. State Management
- All state changes MUST be traceable through events
- State reconstruction MUST be possible from events alone
- Use snapshots for performance, not as source of truth
- Always verify state consistency

### 3. Error Handling & Recovery
```typescript
// ALWAYS handle errors gracefully
try {
  await agent.executeTask(task);
} catch (error) {
  // Log the error event
  await eventStore.append(new AgentEvent({
    type: 'TaskFailed',
    data: { taskId, error: error.message }
  }));
  
  // Attempt recovery
  await agent.attemptRecovery();
  
  // If critical, alert manager
  if (error.critical) {
    await notificationService.alertManager(error);
  }
}
```

### 4. Tool Integration Pattern
```typescript
// ALWAYS use the Tool Registry pattern
interface Tool {
  id: string;
  name: string;
  version: string;
  configure(config: ToolConfig): Promise<void>;
  execute(params: ToolParams): Promise<ToolResult>;
  validate(params: ToolParams): Promise<boolean>;
  handleError(error: Error): Promise<void>;
}

// NEVER directly instantiate tools
// GOOD
const tool = await toolRegistry.get('gmail');
await tool.execute(params);

// BAD
const gmailTool = new GmailTool(config);
```

## Architectural Requirements

### 1. Dashboard Development
```typescript
// Use React with TypeScript
// ALWAYS type your props
interface AgentCardProps {
  agentId: string;
  status: AgentStatus;
  currentTask?: Task;
  metrics: AgentMetrics;
  onAssist: (agentId: string) => Promise<void>;
}

// ALWAYS handle loading and error states
const AgentCard: React.FC<AgentCardProps> = ({ agentId, ...props }) => {
  const { data, error, isLoading } = useAgentData(agentId);
  
  if (isLoading) return <LoadingState />;
  if (error) return <ErrorState error={error} />;
  
  return <AgentCardContent data={data} {...props} />;
};
```

### 2. Event Store Implementation
```typescript
// ALWAYS ensure event store consistency
class EventStore {
  async append(event: AgentEvent): Promise<void> {
    const transaction = await this.db.transaction();
    try {
      // Verify event order
      await this.verifyEventOrder(event);
      
      // Store event
      await transaction.insert('events', event);
      
      // Update projections
      await this.updateProjections(event);
      
      await transaction.commit();
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  }
}
```

### 3. Manager Notification System
```typescript
// ALWAYS use priority levels
enum NotificationPriority {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CRITICAL = 'critical'
}

// ALWAYS support multiple channels
interface NotificationChannel {
  send(notification: Notification): Promise<void>;
  isAvailable(): Promise<boolean>;
  getPreferences(managerId: string): Promise<ChannelPreferences>;
}
```

## Testing Requirements

### 1. Event Testing
```typescript
// ALWAYS test event handling
describe('AgentEventHandler', () => {
  it('should reconstruct state correctly after multiple events', async () => {
    const events = [
      new AgentEvent({ type: 'TaskStarted', data: {...} }),
      new AgentEvent({ type: 'TaskProgressed', data: {...} }),
      new AgentEvent({ type: 'TaskCompleted', data: {...} })
    ];
    
    const state = await stateReconstructor.reconstruct(events);
    expect(state.status).toBe('completed');
  });
});
```

### 2. Tool Testing
```typescript
// ALWAYS mock external tools in tests
jest.mock('../tools/gmail-tool', () => ({
  GmailTool: jest.fn().mockImplementation(() => ({
    execute: jest.fn().mockResolvedValue({ success: true }),
    validate: jest.fn().mockResolvedValue(true)
  }))
}));
```

## Performance Guidelines

### 1. State Reconstruction
```typescript
// ALWAYS use snapshots for performance
class StateReconstructor {
  async reconstruct(agentId: string): Promise<AgentState> {
    // Get latest snapshot
    const snapshot = await this.getLatestSnapshot(agentId);
    
    // Get events after snapshot
    const events = await this.getEventsSinceSnapshot(
      agentId, 
      snapshot.timestamp
    );
    
    // Apply events to snapshot
    return events.reduce(
      (state, event) => this.applyEvent(state, event),
      snapshot.state
    );
  }
}
```

### 2. Dashboard Performance
```typescript
// ALWAYS implement pagination and virtualization
const AgentList: React.FC = () => {
  return (
    <VirtualizedList
      itemCount={totalAgents}
      itemSize={80}
      onItemsRendered={loadMoreAgents}
      overscan={5}
    >
      {({ index, style }) => (
        <AgentCard
          key={agents[index].id}
          style={style}
          {...agents[index]}
        />
      )}
    </VirtualizedList>
  );
};
```

## Error Handling Patterns

### 1. Graceful Degradation
```typescript
// ALWAYS implement fallback behaviors
class ToolExecutor {
  async execute(tool: Tool, params: ToolParams): Promise<ToolResult> {
    try {
      return await tool.execute(params);
    } catch (error) {
      // Try fallback if available
      if (tool.fallback) {
        return await tool.fallback.execute(params);
      }
      
      // Degrade gracefully
      if (tool.degradedMode) {
        return await tool.degradedMode(params);
      }
      
      throw error;
    }
  }
}
```

### 2. Recovery Procedures
```typescript
// ALWAYS implement recovery procedures
class AgentRecovery {
  async recover(agent: Agent): Promise<void> {
    // Log recovery attempt
    await this.eventStore.append(new AgentEvent({
      type: 'RecoveryStarted',
      agentId: agent.id
    }));
    
    // Attempt state reconstruction
    const state = await this.stateReconstructor.reconstruct(agent.id);
    
    // Verify state consistency
    await this.verifyStateConsistency(state);
    
    // Resume from last known good state
    await agent.resume(state);
  }
}
```

## Code Organization

### 1. Module Structure
```
src/
├── core/
│   ├── events/
│   ├── state/
│   └── tools/
├── dashboard/
│   ├── components/
│   └── hooks/
├── services/
│   ├── notification/
│   └── recovery/
└── utils/
```

### 2. Import Guidelines
```typescript
// ALWAYS use absolute imports from src
import { AgentEvent } from '@/core/events';
import { useAgentData } from '@/dashboard/hooks';
import { NotificationService } from '@/services/notification';
```

## Documentation Requirements

### 1. Code Documentation
```typescript
/**
 * Executes a tool with proper error handling and recovery
 * @param tool - The tool to execute
 * @param params - Tool execution parameters
 * @returns Tool execution result
 * @throws ToolExecutionError if both primary and fallback execution fail
 */
async function executeTool(
  tool: Tool,
  params: ToolParams
): Promise<ToolResult> {
  // Implementation
}
```

### 2. Event Documentation
```typescript
/**
 * Event Types
 * 
 * TaskStarted - Emitted when an agent begins a new task
 * TaskProgressed - Emitted when task progress is made
 * TaskCompleted - Emitted when a task is successfully completed
 * TaskFailed - Emitted when a task fails
 * RecoveryStarted - Emitted when agent recovery begins
 * RecoveryCompleted - Emitted when agent recovery succeeds
 */
```

## Monitoring & Logging

### 1. Event Logging
```typescript
// ALWAYS include context in logs
interface EventLog {
  event: AgentEvent;
  context: {
    agentId: string;
    managerId?: string;
    timestamp: Date;
    environment: string;
    version: string;
  };
  metadata: {
    correlationId: string;
    causationId?: string;
  };
}
```

### 2. Metrics Collection
```typescript
// ALWAYS collect performance metrics
interface AgentMetrics {
  taskSuccess: number;
  taskFailure: number;
  averageExecutionTime: number;
  toolUsage: Record<string, number>;
  recoveryAttempts: number;
  managerInterventions: number;
}
```

Follow these guidelines to ensure consistent, reliable, and maintainable code throughout the AI Agent Platform.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rkwai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
