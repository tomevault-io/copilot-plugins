## analysis-claude-code

> enableSteering: boolean;

# Agent类型系统定义 - 自然语言实现规范

## 1. 模块概述

### 1.1 功能定位
Agent类型系统是Claude Code分层多Agent架构的核心类型定义模块，提供完整的TypeScript类型声明和接口规范，确保整个Agent系统的类型安全和接口一致性。

### 1.2 核心职责
- Agent类型定义：定义nO主Agent、I2A交互Agent、UH1用户处理Agent、KN5知识处理Agent的完整类型
- 接口规范制定：制定Agent间通信、状态管理、生命周期管理的标准接口
- 类型安全保障：通过严格的类型检查确保Agent系统的可靠性和可维护性
- 扩展性支持：提供灵活的类型扩展机制，支持新Agent类型的快速集成
- 性能优化：通过类型推断和编译时优化提升运行时性能

### 1.3 设计目标
实现100%类型覆盖率、零运行时类型错误、完整的IDE智能提示支持、高性能的类型推断和清晰的类型文档。

## 2. 接口定义

### 2.1 输入输出规范

#### 核心Agent接口
```typescript
// 基础Agent接口
interface IAgent {
  readonly id: string;
  readonly type: AgentType;
  readonly capabilities: AgentCapability[];
  readonly status: AgentStatus;
  readonly metadata: AgentMetadata;
  
  // 生命周期方法
  initialize(config: AgentConfiguration): Promise<void>;
  start(): Promise<void>;
  stop(): Promise<void>;
  dispose(): Promise<void>;
  
  // 状态管理
  getStatus(): AgentStatus;
  getHealth(): Promise<AgentHealth>;
  getMetrics(): AgentMetrics;
  
  // 配置管理
  updateConfiguration(config: Partial<AgentConfiguration>): Promise<void>;
  getConfiguration(): AgentConfiguration;
}

// Agent工厂接口
interface IAgentFactory {
  createAgent(type: AgentType, config: AgentConfiguration): Promise<IAgent>;
  destroyAgent(agentId: string): Promise<void>;
  listAgents(filter?: AgentFilter): Promise<IAgent[]>;
  getAgentById(id: string): Promise<IAgent | null>;
}
```

### 2.2 参数验证规则
- agentId: 必须是UUID v4格式，长度36字符
- AgentType: 枚举值，限定为'nO'|'I2A'|'UH1'|'KN5'
- AgentConfiguration: 必须包含必填字段和有效的配置值
- AgentCapability: 字符串数组，每个能力标识符长度1-50字符
- AgentMetrics: 数值类型必须非负，性能指标必须在合理范围内

### 2.3 返回格式定义
```typescript
// Agent状态返回格式
interface AgentStatusResponse {
  success: boolean;
  data: AgentStatus;
  timestamp: number;
  metadata?: Record<string, any>;
}

// Agent操作结果格式
interface AgentOperationResult<T = any> {
  success: boolean;
  data?: T;
  error?: AgentError;
  duration: number;
  metadata: OperationMetadata;
}
```

## 3. 核心逻辑

### 3.1 处理流程描述

#### Agent类型层次结构
```typescript
// Agent类型枚举
enum AgentType {
  MAIN = 'nO',           // 主Agent循环
  INTERACTION = 'I2A',   // 交互Agent
  USER_HANDLER = 'UH1',  // 用户处理Agent
  KNOWLEDGE = 'KN5'      // 知识处理Agent
}

// Agent状态枚举
enum AgentStatus {
  CREATED = 'created',
  INITIALIZING = 'initializing',
  READY = 'ready',
  RUNNING = 'running',
  BUSY = 'busy',
  PAUSED = 'paused',
  ERROR = 'error',
  STOPPING = 'stopping',
  STOPPED = 'stopped',
  DISPOSED = 'disposed'
}

// Agent能力类型
type AgentCapability = 
  | 'natural_language_processing'
  | 'task_planning'
  | 'tool_execution'
  | 'user_interaction'
  | 'knowledge_retrieval'
  | 'context_management'
  | 'error_handling'
  | 'performance_monitoring';
```

### 3.2 关键算法说明

#### Agent状态转换逻辑
```typescript
// Agent状态机定义
class AgentStateMachine {
  private static readonly VALID_TRANSITIONS: Record<AgentStatus, AgentStatus[]> = {
    [AgentStatus.CREATED]: [AgentStatus.INITIALIZING, AgentStatus.ERROR],
    [AgentStatus.INITIALIZING]: [AgentStatus.READY, AgentStatus.ERROR],
    [AgentStatus.READY]: [AgentStatus.RUNNING, AgentStatus.STOPPING],
    [AgentStatus.RUNNING]: [AgentStatus.BUSY, AgentStatus.PAUSED, AgentStatus.STOPPING, AgentStatus.ERROR],
    [AgentStatus.BUSY]: [AgentStatus.RUNNING, AgentStatus.ERROR],
    [AgentStatus.PAUSED]: [AgentStatus.RUNNING, AgentStatus.STOPPING],
    [AgentStatus.ERROR]: [AgentStatus.READY, AgentStatus.STOPPING],
    [AgentStatus.STOPPING]: [AgentStatus.STOPPED],
    [AgentStatus.STOPPED]: [AgentStatus.DISPOSED],
    [AgentStatus.DISPOSED]: []
  };
  
  static isValidTransition(from: AgentStatus, to: AgentStatus): boolean {
    return this.VALID_TRANSITIONS[from]?.includes(to) ?? false;
  }
}
```

### 3.3 数据结构定义

#### 核心数据结构
```typescript
// Agent基础元数据
interface AgentMetadata {
  name: string;
  version: string;
  description: string;
  author: string;
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
  capabilities: AgentCapability[];
  dependencies: string[];
  configuration: AgentConfiguration;
}

// Agent配置类型
interface AgentConfiguration {
  // 基础配置
  id: string;
  type: AgentType;
  name: string;
  description?: string;
  
  // 性能配置
  performance: {
    maxConcurrency: number;
    timeout: number;
    retryAttempts: number;
    memoryLimit: number;
    cpuLimit: number;
  };
  
  // 通信配置
  communication: {
    messageQueueSize: number;
    heartbeatInterval: number;
    communicationTimeout: number;
    enableSteering: boolean;
  };
  
  // 工具配置
  tools: {
    enabled: string[];
    disabled: string[];
    permissions: ToolPermissionMatrix;
  };
  
  // 日志配置
  logging: {
    level: LogLevel;
    enableMetrics: boolean;
    enableTracing: boolean;
  };
}
```

## 4. 状态管理

### 4.1 内部状态定义

#### Agent状态管理器
```typescript
// Agent状态接口
interface AgentState {
  id: string;
  type: AgentType;
  status: AgentStatus;
  currentTask?: TaskReference;
  assignedTasks: TaskReference[];
  performance: PerformanceMetrics;
  health: HealthMetrics;
  lastActivity: Date;
  uptime: number;
  
  // 状态变更历史
  statusHistory: StatusChangeEvent[];
  
  // 错误信息
  lastError?: AgentError;
  errorCount: number;
}

// 性能指标
interface PerformanceMetrics {
  cpu: {
    usage: number;      // CPU使用率 (0-100)
    peak: number;       // 峰值CPU使用率
    average: number;    // 平均CPU使用率
  };
  
  memory: {
    used: number;       // 已使用内存 (bytes)
    allocated: number;  // 分配的内存 (bytes)
    peak: number;       // 峰值内存使用
  };
  
  tasks: {
    completed: number;  // 已完成任务数
    failed: number;     // 失败任务数
    active: number;     // 活跃任务数
    averageTime: number; // 平均任务执行时间
  };
  
  network: {
    messagesReceived: number;
    messagesSent: number;
    bytesReceived: number;
    bytesSent: number;
  };
}

// 健康指标
interface HealthMetrics {
  status: HealthStatus;
  score: number;        // 健康分数 (0-100)
  uptime: number;       // 运行时间 (milliseconds)
  lastCheck: Date;      // 最后检查时间
  
  checks: {
    connectivity: boolean;  // 连接性检查
    responsiveness: boolean; // 响应性检查
    resourceUsage: boolean;  // 资源使用检查
    errorRate: boolean;      // 错误率检查
  };
  
  warnings: string[];
  errors: string[];
}
```

### 4.2 状态生命周期

#### Agent生命周期阶段
```typescript
// 生命周期事件
interface LifecycleEvent {
  type: LifecycleEventType;
  timestamp: Date;
  agentId: string;
  phase: LifecyclePhase;
  data?: Record<string, any>;
  error?: Error;
}

enum LifecycleEventType {
  CREATION = 'creation',
  INITIALIZATION = 'initialization',
  STARTUP = 'startup',
  READY = 'ready',
  TASK_ASSIGNMENT = 'task_assignment',
  TASK_COMPLETION = 'task_completion',
  ERROR_OCCURRED = 'error_occurred',
  RECOVERY = 'recovery',
  SHUTDOWN = 'shutdown',
  DISPOSAL = 'disposal'
}

enum LifecyclePhase {
  CREATION = 'creation',
  INITIALIZATION = 'initialization',
  RUNTIME = 'runtime',
  TERMINATION = 'termination'
}
```

### 4.3 持久化策略

#### Agent状态持久化
```typescript
// 状态持久化接口
interface IAgentStatePersistence {
  saveState(agentId: string, state: AgentState): Promise<void>;
  loadState(agentId: string): Promise<AgentState | null>;
  deleteState(agentId: string): Promise<void>;
  
  // 批量操作
  saveStates(states: Map<string, AgentState>): Promise<void>;
  loadStates(agentIds: string[]): Promise<Map<string, AgentState>>;
  
  // 查询操作
  queryStates(filter: StateFilter): Promise<AgentState[]>;
  getStateHistory(agentId: string, timeRange: TimeRange): Promise<StateSnapshot[]>;
}

// 状态快照
interface StateSnapshot {
  timestamp: Date;
  agentId: string;
  state: AgentState;
  checksum: string;
  version: string;
}
```

## 5. 异常处理

### 5.1 异常分类体系

#### Agent异常类型层次
```typescript
// 基础Agent异常
abstract class AgentError extends Error {
  public readonly code: string;
  public readonly agentId: string;
  public readonly timestamp: Date;
  public readonly context: Record<string, any>;
  
  constructor(
    message: string,
    code: string,
    agentId: string,
    context: Record<string, any> = {}
  ) {
    super(message);
    this.name = this.constructor.name;
    this.code = code;
    this.agentId = agentId;
    this.timestamp = new Date();
    this.context = context;
  }
}

// 具体异常类型
class AgentInitializationError extends AgentError {
  constructor(agentId: string, cause: string, context: Record<string, any> = {}) {
    super(`Agent initialization failed: ${cause}`, 'AGENT_INIT_ERROR', agentId, context);
  }
}

class AgentCommunicationError extends AgentError {
  constructor(agentId: string, target: string, context: Record<string, any> = {}) {
    super(`Communication failed with ${target}`, 'AGENT_COMM_ERROR', agentId, context);
  }
}

class AgentTaskExecutionError extends AgentError {
  public readonly taskId: string;
  
  constructor(agentId: string, taskId: string, cause: string, context: Record<string, any> = {}) {
    super(`Task execution failed: ${cause}`, 'AGENT_TASK_ERROR', agentId, context);
    this.taskId = taskId;
  }
}

class AgentResourceExhaustionError extends AgentError {
  public readonly resource: string;
  public readonly limit: number;
  public readonly current: number;
  
  constructor(
    agentId: string, 
    resource: string, 
    limit: number, 
    current: number,
    context: Record<string, any> = {}
  ) {
    super(`Resource exhaustion: ${resource} (${current}/${limit})`, 'AGENT_RESOURCE_ERROR', agentId, context);
    this.resource = resource;
    this.limit = limit;
    this.current = current;
  }
}
```

### 5.2 监控日志策略

#### Agent监控指标
```typescript
// 监控指标收集器
interface IAgentMetricsCollector {
  recordEvent(agentId: string, event: AgentEvent): void;
  recordMetric(agentId: string, metric: string, value: number): void;
  recordError(agentId: string, error: AgentError): void;
  
  getMetrics(agentId: string): Promise<AgentMetrics>;
  getAggregatedMetrics(filter: MetricsFilter): Promise<AggregatedMetrics>;
}

// Agent事件类型
interface AgentEvent {
  type: AgentEventType;
  timestamp: Date;
  agentId: string;
  data: Record<string, any>;
  tags: string[];
}

enum AgentEventType {
  STATUS_CHANGE = 'status_change',
  TASK_STARTED = 'task_started',
  TASK_COMPLETED = 'task_completed',
  TASK_FAILED = 'task_failed',
  COMMUNICATION_SENT = 'communication_sent',
  COMMUNICATION_RECEIVED = 'communication_received',
  ERROR_OCCURRED = 'error_occurred',
  RECOVERY_STARTED = 'recovery_started',
  PERFORMANCE_THRESHOLD = 'performance_threshold'
}
```

### 5.3 错误恢复机制

#### Agent恢复策略
```typescript
// 恢复策略定义
interface RecoveryStrategy {
  name: string;
  condition: (error: AgentError, context: AgentContext) => boolean;
  execute: (agentId: string, error: AgentError) => Promise<RecoveryResult>;
  maxAttempts: number;
  backoffStrategy: BackoffStrategy;
}

// 恢复结果
interface RecoveryResult {
  success: boolean;
  newStatus: AgentStatus;
  message: string;
  nextAction?: RecoveryAction;
  retryAfter?: number;
}

// 恢复管理器
interface IAgentRecoveryManager {
  registerStrategy(strategy: RecoveryStrategy): void;
  executeRecovery(agentId: string, error: AgentError): Promise<RecoveryResult>;
  getRecoveryHistory(agentId: string): Promise<RecoveryAttempt[]>;
}
```

## 6. 性能要求

### 6.1 响应时间目标
- **Agent创建**: < 100ms
- **状态查询**: < 10ms
- **配置更新**: < 50ms
- **健康检查**: < 20ms
- **指标收集**: < 5ms

### 6.2 并发处理能力
- **同时活跃Agent数**: 支持1000+个Agent实例
- **状态更新频率**: 每秒10000次状态变更
- **消息吞吐量**: 每秒处理100000条Agent间消息
- **指标收集频率**: 每秒收集50000个性能指标

### 6.3 资源使用限制
- **类型检查开销**: < 编译时间的5%
- **运行时类型验证**: < 方法执行时间的1%
- **内存占用**: 每个Agent类型定义 < 1KB
- **类型推断时间**: < 100ms per file

## 7. 安全考虑

### 7.1 权限控制机制

#### Agent权限模型
```typescript
// Agent权限接口
interface AgentPermission {
  agentId: string;
  permissions: Permission[];
  restrictions: Restriction[];
  grantedAt: Date;
  expiresAt?: Date;
  grantedBy: string;
}

// 权限定义
interface Permission {
  action: string;      // 'read', 'write', 'execute', 'create', 'delete'
  resource: string;    // 资源标识符
  conditions?: PermissionCondition[];
}

// 权限验证器
interface IAgentPermissionValidator {
  validatePermission(agentId: string, action: string, resource: string): Promise<boolean>;
  grantPermission(agentId: string, permission: Permission): Promise<void>;
  revokePermission(agentId: string, permission: Permission): Promise<void>;
  listPermissions(agentId: string): Promise<Permission[]>;
}
```

### 7.2 数据安全保护

#### 安全类型定义
```typescript
// 敏感数据标记
type Sensitive<T> = T & { readonly __sensitive: true };
type Encrypted<T> = T & { readonly __encrypted: true };

// 安全上下文
interface SecurityContext {
  userId: string;
  sessionId: string;
  permissions: Permission[];
  securityLevel: SecurityLevel;
  auditTrail: AuditEvent[];
}

enum SecurityLevel {
  PUBLIC = 0,
  INTERNAL = 1,
  CONFIDENTIAL = 2,
  SECRET = 3,
  TOP_SECRET = 4
}
```

### 7.3 攻击防护措施

#### 类型安全防护
```typescript
// 输入验证装饰器
function ValidateInput<T extends Record<string, any>>(schema: Schema<T>) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function (...args: any[]) {
      // 运行时类型验证
      for (let i = 0; i < args.length; i++) {
        if (!schema.validate(args[i])) {
          throw new TypeValidationError(`Invalid input type for parameter ${i}`);
        }
      }
      
      return originalMethod.apply(this, args);
    };
  };
}

// 输出清理装饰器
function SanitizeOutput<T>(sanitizer: (value: T) => T) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = function (...args: any[]) {
      const result = originalMethod.apply(this, args);
      return sanitizer(result);
    };
  };
}
```

## 8. 依赖关系

### 8.1 上游依赖模块

#### 类型系统依赖
```typescript
// 基础类型导入
import type { 
  Message, 
  MessageContent, 
  ToolResult 
} from '../core/message-types';

import type { 
  Configuration, 
  ConfigurationSection 
} from '../config/config-types';

import type { 
  Task, 
  TaskPlan, 
  TaskStep 
} from '../tools/task-types';

// 工具类型依赖
import type { 
  Tool, 
  ToolPermission, 
  ToolResult 
} from '../tools/tool-types';

// UI类型依赖
import type { 
  ComponentProps, 
  EventHandler 
} from '../ui/ui-types';
```

### 8.2 下游调用接口

#### 类型导出接口
```typescript
// 主要类型导出
export type {
  // Agent核心类型
  IAgent,
  AgentType,
  AgentStatus,
  AgentConfiguration,
  AgentMetadata,
  AgentState,
  
  // Agent工厂类型
  IAgentFactory,
  AgentFilter,
  
  // 性能和健康类型
  PerformanceMetrics,
  HealthMetrics,
  HealthStatus,
  
  // 事件和生命周期类型
  AgentEvent,
  LifecycleEvent,
  StatusChangeEvent,
  
  // 错误和恢复类型
  AgentError,
  RecoveryStrategy,
  RecoveryResult,
  
  // 权限和安全类型
  AgentPermission,
  SecurityContext,
  SecurityLevel
};

// 实用类型导出
export type {
  Sensitive,
  Encrypted,
  AgentID,
  TaskReference,
  MetricsFilter,
  TimeRange
};
```

### 8.3 配置依赖项目

#### 类型配置定义
```typescript
// TypeScript编译器配置
interface TypeScriptConfig {
  compilerOptions: {
    strict: true;
    noImplicitAny: true;
    strictNullChecks: true;
    strictFunctionTypes: true;
    noImplicitReturns: true;
    noImplicitThis: true;
    exactOptionalPropertyTypes: true;
  };
  include: string[];
  exclude: string[];
}

// 类型检查配置
interface TypeCheckingConfig {
  enableRuntimeValidation: boolean;
  validateAgentInterfaces: boolean;
  enforcePermissionTypes: boolean;
  enablePerformanceTypes: boolean;
}
```

## 9. 测试验证

### 9.1 单元测试规范

#### 类型测试框架
```typescript
describe('Agent Types', () => {
  describe('AgentConfiguration', () => {
    it('should validate required fields', () => {
      const config: AgentConfiguration = {
        id: 'test-agent-001',
        type: AgentType.INTERACTION,
        name: 'Test Agent',
        performance: {
          maxConcurrency: 10,
          timeout: 30000,
          retryAttempts: 3,
          memoryLimit: 512 * 1024 * 1024,
          cpuLimit: 80
        },
        communication: {
          messageQueueSize: 1000,
          heartbeatInterval: 5000,
          communicationTimeout: 10000,
          enableSteering: true
        },
        tools: {
          enabled: ['read', 'write'],
          disabled: [],
          permissions: new Map()
        },
        logging: {
          level: LogLevel.INFO,
          enableMetrics: true,
          enableTracing: false
        }
      };
      
      expect(isValidAgentConfiguration(config)).toBe(true);
    });
  });
  
  describe('Type Guards', () => {
    it('should correctly identify agent types', () => {
      const mainAgent = { type: AgentType.MAIN };
      expect(isMainAgent(mainAgent)).toBe(true);
      expect(isInteractionAgent(mainAgent)).toBe(false);
    });
  });
});
```

#### 类型推断测试
```typescript
// 编译时类型测试
type Test_AgentType_Inference = AgentType extends string ? true : false;
type Test_AgentStatus_Union = AgentStatus extends 
  | 'created' 
  | 'initializing' 
  | 'ready' 
  | 'running' 
  | 'busy' 
  | 'paused' 
  | 'error' 
  | 'stopping' 
  | 'stopped' 
  | 'disposed' 
  ? true 
  : false;

// 运行时类型验证测试
describe('Runtime Type Validation', () => {
  it('should validate agent state transitions', () => {
    expect(AgentStateMachine.isValidTransition(
      AgentStatus.READY,
      AgentStatus.RUNNING
    )).toBe(true);
    
    expect(AgentStateMachine.isValidTransition(
      AgentStatus.DISPOSED,
      AgentStatus.RUNNING
    )).toBe(false);
  });
});
```

### 9.2 集成测试场景

#### 类型系统集成测试
```typescript
describe('Agent Type System Integration', () => {
  it('should maintain type safety across module boundaries', async () => {
    // 创建Agent实例
    const factory = new AgentFactory();
    const agent = await factory.createAgent(AgentType.INTERACTION, {
      // ... configuration
    });
    
    // 类型安全的状态检查
    expect(agent.getStatus()).toSatisfy(isValidAgentStatus);
    
    // 类型安全的配置更新
    await agent.updateConfiguration({
      performance: {
        maxConcurrency: 20
      }
    });
    
    // 验证类型一致性
    const updatedConfig = agent.getConfiguration();
    expect(updatedConfig.performance.maxConcurrency).toBe(20);
  });
  
  it('should handle agent communication with type safety', async () => {
    const agent1 = await createTestAgent(AgentType.INTERACTION);
    const agent2 = await createTestAgent(AgentType.USER_HANDLER);
    
    // 类型安全的消息传递
    const message: AgentMessage = {
      id: generateMessageId(),
      from: agent1.id,
      to: agent2.id,
      type: MessageType.TASK_REQUEST,
      payload: {
        taskId: 'test-task-001',
        description: 'Test task'
      },
      timestamp: new Date()
    };
    
    await agent1.sendMessage(message);
    
    // 验证消息类型安全
    expect(message).toSatisfy(isValidAgentMessage);
  });
});
```

### 9.3 验收标准定义

#### 类型安全验收标准
- **编译时类型检查**: 100%通过，零类型错误
- **运行时类型验证**: > 99.9%成功率
- **类型推断准确性**: 100%准确的IDE智能提示
- **类型文档覆盖率**: 100%的公开接口都有完整类型声明

#### 性能验收标准
- **类型检查时间**: < 编译总时间的5%
- **运行时验证开销**: < 方法执行时间的1%
- **内存占用**: 类型定义 < 总内存的0.1%
- **智能提示响应**: < 100ms

#### 可维护性验收标准
- **类型一致性**: 100%的接口保持一致
- **向后兼容性**: 新版本兼容旧版本的类型定义
- **扩展性**: 支持新Agent类型的无缝集成

## 10. AI编译器指令

### 10.1 实现语言选择
- **主语言**: TypeScript 5.0+，启用最严格的类型检查选项
- **类型定义**: 使用interface和type进行精确的类型声明
- **泛型约束**: 广泛使用泛型和条件类型确保类型安全
- **装饰器**: 使用装饰器实现运行时类型验证和元数据注入

### 10.2 代码风格要求
- **类型命名**: 使用PascalCase命名接口和类型
- **枚举定义**: 使用字符串枚举确保序列化安全
- **注释标准**: 每个公开类型都必须有完整的JSDoc注释
- **导入导出**: 使用type-only imports确保编译优化

### 10.3 部署方式规范
- **类型声明文件**: 生成完整的.d.ts声明文件
- **包发布**: 支持npm包形式的类型定义分发
- **版本管理**: 类型定义版本与主版本保持同步
- **文档生成**: 自动生成类型API文档

### 具体实现示例

#### 完整的Agent类型系统实现
```typescript
// src/types/agent.ts

// 基础类型导入
import type { EventEmitter } from 'events';
import type { 
  Message, 
  MessageContent, 
  ToolResult 
} from './message-types';

// Agent类型枚举
export enum AgentType {
  MAIN = 'nO',
  INTERACTION = 'I2A',
  USER_HANDLER = 'UH1',
  KNOWLEDGE = 'KN5'
}

// Agent状态枚举
export enum AgentStatus {
  CREATED = 'created',
  INITIALIZING = 'initializing',
  READY = 'ready',
  RUNNING = 'running',
  BUSY = 'busy',
  PAUSED = 'paused',
  ERROR = 'error',
  STOPPING = 'stopping',
  STOPPED = 'stopped',
  DISPOSED = 'disposed'
}

// 核心Agent接口
export interface IAgent extends EventEmitter {
  readonly id: AgentID;
  readonly type: AgentType;
  readonly capabilities: readonly AgentCapability[];
  readonly status: AgentStatus;
  readonly metadata: Readonly<AgentMetadata>;
  
  // 生命周期方法
  initialize(config: AgentConfiguration): Promise<void>;
  start(): Promise<void>;
  stop(): Promise<void>;
  dispose(): Promise<void>;
  
  // 状态管理
  getStatus(): AgentStatus;
  getHealth(): Promise<HealthMetrics>;
  getMetrics(): Promise<PerformanceMetrics>;
  
  // 任务管理
  assignTask(task: TaskReference): Promise<void>;
  executeTask(taskId: string): AsyncGenerator<TaskProgress>;
  cancelTask(taskId: string): Promise<void>;
  
  // 通信管理
  sendMessage(message: AgentMessage): Promise<void>;
  receiveMessage(message: AgentMessage): Promise<void>;
  
  // 配置管理
  updateConfiguration(config: Partial<AgentConfiguration>): Promise<void>;
  getConfiguration(): Readonly<AgentConfiguration>;
}

// Agent工厂接口
export interface IAgentFactory {
  createAgent<T extends AgentType>(
    type: T, 
    config: AgentConfigurationByType<T>
  ): Promise<AgentByType<T>>;
  
  destroyAgent(agentId: AgentID): Promise<void>;
  listAgents(filter?: AgentFilter): Promise<IAgent[]>;
  getAgentById(id: AgentID): Promise<IAgent | null>;
}

// 条件类型：根据Agent类型返回对应的Agent实例
export type AgentByType<T extends AgentType> = 
  T extends AgentType.MAIN ? MainAgent :
  T extends AgentType.INTERACTION ? InteractionAgent :
  T extends AgentType.USER_HANDLER ? UserHandlerAgent :
  T extends AgentType.KNOWLEDGE ? KnowledgeAgent :
  never;

// 条件类型：根据Agent类型返回对应的配置
export type AgentConfigurationByType<T extends AgentType> = 
  T extends AgentType.MAIN ? MainAgentConfiguration :
  T extends AgentType.INTERACTION ? InteractionAgentConfiguration :
  T extends AgentType.USER_HANDLER ? UserHandlerAgentConfiguration :
  T extends AgentType.KNOWLEDGE ? KnowledgeAgentConfiguration :
  never;

// 类型保护函数
export function isAgent(obj: any): obj is IAgent {
  return obj && 
    typeof obj.id === 'string' &&
    Object.values(AgentType).includes(obj.type) &&
    typeof obj.getStatus === 'function';
}

export function isMainAgent(agent: IAgent): agent is MainAgent {
  return agent.type === AgentType.MAIN;
}

export function isInteractionAgent(agent: IAgent): agent is InteractionAgent {
  return agent.type === AgentType.INTERACTION;
}

export function isUserHandlerAgent(agent: IAgent): agent is UserHandlerAgent {
  return agent.type === AgentType.USER_HANDLER;
}

export function isKnowledgeAgent(agent: IAgent): agent is KnowledgeAgent {
  return agent.type === AgentType.KNOWLEDGE;
}

// 实用类型
export type AgentID = string & { readonly __brand: 'AgentID' };
export type TaskReference = string & { readonly __brand: 'TaskReference' };

// 创建类型安全的ID生成器
export function createAgentId(): AgentID {
  return crypto.randomUUID() as AgentID;
}

export function createTaskReference(taskId: string): TaskReference {
  return taskId as TaskReference;
}

// 高级类型操作
export type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
export type RequiredBy<T, K extends keyof T> = T & Required<Pick<T, K>>;

// 配置验证类型
export type ValidatedConfiguration<T> = T & { readonly __validated: true };

// 验证装饰器类型
export type ConfigValidator<T> = (config: T) => config is ValidatedConfiguration<T>;

// 异步操作结果类型
export type AsyncResult<T, E = Error> = Promise<
  | { success: true; data: T }
  | { success: false; error: E }
>;

// 事件类型映射
export interface AgentEventMap {
  'status-change': [AgentStatus, AgentStatus];
  'task-assigned': [TaskReference];
  'task-completed': [TaskReference, any];
  'task-failed': [TaskReference, Error];
  'message-received': [AgentMessage];
  'message-sent': [AgentMessage];
  'error': [AgentError];
  'health-change': [HealthMetrics];
}

// 类型安全的事件发射器
export interface TypedEventEmitter<T extends Record<string, any[]>> {
  on<K extends keyof T>(event: K, listener: (...args: T[K]) => void): this;
  emit<K extends keyof T>(event: K, ...args: T[K]): boolean;
  off<K extends keyof T>(event: K, listener: (...args: T[K]) => void): this;
}

// 最终的Agent接口，结合类型安全的事件发射器
export interface TypedAgent extends 
  Omit<IAgent, keyof EventEmitter>, 
  TypedEventEmitter<AgentEventMap> {}
```

这个完整的Agent类型系统提供了：

1. **完整的类型安全**: 通过严格的TypeScript类型确保编译时和运行时的类型安全
2. **智能类型推断**: 使用条件类型和泛型实现智能的类型推断
3. **可扩展性**: 通过接口和抽象类型支持新Agent类型的无缝集成
4. **性能优化**: 通过类型品牌和类型保护实现零运行时开销的类型检查
5. **开发体验**: 提供完整的IDE智能提示和错误检查支持

---
> Source: [ThreeFish-AI/analysis_claude_code](https://github.com/ThreeFish-AI/analysis_claude_code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
