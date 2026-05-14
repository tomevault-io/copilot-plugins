## typescript-standards

> name: SmartLead MCP TypeScript Standards

---
name: SmartLead MCP TypeScript Standards
description: TypeScript coding standards, patterns, and best practices for the SmartLead MCP Server
author: LeadMagic Team
version: 1.0.0
# 📝 SmartLead TypeScript Standards & Best Practices

> **Code Quality Guide**: Master TypeScript patterns and standards used in the SmartLead MCP Server. Write clean, type-safe, maintainable code that follows our established conventions.

## 🎯 **Quick Reference**

### **🔥 Core Principles**
1. **Strict Type Safety** → No `any` types, use proper interfaces
2. **Zod Integration** → All inputs validated with Zod schemas
3. **Error Handling** → Use `SmartLeadError` for API errors
4. **Async Patterns** → Consistent async/await usage
5. **Documentation** → JSDoc for public APIs

### **⚡ Quick Checklist**
- [ ] No `any` types used
- [ ] Zod schemas for validation
- [ ] Proper error handling
- [ ] JSDoc for public methods
- [ ] Consistent naming conventions

# SmartLead MCP Server - TypeScript Standards

## 🎯 **TypeScript Configuration**

### **Compiler Settings** (`tsconfig.json`)
The project uses strict TypeScript configuration defined in **[tsconfig.json](mdc:tsconfig.json)**:
- **Target**: ES2022 for modern JavaScript features
- **Module**: ESNext with bundler resolution for optimal tree-shaking
- **Strict Mode**: All strict options enabled for maximum type safety
- **JSX**: React JSX for the installer components in **[src/installer.tsx](mdc:src/installer.tsx)**

### **Key Compiler Options**
```json
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true,
  "noImplicitReturns": true,
  "noFallthroughCasesInSwitch": true,
  "noUncheckedIndexedAccess": true,
  "useUnknownInCatchVariables": true
}
```

## 📝 **Type Definition Patterns**

### **Global Types** (`src/types.ts`)
All shared types and Zod schemas are defined in **[src/types.ts](mdc:src/types.ts)**:
```typescript
// Zod schema with TypeScript type inference
const CreateCampaignSchema = z.object({
  name: z.string(),
  from_email: z.string().email(),
  // ... other fields
});

// Inferred TypeScript type
type CreateCampaignRequest = z.infer<typeof CreateCampaignSchema>;
```

### **Module-Specific Types**
Each module can define its own types when needed:
```typescript
// In src/modules/campaigns/client.ts
interface CampaignResponse {
  id: number;
  name: string;
  status: 'active' | 'paused' | 'completed';
  created_at: string;
}
```

### **MCP Configuration Types** (`src/types/config.ts`)
MCP-specific types are in **[src/types/config.ts](mdc:src/types/config.ts)**:
```typescript
export interface MCPToolResponse {
  content: Array<{
    type: 'text';
    text: string;
  }>;
  isError?: boolean;
}
```

## 🏗️ **Class and Interface Patterns**

### **Base Client Pattern**
All API clients extend from the base class in **[src/client/base.ts](mdc:src/client/base.ts)**:
```typescript
export abstract class BaseSmartLeadClient {
  protected config: SmartLeadConfig;
  
  constructor(config: SmartLeadConfig) {
    this.config = config;
  }
  
  protected async withRetry<T>(operation: () => Promise<T>): Promise<T> {
    // Implementation with proper typing
  }
}
```

### **Module Client Pattern**
Individual modules follow this pattern:
```typescript
export class CampaignManagementClient extends BaseSmartLeadClient {
  async createCampaign(params: CreateCampaignRequest): Promise<CampaignResponse> {
    return this.withRetry(() => this.post('/campaigns', params));
  }
}
```

### **Error Class Pattern**
Custom error classes with proper typing:
```typescript
export class SmartLeadError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly status: number,
    public readonly data?: unknown,
    public readonly isRetryable: boolean = false
  ) {
    super(message);
    this.name = 'SmartLeadError';
  }
}
```

## 🔍 **Type Safety Best Practices**

### **Avoid `any` Types**
❌ **Don't do this:**
```typescript
function handleResponse(data: any): any {
  return data.result;
}
```

✅ **Do this instead:**
```typescript
interface ApiResponse<T> {
  success: boolean;
  data: T;
  message?: string;
}

function handleResponse<T>(data: ApiResponse<T>): T {
  return data.data;
}
```

### **Use Type Guards**
```typescript
function isSmartLeadError(error: unknown): error is SmartLeadError {
  return error instanceof SmartLeadError;
}

// Usage in error handling
catch (error: unknown) {
  if (isSmartLeadError(error)) {
    // TypeScript knows this is SmartLeadError
    console.log(error.code, error.status);
  }
}
```

### **Proper Async/Await Typing**
```typescript
async function fetchCampaign(id: number): Promise<CampaignResponse> {
  try {
    const response = await this.get<CampaignResponse>(`/campaigns/${id}`);
    return response.data;
  } catch (error: unknown) {
    throw this.handleError(error);
  }
}
```

## 📋 **Zod Integration Patterns**

### **Schema Definition**
Schemas are defined in **[src/types.ts](mdc:src/types.ts)** with proper validation:
```typescript
const UpdateCampaignSchema = z.object({
  id: z.number().int().positive('Campaign ID must be positive'),
  name: z.string().min(1, 'Campaign name is required').optional(),
  status: z.enum(['active', 'paused', 'completed']).optional(),
  settings: z.record(z.unknown()).optional()
});

type UpdateCampaignRequest = z.infer<typeof UpdateCampaignSchema>;
```

### **Validation in MCP Tools**
```typescript
async (params) => {
  try {
    const validatedParams = UpdateCampaignSchema.parse(params);
    // validatedParams is now properly typed
    const result = await client.campaigns.updateCampaign(validatedParams);
    return formatSuccessResponse('Campaign updated', result);
  } catch (error) {
    if (error instanceof z.ZodError) {
      // Handle validation errors specifically
      return handleError(new Error(`Validation failed: ${error.message}`));
    }
    return handleError(error);
  }
}
```

## 🔧 **Generic Type Patterns**

### **HTTP Client Generics**
```typescript
class BaseSmartLeadClient {
  protected async get<T>(url: string): Promise<ApiResponse<T>> {
    // Implementation with proper return typing
  }
  
  protected async post<T, U = unknown>(url: string, data: U): Promise<ApiResponse<T>> {
    // Implementation with request and response typing
  }
}
```

### **Tool Registration Generics**
```typescript
interface ToolRegistration<TInput, TOutput> {
  name: string;
  schema: z.ZodSchema<TInput>;
  handler: (params: TInput) => Promise<TOutput>;
}
```

## 🎨 **React/JSX Patterns** (For Installer)

### **Component Props Typing**
In **[src/installer.tsx](mdc:src/installer.tsx)**:
```typescript
interface WelcomeScreenProps {
  onNext: () => void;
}

const WelcomeScreen: React.FC<WelcomeScreenProps> = ({ onNext }) => {
  // Component implementation
};
```

### **State Management Typing**
```typescript
const [clients, setClients] = useState<MCPClient[]>([]);
const [selectedClients, setSelectedClients] = useState<MCPClient[]>([]);
const [step, setStep] = useState<Step>('welcome');
```

### **Event Handler Typing**
```typescript
const handleSelect = (item: { value: string; label: string }) => {
  // Handler implementation with proper typing
};
```

## 🚨 **Error Handling Patterns**

### **Typed Error Responses**
```typescript
interface ErrorResponse {
  success: false;
  error: string;
  code?: string;
  status?: number;
  details?: unknown;
}

function createErrorResponse(error: unknown): ErrorResponse {
  if (error instanceof SmartLeadError) {
    return {
      success: false,
      error: error.message,
      code: error.code,
      status: error.status,
      details: error.data
    };
  }
  
  return {
    success: false,
    error: error instanceof Error ? error.message : 'Unknown error'
  };
}
```

### **Union Types for Responses**
```typescript
type ApiResult<T> = 
  | { success: true; data: T }
  | { success: false; error: string; code?: string };

async function safeApiCall<T>(operation: () => Promise<T>): Promise<ApiResult<T>> {
  try {
    const data = await operation();
    return { success: true, data };
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error'
    };
  }
}
```

## 🔍 **Utility Type Usage**

### **Common Utility Types**
```typescript
// Make all properties optional
type PartialCampaign = Partial<CreateCampaignRequest>;

// Pick specific properties
type CampaignSummary = Pick<CampaignResponse, 'id' | 'name' | 'status'>;

// Omit specific properties
type CampaignUpdate = Omit<CreateCampaignRequest, 'id'>;

// Record types for dynamic objects
type CampaignSettings = Record<string, string | number | boolean>;
```

### **Conditional Types**
```typescript
type ApiMethod<T> = T extends 'GET' 
  ? (url: string) => Promise<unknown>
  : (url: string, data: unknown) => Promise<unknown>;
```

## 📏 **Code Quality Standards**

### **Naming Conventions**
- **Interfaces**: PascalCase (`ApiResponse`, `CampaignData`)
- **Types**: PascalCase (`CreateCampaignRequest`)
- **Enums**: PascalCase (`CampaignStatus`)
- **Functions**: camelCase (`createCampaign`, `handleError`)
- **Variables**: camelCase (`campaignId`, `responseData`)
- **Constants**: UPPER_SNAKE_CASE (`API_BASE_URL`)

### **Import/Export Patterns**
```typescript
// Prefer named exports
export { SmartLeadClient, SmartLeadError };
export type { SmartLeadConfig, CampaignResponse };

// Use type-only imports when appropriate
import type { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { z } from 'zod';
```

### **Documentation Standards**
```typescript
/**
 * Creates a new SmartLead campaign with the specified configuration
 * @param params - Campaign creation parameters
 * @returns Promise resolving to the created campaign data
 * @throws SmartLeadError when API request fails
 */
async createCampaign(params: CreateCampaignRequest): Promise<CampaignResponse> {
  // Implementation
}
```

async createCampaign(params: CreateCampaignRequest): Promise<CampaignResponse> {
  // Implementation
}
```

---
> Source: [LeadMagic/smartlead-mcp-server](https://github.com/LeadMagic/smartlead-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
