## mcp-tools

> name: SmartLead MCP Tools Reference

---
name: SmartLead MCP Tools Reference
description: Complete reference guide for all 116+ MCP tools organized by category
author: LeadMagic Team
version: 1.0.0
# 🛠️ SmartLead MCP Tools - Complete Implementation Guide

> **Tool Master Reference**: Your comprehensive guide to all 116+ MCP tools in the SmartLead server. Find tools by category, understand implementation patterns, and master the complete SmartLead automation toolkit.

## 🎯 **Quick Tool Finder**

### **🔥 Most Used Tools**
1. **Campaign Management**: `smartlead_create_campaign`, `smartlead_list_campaigns`
2. **Lead Management**: `smartlead_add_leads_to_campaign`, `smartlead_list_leads_by_campaign`
3. **Analytics**: `smartlead_get_analytics_overall_stats_v2`, `smartlead_fetch_campaign_statistics`
4. **Email Accounts**: `smartlead_create_email_account`, `smartlead_fetch_all_email_accounts`

### **📊 Tool Categories Overview**
- 🎯 **Campaign Tools**: 14 tools for campaign management
- 👥 **Lead Tools**: 17 tools for prospect management
- 📧 **Email Account Tools**: 15 tools for account management
- 📊 **Analytics Tools**: 18 tools for reporting
- 📈 **Statistics Tools**: 18 tools for metrics
- 🚀 **Smart Delivery Tools**: 11 tools for deliverability
- 🤖 **Smart Senders Tools**: 12 tools for domain management
- 🔗 **Webhook Tools**: 9 tools for automation
- 👤 **Client Management Tools**: 8 tools for team management

## 🛠️ **Tool Categories Reference**

### **🎯 Campaign Tools** (`src/tools/campaigns.ts`)
> **14 tools** for complete campaign lifecycle management

#### **🚀 Core Campaign Operations**
```typescript
// Create and manage campaigns
smartlead_create_campaign              // Create new email campaigns
smartlead_update_campaign_schedule     // Modify sending schedules
smartlead_get_campaigns_with_analytics // Get campaigns with performance data
smartlead_list_campaigns              // List all campaigns with filtering
smartlead_get_campaign_by_id          // Fetch specific campaign details
smartlead_update_campaign_settings    // Modify campaign configuration
smartlead_delete_campaign             // Remove campaigns
```

#### **⚙️ Campaign Configuration**
```typescript
// Advanced campaign settings
smartlead_pause_campaign              // Pause active campaigns
smartlead_resume_campaign             // Resume paused campaigns
smartlead_clone_campaign              // Duplicate existing campaigns
smartlead_update_campaign_sequences   // Modify email sequences
smartlead_set_campaign_limits         // Configure sending limits
smartlead_update_campaign_tracking    // Enable/disable tracking
smartlead_schedule_campaign           // Set up campaign scheduling
```

**💡 Use Cases:**
- 🎯 **New Campaign Setup**: Use `create_campaign` → `add_leads_to_campaign` → `resume_campaign`
- 📊 **Performance Review**: Use `get_campaigns_with_analytics` for overview
- ⚙️ **Campaign Optimization**: Use `update_campaign_settings` and `update_campaign_sequences`

### **👥 Lead Tools** (`src/tools/leads.ts`)
> **17 tools** for comprehensive prospect and lead management

#### **📥 Lead Import & Management**
```typescript
// Core lead operations
smartlead_add_leads_to_campaign        // Import prospects to campaigns (bulk)
smartlead_list_leads_by_campaign       // Get all prospects in a campaign
smartlead_fetch_lead_by_email          // Find specific prospect by email
smartlead_update_lead_category         // Categorize prospects (interested, not_interested, etc.)
smartlead_remove_lead_from_campaign    // Remove prospects from campaigns
smartlead_get_lead_details             // Fetch detailed prospect information
```

#### **📊 Lead Analytics & Tracking**
```typescript
// Lead performance and tracking
smartlead_get_lead_activity_history    // View prospect interaction timeline
smartlead_fetch_lead_email_history     // Get all emails sent to prospect
smartlead_get_lead_response_data       // Analyze prospect responses
smartlead_track_lead_engagement        // Monitor engagement metrics
smartlead_get_lead_conversion_data     // Track conversion funnel
```

#### **🔄 Lead Automation**
```typescript
// Automated lead management
smartlead_bulk_update_leads            // Update multiple prospects at once
smartlead_auto_categorize_leads        // AI-powered lead categorization
smartlead_schedule_lead_followup       // Set up automated follow-ups
smartlead_sync_leads_with_crm          // CRM integration and sync
smartlead_export_leads_data            // Export prospect data
smartlead_import_leads_from_csv        // Bulk import from CSV files
```

**💡 Use Cases:**
- 📥 **Lead Import**: Use `import_leads_from_csv` → `add_leads_to_campaign`
- 📊 **Performance Analysis**: Use `get_lead_activity_history` and `get_lead_response_data`
- 🔄 **Lead Management**: Use `update_lead_category` and `bulk_update_leads`

### **📧 Email Account Tools** (`src/tools/email-accounts.ts`)
> **15 tools** for email account management and warmup

#### **🔧 Account Setup & Configuration**
```typescript
// Email account management
smartlead_create_email_account         // Add new sending accounts
smartlead_fetch_all_email_accounts     // List all email accounts
smartlead_update_email_account         // Modify account settings
smartlead_delete_email_account         // Remove email accounts
smartlead_test_email_account           // Verify account connectivity
smartlead_get_email_account_details    // Fetch detailed account info
```

#### **🔥 Warmup & Deliverability**
```typescript
// Warmup and reputation management
smartlead_add_update_warmup_to_email_account  // Configure warmup settings
smartlead_get_warmup_status            // Check warmup progress
smartlead_pause_warmup                 // Temporarily pause warmup
smartlead_resume_warmup                // Resume warmup process
smartlead_get_warmup_statistics        // View warmup performance
smartlead_optimize_warmup_settings     // AI-powered warmup optimization
```

#### **📊 Account Analytics**
```typescript
// Performance and monitoring
smartlead_get_account_sending_stats    // View sending statistics
smartlead_fetch_account_deliverability // Check deliverability metrics
smartlead_get_account_reputation_score // Monitor sender reputation
```

**💡 Use Cases:**
- 🔧 **Account Setup**: Use `create_email_account` → `add_update_warmup_to_email_account`
- 📊 **Health Monitoring**: Use `get_warmup_status` and `get_account_reputation_score`
- ⚡ **Performance Optimization**: Use `optimize_warmup_settings`

### **📊 Analytics Tools** (`src/tools/analytics.ts`)
> **18 tools** for comprehensive campaign analytics and reporting

#### **📈 Campaign Analytics**
```typescript
// Campaign performance analysis
smartlead_get_analytics_campaign_list  // List campaigns with analytics
smartlead_get_analytics_overall_stats_v2 // Overall performance statistics
smartlead_get_analytics_day_wise_overall_stats // Daily performance breakdown
smartlead_get_campaign_performance_summary // Campaign-specific metrics
smartlead_get_sequence_performance_analytics // Email sequence analysis
smartlead_get_campaign_conversion_funnel // Conversion tracking
```

#### **👥 Client & Team Analytics**
```typescript
// Team and client insights
smartlead_get_analytics_client_list    // Client analytics overview
smartlead_get_team_performance_analytics // Team productivity metrics
smartlead_get_user_activity_analytics  // Individual user performance
smartlead_get_client_campaign_analytics // Client-specific campaign data
```

#### **📊 Advanced Reporting**
```typescript
// Detailed reporting and insights
smartlead_get_deliverability_analytics // Email deliverability insights
smartlead_get_engagement_analytics     // Open/click/response rates
smartlead_get_time_based_analytics     // Performance over time
smartlead_get_geographic_analytics     // Location-based insights
smartlead_get_device_analytics         // Device and email client data
smartlead_export_analytics_report      // Export comprehensive reports
smartlead_get_real_time_analytics      // Live performance dashboard
smartlead_get_comparative_analytics    // Compare campaigns/periods
```

**💡 Use Cases:**
- 📊 **Performance Overview**: Use `get_analytics_overall_stats_v2`
- 📈 **Campaign Analysis**: Use `get_campaign_performance_summary`
- 📋 **Detailed Reporting**: Use `export_analytics_report`

### **📈 Statistics Tools** (`src/tools/statistics.ts`)
> **18 tools** for detailed statistics and performance metrics

#### **📊 Campaign Statistics**
```typescript
// Campaign-level statistics
smartlead_fetch_campaign_statistics    // Comprehensive campaign stats
smartlead_fetch_campaign_top_level_analytics // High-level campaign metrics
smartlead_get_campaign_sending_stats   // Sending volume and timing
smartlead_get_campaign_response_rates  // Response and engagement rates
smartlead_get_campaign_bounce_analysis // Bounce rate analysis
smartlead_get_campaign_unsubscribe_stats // Unsubscribe tracking
```

#### **🔥 Warmup Statistics**
```typescript
// Email warmup performance
smartlead_fetch_warmup_stats_by_email_account // Account-specific warmup data
smartlead_get_warmup_progress_statistics // Warmup progression tracking
smartlead_get_warmup_engagement_stats  // Warmup engagement metrics
smartlead_get_warmup_reputation_trends // Reputation improvement tracking
```

#### **📈 Performance Metrics**
```typescript
// Advanced performance tracking
smartlead_download_campaign_data       // Export detailed campaign data
smartlead_get_deliverability_statistics // Inbox placement rates
smartlead_get_sender_reputation_stats  // Domain and IP reputation
smartlead_get_time_zone_performance    // Performance by time zone
smartlead_get_subject_line_performance // Subject line A/B testing
smartlead_get_email_client_statistics  // Performance by email client
smartlead_get_seasonal_performance_trends // Performance over time
smartlead_get_competitive_benchmarks   // Industry comparison data
```

**💡 Use Cases:**
- 📊 **Campaign Review**: Use `fetch_campaign_statistics`
- 🔥 **Warmup Monitoring**: Use `fetch_warmup_stats_by_email_account`
- 📈 **Performance Optimization**: Use `get_subject_line_performance`

### **🚀 Smart Delivery Tools** (`src/tools/smart-delivery.ts`)
> **11 tools** for deliverability optimization and spam testing

#### **🧪 Spam Testing**
```typescript
// Deliverability testing
smartlead_create_manual_placement_test  // Manual spam folder testing
smartlead_create_automated_placement_test // Automated spam testing
smartlead_get_spam_test_details         // View test results
smartlead_get_placement_test_history    // Historical test data
smartlead_schedule_recurring_tests      // Automated testing schedule
```

#### **📊 Deliverability Analytics**
```typescript
// Provider and deliverability insights
smartlead_get_provider_wise_report      // Performance by email provider
smartlead_get_deliverability_score      // Overall deliverability rating
smartlead_get_inbox_placement_rates     // Inbox vs spam placement
smartlead_get_domain_reputation_analysis // Domain health check
smartlead_get_ip_reputation_monitoring  // IP address reputation
smartlead_optimize_delivery_settings    // AI-powered optimization
```

**💡 Use Cases:**
- 🧪 **Pre-launch Testing**: Use `create_automated_placement_test`
- 📊 **Performance Monitoring**: Use `get_provider_wise_report`
- ⚡ **Optimization**: Use `optimize_delivery_settings`

### **🤖 Smart Senders Tools** (`src/tools/smart-senders.ts`)
> **12 tools** for domain and sender reputation management

#### **🔍 Domain Management**
```typescript
// Domain operations
smartlead_search_domain                // Domain reputation lookup
smartlead_get_domain_list              // List owned domains
smartlead_add_domain                   // Add new domains
smartlead_verify_domain                // Domain verification
smartlead_get_domain_health_score      // Domain reputation score
smartlead_configure_domain_settings    // Domain configuration
```

#### **📧 Mailbox Management**
```typescript
// Email account generation and management
smartlead_get_vendors                  // Available email service providers
smartlead_auto_generate_mailboxes      // Generate email accounts
smartlead_bulk_create_accounts         // Create multiple accounts
smartlead_configure_mailbox_settings   // Account configuration
smartlead_test_mailbox_connectivity    // Verify account setup
smartlead_optimize_sender_rotation     // Smart sender rotation
```

**💡 Use Cases:**
- 🔍 **Domain Research**: Use `search_domain` and `get_domain_health_score`
- 📧 **Account Setup**: Use `auto_generate_mailboxes`
- ⚡ **Optimization**: Use `optimize_sender_rotation`

### **🔗 Webhook Tools** (`src/tools/webhooks.ts`)
> **9 tools** for webhook management and automation

#### **⚙️ Webhook Configuration**
```typescript
// Webhook management
smartlead_fetch_webhooks_by_campaign   // Get campaign webhooks
smartlead_upsert_campaign_webhook      // Create/update webhooks
smartlead_delete_campaign_webhook      // Remove webhooks
smartlead_test_webhook_endpoint        // Verify webhook connectivity
smartlead_get_webhook_logs             // View webhook delivery logs
```

#### **📊 Webhook Analytics**
```typescript
// Webhook monitoring and analytics
smartlead_get_webhooks_publish_summary // Webhook delivery statistics
smartlead_get_webhook_failure_analysis // Failed delivery analysis
smartlead_retry_failed_webhooks        // Retry failed deliveries
smartlead_configure_webhook_retry_policy // Set retry behavior
```

**💡 Use Cases:**
- ⚙️ **Integration Setup**: Use `upsert_campaign_webhook`
- 📊 **Monitoring**: Use `get_webhooks_publish_summary`
- 🔧 **Troubleshooting**: Use `get_webhook_failure_analysis`

### **👤 Client Management Tools** (`src/tools/client-management.ts`)
> **8 tools** for team and client management

#### **👥 Team Management**
```typescript
// Client and team operations
smartlead_create_client                // Add new team members
smartlead_get_all_clients              // List all clients/users
smartlead_update_client_permissions    // Modify user permissions
smartlead_delete_client                // Remove team members
smartlead_get_team_details             // Team information and metrics
```

#### **🔑 API & Access Management**
```typescript
// API and access control
smartlead_create_client_api_key        // Generate API keys
smartlead_revoke_client_api_key        // Remove API access
smartlead_get_client_usage_statistics  // Monitor API usage
```

**💡 Use Cases:**
- 👥 **Team Setup**: Use `create_client` and `update_client_permissions`
- 🔑 **API Management**: Use `create_client_api_key`
- 📊 **Usage Monitoring**: Use `get_client_usage_statistics`

## 🏗️ **Tool Implementation Patterns**

### **🎯 Standard Tool Structure**
> **Reference**: [`src/tools/index.ts`](mdc:src/tools/index.ts)

All tools follow this registration pattern:

```typescript
// Central export pattern
export { registerAnalyticsTools } from './analytics.js';
export { registerCampaignTools } from './campaigns.js';
export { registerLeadTools } from './leads.js';
export { registerEmailAccountTools } from './email-accounts.js';
export { registerStatisticsTools } from './statistics.js';
export { registerSmartDeliveryTools } from './smart-delivery.js';
export { registerSmartSendersTools } from './smart-senders.js';
export { registerWebhookTools } from './webhooks.js';
export { registerClientManagementTools } from './client-management.js';
```

### **🛠️ Individual Tool Implementation**
> **Example**: [`src/tools/campaigns.ts`](mdc:src/tools/campaigns.ts)

```typescript
export function registerCampaignTools(
  server: McpServer,
  client: SmartLeadClient,
  formatSuccessResponse: FormatSuccessResponseFn,
  handleError: HandleErrorFn
): void {
  
  server.registerTool(
    'smartlead_create_campaign',
    {
      title: 'Create SmartLead Campaign',
      description: 'Create a new email campaign with sequences, settings, and automation rules. Perfect for launching new outreach efforts.',
      inputSchema: CreateCampaignSchema.shape,
    },
    async (params) => {
      try {
        const validated = CreateCampaignSchema.parse(params);
        const result = await client.campaigns.createCampaign(validated);
        
        return formatSuccessResponse(
          'Campaign created successfully',
          result,
          `Created "${validated.name}" with ${validated.sequences?.length || 0} sequences`
        );
      } catch (error) {
        return handleError(error);
      }
    }
  );
}
```

### **📋 Schema Definition Pattern**
> **Reference**: [`src/types.ts`](mdc:src/types.ts)

All input validation uses Zod schemas:

```typescript
const CreateCampaignSchema = z.object({
  name: z.string().min(1, 'Campaign name is required'),
  from_email: z.string().email('Valid email address required'),
  sequences: z.array(z.object({
    subject: z.string().min(1, 'Subject line required'),
    body: z.string().min(1, 'Email body required'),
    delay_days: z.number().min(0).max(365)
  })).min(1, 'At least one sequence required'),
  settings: z.object({
    daily_limit: z.number().min(1).max(1000).optional(),
    timezone: z.string().optional(),
    track_opens: z.boolean().default(true),
    track_clicks: z.boolean().default(true)
  }).optional()
});
```

## 🎯 **Tool Usage Guidelines**

### **🏷️ Tool Naming Standards**
- **Prefix**: Always `smartlead_`
- **Action**: Clear verbs (`get`, `create`, `update`, `delete`, `list`, `fetch`)
- **Resource**: Primary resource being acted upon
- **Specificity**: Add context when needed (`by_campaign`, `with_analytics`)

### **📝 Tool Description Best Practices**
1. **Be specific** about what the tool does
2. **Include use cases** and benefits
3. **Mention key parameters** when helpful
4. **Keep it concise** but informative
5. **Use action-oriented language**

### **✅ Input Schema Best Practices**
1. **Use descriptive field names** matching SmartLead API
2. **Add helpful validation messages** for better UX
3. **Mark optional fields** with `.optional()`
4. **Use appropriate Zod validators** (`.email()`, `.url()`, `.min()`)
5. **Set sensible defaults** where applicable

## 📊 **Tool Categories by Function**

### **🔍 Read Operations** (Safe, No Side Effects)
```typescript
// Query and fetch operations
get_*, list_*, fetch_*, search_*, download_*
```
- Safe to call repeatedly
- No modifications to SmartLead data
- Perfect for dashboards and reporting

### **✏️ Write Operations** (Create/Update)
```typescript
// Modification operations
create_*, add_*, update_*, upsert_*, configure_*
```
- Modify SmartLead data
- Include confirmation in responses
- Validate inputs thoroughly

### **🗑️ Delete Operations** (Destructive)
```typescript
// Removal operations
delete_*, remove_*, revoke_*
```
- Permanently modify data
- Include clear warnings in descriptions
- Consider soft delete when possible

### **📊 Analytics Operations** (Read-only, Computed)
```typescript
// Analytics and reporting
get_analytics_*, fetch_statistics_*, get_*_report
```
- Return computed/aggregated data
- Often support date ranges
- May have larger response sizes

## 🚀 **Advanced Tool Features**

### **📦 Bulk Operations**
Tools that handle multiple items efficiently:
- `smartlead_add_leads_to_campaign` - Import multiple prospects
- `smartlead_bulk_update_leads` - Update multiple prospects
- `smartlead_bulk_create_accounts` - Create multiple email accounts

### **📥 Export/Download Tools**
Tools for data export and reporting:
- `smartlead_download_campaign_data` - Export campaign data
- `smartlead_export_analytics_report` - Export analytics
- `smartlead_export_leads_data` - Export prospect data

### **⏰ Real-time Tools**
Tools providing live data and monitoring:
- `smartlead_get_real_time_analytics` - Live performance metrics
- `smartlead_get_webhook_logs` - Recent webhook activity
- `smartlead_get_warmup_status` - Current warmup progress

## 🎯 **Quality Assurance Checklist**

### **✅ Tool Development Checklist**
- [ ] **Schema validation** works for all required/optional fields
- [ ] **Error handling** provides helpful, actionable messages
- [ ] **Success responses** include meaningful summaries
- [ ] **API integration** correctly calls SmartLead endpoints
- [ ] **Type safety** maintained throughout call chain
- [ ] **Documentation** is clear and comprehensive

### **🚨 Common Issues to Avoid**
1. **Missing input validation** - Always use Zod schemas
2. **Poor error messages** - Provide context and solutions
3. **Inconsistent naming** - Follow established patterns
4. **Missing descriptions** - Every tool needs clear docs
5. **Type casting** - Avoid `any` types, use proper interfaces
6. **Insufficient testing** - Test all edge cases

## 🎉 **Ready to Use the Tools?**

You now have complete knowledge of all 116+ SmartLead MCP tools! Use this guide to find the right tools for your automation needs and implement them effectively.

**Quick Start:**
1. 🔍 **Find your tool** using the category sections above
2. 📋 **Check the schema** requirements in `src/types.ts`
3. 🛠️ **Implement** following the patterns in `api-patterns.mdc`
4. ✅ **Test** thoroughly with different inputs

**Next Steps:**
- 🔧 Check `api-patterns.mdc` for implementation details
- 📝 Review `typescript-standards.mdc` for coding standards
- 🏗️ See `project-overview.mdc` for project structure

2. **Poor error messages** - Provide context and solutions
3. **Inconsistent naming** - Follow the established patterns
4. **Missing descriptions** - Every tool needs clear documentation
5. **Type casting** - Avoid `any` types, use proper interfaces

---
> Source: [LeadMagic/smartlead-mcp-server](https://github.com/LeadMagic/smartlead-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
