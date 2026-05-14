## api-patterns

> name: SmartLead MCP API Development Patterns

---
name: SmartLead MCP API Development Patterns
description: Development patterns for API clients and MCP tools in the SmartLead server
author: LeadMagic Team
version: 1.0.0
# 🏗️ SmartLead API Development Patterns & Best Practices

> **Developer Guide**: Master the patterns and practices used throughout the SmartLead MCP Server. This guide will help you write consistent, maintainable code that follows our established conventions.

## 🎯 **Quick Reference**

### **🔥 Most Important Patterns**
1. **Base Client Extension** → All API modules extend `BaseSmartLeadClient`
2. **MCP Tool Registration** → Use the standard registration pattern
3. **Zod Schema Validation** → All inputs validated with Zod
4. **Error Handling** → Use `SmartLeadError` for API errors
5. **Type Safety** → Avoid `any`, use proper TypeScript types

## 🏗️ **SmartLead API Client Architecture**

### **🛠️ Base Client Pattern** 
> **File**: [`src/client/base.ts`](mdc:src/client/base.ts)

The foundation of all API interactions:

```typescript
export abstract class BaseSmartLeadClient {
  protected config: SmartLeadConfig;
  
  constructor(config: SmartLeadConfig) {
    this.config = config;
  }
  
  // 🔄 Automatic retry with exponential backoff
  protected async withRetry<T>(operation: () => Promise<T>): Promise<T> {
    // Handles transient failures automatically
  }
  
  // 🌐 HTTP methods with proper typing
  protected async get<T>(url: string): Promise<T> { }
  protected async post<T>(url: string, data: unknown): Promise<T> { }
  protected async put<T>(url: string, data: unknown): Promise<T> { }
  protected async delete<T>(url: string): Promise<T> { }
}
```

**Key Features:**
- ✅ **HTTP request handling** with Axios
- ✅ **Automatic retry logic** with exponential backoff  
- ✅ **Rate limiting** and request queuing
- ✅ **Error handling** with SmartLeadError class
- ✅ **Request/response logging** for debugging

### **🔌 Module Structure Pattern**
> **Example**: [`src/modules/campaigns/client.ts`](mdc:src/modules/campaigns/client.ts)

Each API module follows this pattern:

```typescript
export class CampaignManagementClient extends BaseSmartLeadClient {
  /**
   * 🎯 Create a new campaign
   * @param params - Campaign creation parameters
   * @returns Promise resolving to campaign data
   */
  async createCampaign(params: CreateCampaignRequest): Promise<CampaignResponse> {
    return this.withRetry(() => 
      this.post<CampaignResponse>('/campaigns', params)
    );
  }
  
  /**
   * 📋 List campaigns with filtering
   * @param filters - Optional filtering parameters
   * @returns Promise resolving to campaign list
   */
  async listCampaigns(filters?: CampaignFilters): Promise<CampaignListResponse> {
    const params = new URLSearchParams();
    if (filters?.status) params.append('status', filters.status);
    if (filters?.limit) params.append('limit', filters.limit.toString());
    
    return this.withRetry(() => 
      this.get<CampaignListResponse>(`/campaigns?${params}`)
    );
  }
}
```

## 🛠️ **MCP Tool Development Patterns**

### **🎯 Tool Registration Pattern**
> **Example**: [`src/tools/campaigns.ts`](mdc:src/tools/campaigns.ts)

Standard structure for all MCP tools:

```typescript
export function registerCampaignTools(
  server: McpServer,
  client: SmartLeadClient,
  formatSuccessResponse: (message: string, data: unknown, summary?: string) => MCPToolResponse,
  handleError: (error: unknown) => MCPToolResponse
): void {
  
  // 🎯 Tool Registration
  server.registerTool(
    'smartlead_create_campaign',
    {
      title: 'Create SmartLead Campaign',
      description: 'Create a new email campaign with sequences and settings. Perfect for starting new outreach efforts.',
      inputSchema: CreateCampaignSchema.shape,
    },
    async (params) => {
      try {
        // ✅ 1. Validate input with Zod
        const validatedParams = CreateCampaignSchema.parse(params);
        
        // ✅ 2. Call API method  
        const result = await client.campaigns.createCampaign(validatedParams);
        
        // ✅ 3. Format success response with helpful summary
        const summary = `Created campaign "${validatedParams.name}" with ${validatedParams.sequences?.length || 0} sequences`;
        
        return formatSuccessResponse(
          'Campaign created successfully',
          result,
          summary
        );
      } catch (error) {
        // ✅ 4. Handle errors gracefully
        return handleError(error);
      }
    }
  );
}
```

### **🏷️ Tool Naming Convention**

| **Pattern** | **Example** | **Usage** |
|-------------|-------------|-----------|
| `smartlead_create_*` | `smartlead_create_campaign` | Creating new resources |
| `smartlead_get_*` | `smartlead_get_campaign` | Fetching single items |
| `smartlead_list_*` | `smartlead_list_campaigns` | Fetching multiple items |
| `smartlead_update_*` | `smartlead_update_campaign` | Modifying existing resources |
| `smartlead_delete_*` | `smartlead_delete_campaign` | Removing resources |
| `smartlead_fetch_*` | `smartlead_fetch_analytics` | Getting computed data |

**Specificity Rules:**
- Add `_by_*` for filtering: `smartlead_list_leads_by_campaign`
- Add `_with_*` for includes: `smartlead_get_campaigns_with_analytics`
- Add context when needed: `smartlead_fetch_warmup_stats_by_email_account`

## 📝 **Type Safety Patterns**

### **🛡️ Zod Schema Validation**
> **File**: [`src/types.ts`](mdc:src/types.ts)

All API inputs use Zod schemas for validation:

```typescript
// ✅ Define schema with validation
const CreateCampaignSchema = z.object({
  name: z.string().min(1, 'Campaign name is required'),
  from_email: z.string().email('Valid email address required'),
  sequences: z.array(z.object({
    subject: z.string().min(1, 'Subject is required'),
    body: z.string().min(1, 'Body is required'),
    delay_days: z.number().min(0, 'Delay must be non-negative').max(365, 'Delay cannot exceed 365 days')
  })).min(1, 'At least one sequence is required'),
  settings: z.object({
    daily_limit: z.number().min(1).max(1000).optional(),
    timezone: z.string().optional(),
    track_opens: z.boolean().optional(),
    track_clicks: z.boolean().optional()
  }).optional()
});

// ✅ Infer TypeScript type
type CreateCampaignRequest = z.infer<typeof CreateCampaignSchema>;

// ✅ Export both for use in modules and tools
export { CreateCampaignSchema };
export type { CreateCampaignRequest };
```

### **📊 Response Type Patterns**

Consistent response structures across the API:

```typescript
// ✅ Success Response Pattern
interface SuccessResponse<T = unknown> {
  success: true;
  message?: string;
  data: T;
  meta?: {
    total?: number;
    page?: number;
    limit?: number;
  };
}

// ✅ Error Response Pattern  
interface ErrorResponse {
  success: false;
  error: string;
  code?: string;
  status?: number;
  details?: unknown;
}

// ✅ Union type for API results
type ApiResult<T> = SuccessResponse<T> | ErrorResponse;
```

## 🔄 **Error Handling Patterns**

### **🚨 SmartLeadError Class**
> **File**: [`src/client/base.ts`](mdc:src/client/base.ts)

Custom error class for API-specific errors:

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
  
  // ✅ Helper methods for error classification
  get isAuthError(): boolean {
    return this.status === 401 || this.status === 403;
  }
  
  get isRateLimit(): boolean {
    return this.status === 429;
  }
  
  get isServerError(): boolean {
    return this.status >= 500;
  }
}
```

### **🔄 Retry Logic Pattern**

Intelligent retry with exponential backoff:

```typescript
async withRetry<T>(operation: () => Promise<T>): Promise<T> {
  let lastError: Error;
  
  for (let attempt = 1; attempt <= this.config.retryAttempts; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;
      
      // ✅ Check if error is retryable
      if (!this.shouldRetry(error) || attempt === this.config.retryAttempts) {
        throw error;
      }
      
      // ✅ Wait with exponential backoff
      const delay = this.getRetryDelay(attempt);
      await this.delay(delay);
    }
  }
  
  throw lastError!;
}

private shouldRetry(error: unknown): boolean {
  if (error instanceof SmartLeadError) {
    return error.isRetryable || error.isRateLimit || error.isServerError;
  }
  return false;
}

private getRetryDelay(attempt: number): number {
  return Math.min(1000 * Math.pow(2, attempt - 1), 30000); // Max 30s
}
```

## 🔗 **Integration Patterns**

### **🎯 Client Integration**
> **File**: [`src/client/index.ts`](mdc:src/client/index.ts)

Main client aggregates all modules:

```typescript
export class SmartLeadClient extends BaseSmartLeadClient {
  // ✅ Module instances
  public readonly campaigns: CampaignManagementClient;
  public readonly leads: LeadManagementClient;
  public readonly emailAccounts: EmailAccountManagementClient;
  public readonly analytics: AnalyticsClient;
  public readonly statistics: StatisticsClient;
  public readonly smartDelivery: SmartDeliveryClient;
  public readonly smartSenders: SmartSendersClient;
  public readonly webhooks: WebhookManagementClient;
  public readonly clientManagement: ClientManagementClient;
  
  constructor(config: SmartLeadConfig) {
    super(config);
    
    // ✅ Initialize all modules
    this.campaigns = new CampaignManagementClient(this);
    this.leads = new LeadManagementClient(this);
    this.emailAccounts = new EmailAccountManagementClient(this);
    this.analytics = new AnalyticsClient(this);
    this.statistics = new StatisticsClient(this);
    this.smartDelivery = new SmartDeliveryClient(this);
    this.smartSenders = new SmartSendersClient(this);
    this.webhooks = new WebhookManagementClient(this);
    this.clientManagement = new ClientManagementClient(this);
  }
}
```

### **🖥️ Server Integration** 
> **File**: [`src/server.ts`](mdc:src/server.ts)

MCP server registers all tools:

```typescript
// ✅ Register all tool categories
registerCampaignTools(server, client, formatSuccessResponse, handleError);
registerLeadTools(server, client, formatSuccessResponse, handleError);
registerEmailAccountTools(server, client, formatSuccessResponse, handleError);
registerAnalyticsTools(server, client, formatSuccessResponse, handleError);
registerStatisticsTools(server, client, formatSuccessResponse, handleError);
registerSmartDeliveryTools(server, client, formatSuccessResponse, handleError);
registerSmartSendersTools(server, client, formatSuccessResponse, handleError);
registerWebhookTools(server, client, formatSuccessResponse, handleError);
registerClientManagementTools(server, client, formatSuccessResponse, handleError);
```

## 🎯 **Development Best Practices**

### **✅ API Method Guidelines**

1. **📝 Use descriptive method names** that clearly indicate the action
2. **🛡️ Accept typed parameters** using Zod schemas  
3. **📊 Return typed responses** with proper error handling
4. **📚 Include JSDoc comments** for complex methods
5. **⚡ Use async/await** consistently
6. **🔄 Implement retry logic** for API calls
7. **🏷️ Add proper TypeScript types** for all inputs/outputs

### **🛠️ MCP Tool Guidelines**

1. **📋 Provide clear titles and descriptions** for each tool
2. **✅ Use Zod schemas** for input validation
3. **📊 Return consistent response formats** using helper functions
4. **💬 Include helpful error messages** for users
5. **📝 Add summary information** when useful
6. **🎯 Follow naming conventions** consistently
7. **🔍 Test all edge cases** thoroughly

### **🚨 Error Handling Guidelines**

1. **🛡️ Catch and handle all errors** appropriately
2. **🎯 Use SmartLeadError** for API-related errors
3. **💬 Provide user-friendly error messages** in MCP tools
4. **📝 Log errors appropriately** for debugging
5. **🔄 Implement retry logic** for transient failures
6. **📊 Include error context** in responses
7. **🔍 Test error scenarios** thoroughly

## 🚀 **Quick Development Checklist**

### **✅ Adding a New API Method**
- [ ] Define Zod schema in `src/types.ts`
- [ ] Add method to appropriate module client
- [ ] Use proper TypeScript types
- [ ] Include JSDoc documentation
- [ ] Handle errors with SmartLeadError
- [ ] Test with different inputs

### **✅ Adding a New MCP Tool**
- [ ] Follow naming convention (`smartlead_action_resource`)
- [ ] Use existing Zod schema for validation
- [ ] Provide clear title and description
- [ ] Handle errors gracefully
- [ ] Include helpful summary in response
- [ ] Test tool functionality

### **✅ Code Review Checklist**
- [ ] No `any` types used
- [ ] Proper error handling implemented
- [ ] Zod schemas used for validation
- [ ] TypeScript types are specific
- [ ] JSDoc comments for public methods
- [ ] Consistent naming conventions
- [ ] Proper async/await usage

## 🎉 **Ready to Build?**

You now have all the patterns and practices needed to contribute effectively to the SmartLead MCP Server! Remember to check the other Cursor rules for specific tool implementations and TypeScript standards.

**Next Steps:**
- 🛠️ See `mcp-tools.mdc` for complete tool reference
- 📝 Review `typescript-standards.mdc` for coding standards
- 🏗️ Check `project-overview.mdc` for project structure

2. **Use SmartLeadError** for API-related errors
3. **Provide user-friendly error messages** in MCP tools
4. **Log errors appropriately** for debugging
5. **Implement retry logic** for transient failures

---
> Source: [LeadMagic/smartlead-mcp-server](https://github.com/LeadMagic/smartlead-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
