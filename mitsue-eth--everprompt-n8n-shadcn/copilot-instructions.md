## simple-implementation

> Simple, cost-effective implementation approach for EverPrompt


# Simple Implementation Guide

## Core Philosophy

### 1. **Start Simple, Scale Smart**

- No LLM usage for core features
- Pure JavaScript parsing
- Minimal external dependencies
- Predictable costs from day 1

### 2. **n8n Community First**

- JSON workflow parser (no AI needed)
- Prompt extraction and organization
- Community sharing and discovery
- Perfect for YouTube content

## MVP Features (Weeks 1-4)

### **Core Functionality**

- [ ] **Prompt Editor**: Large textarea with autosave
- [ ] **Label System**: Visual arc navigation
- [ ] **n8n JSON Parser**: Extract prompts from workflows
- [ ] **Dark/Light Mode**: Simple theme toggle
- [ ] **Basic Authentication**: Clerk integration

### **No-LLM JSON Parser**

```typescript
// Pure JavaScript - no AI costs
class N8nWorkflowParser {
  parseWorkflow(json: string): ParsedWorkflow {
    const workflow = JSON.parse(json);
    return {
      name: workflow.name,
      nodes: workflow.nodes,
      connections: workflow.connections,
      metadata: {
        nodeCount: workflow.nodes.length,
        connectionCount: Object.keys(workflow.connections).length,
      },
    };
  }

  extractPrompts(workflow: ParsedWorkflow): ExtractedPrompt[] {
    const prompts: ExtractedPrompt[] = [];

    workflow.nodes.forEach((node) => {
      if (this.isAINode(node)) {
        const extracted = this.extractFromNode(node);
        prompts.push(...extracted);
      }
    });

    return prompts;
  }

  private isAINode(node: WorkflowNode): boolean {
    const aiNodeTypes = [
      "n8n-nodes-base.perplexity",
      "@n8n/n8n-nodes-langchain.openAi",
      "n8n-nodes-base.chatGpt",
      "n8n-nodes-base.anthropic",
    ];
    return aiNodeTypes.includes(node.type);
  }

  private extractFromNode(node: WorkflowNode): ExtractedPrompt[] {
    const prompts: ExtractedPrompt[] = [];
    const params = node.parameters;

    // Extract from messages array
    if (params.messages?.message) {
      params.messages.message.forEach((msg: any) => {
        prompts.push({
          nodeId: node.id,
          nodeName: node.name,
          nodeType: node.type,
          promptType: msg.role === "system" ? "system" : "user",
          content: msg.content,
          variables: this.extractVariables(msg.content),
          position: node.position,
        });
      });
    }

    // Extract from values array (LangChain format)
    if (params.messages?.values) {
      params.messages.values.forEach((msg: any) => {
        prompts.push({
          nodeId: node.id,
          nodeName: node.name,
          nodeType: node.type,
          promptType: msg.role === "system" ? "system" : "user",
          content: msg.content,
          variables: this.extractVariables(msg.content),
          position: node.position,
        });
      });
    }

    return prompts;
  }

  private extractVariables(content: string): string[] {
    const variables: string[] = [];

    // Extract n8n expressions like {{ $json.field }}
    const expressionRegex = /\{\{\s*\$([^}]+)\s*\}\}/g;
    let match;

    while ((match = expressionRegex.exec(content)) !== null) {
      variables.push(match[1].trim());
    }

    return [...new Set(variables)]; // Remove duplicates
  }
}
```

## Free Tier Strategy

### **Generous Free Limits**

- **100 prompts** per workspace
- **20 labels** per workspace
- **10 workflow collections**
- **1 workspace** per user
- **10MB storage** (text only)

### **What's Free**

- Core prompt management
- n8n workflow parsing
- Basic label system
- Public prompt sharing
- Community library access

## Paid Tier (€5/month)

### **Unlimited Everything**

- **Unlimited prompts** (10,000+)
- **Unlimited labels** (100+)
- **Unlimited workflows** (100+)
- **30-day money-back guarantee** (no questions asked)
- **Export capabilities**
- **API access**

### **Value Proposition**

- "Support EverPrompt development"
- "Unlock unlimited potential"
- "Help build the n8n community"

## Cost Structure

### **Monthly Infrastructure Costs**

- **Vercel Pro**: €20 (unlimited bandwidth)
- **Neon Database**: €19 (1GB storage)
- **Vercel Blob**: €5 (100GB storage)
- **Error Tracking**: €0 (Vercel Analytics + custom logging)
- **Total**: €44/month

### **Break-Even Point**

- **9 paid users** (€44/month)
- **Target**: 200 paid users (€1,000/month)

## Database Schema (Simple)

### **Core Tables**

```sql
-- Users (Clerk integration)
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Workspaces
CREATE TABLE workspaces (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  user_id UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Labels
CREATE TABLE labels (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  color TEXT,
  workspace_id UUID REFERENCES workspaces(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Prompts
CREATE TABLE prompts (
  id UUID PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  workspace_id UUID REFERENCES workspaces(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Prompt-Label junction
CREATE TABLE prompt_labels (
  prompt_id UUID REFERENCES prompts(id),
  label_id UUID REFERENCES labels(id),
  PRIMARY KEY (prompt_id, label_id)
);

-- Workflow Collections
CREATE TABLE workflow_collections (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL,
  workflow_json JSONB,
  workspace_id UUID REFERENCES workspaces(id),
  created_at TIMESTAMP DEFAULT NOW()
);
```

## Implementation Phases

### **Phase 1: MVP (Weeks 1-4)**

- [ ] Next.js 15 setup with App Router
- [ ] Clerk authentication
- [ ] Neon database setup
- [ ] Basic UI components
- [ ] n8n JSON parser
- [ ] Prompt CRUD operations

### **Phase 2: Monetization (Weeks 5-8)**

- [ ] Stripe payment integration
- [ ] Usage limits and tracking
- [ ] Paid tier features
- [ ] Cost monitoring
- [ ] Backup system

### **Phase 3: Community (Weeks 9-12)**

- [ ] Public prompt library
- [ ] Sharing features
- [ ] Community collections
- [ ] Search and filtering

### **Phase 4: Polish (Weeks 13-16)**

- [ ] Performance optimization
- [ ] Advanced features
- [ ] Mobile optimization
- [ ] Analytics dashboard

## Your YouTube Integration

### **Video Workflow**

1. **Create n8n workflow** (as usual)
2. **Export JSON** (one-click)
3. **Upload to EverPrompt** (extract prompts)
4. **Organize and improve** (better prompts)
5. **Share with community** (build following)

### **Content Ideas**

- **"How to organize your n8n prompts"**
- **"Extract prompts from any n8n workflow"**
- **"Build a prompt library for automation"**
- **"Share prompts with the community"**

## Success Metrics

### **Month 1**

- **Users**: 100
- **Prompts**: 500
- **Revenue**: €0
- **Costs**: €70

### **Month 3**

- **Users**: 500
- **Prompts**: 2,500
- **Revenue**: €250 (50 paid users)
- **Costs**: €70
- **Profit**: €180

### **Month 6**

- **Users**: 1,000
- **Prompts**: 5,000
- **Revenue**: €1,000 (200 paid users)
- **Costs**: €200
- **Profit**: €800

## Risk Mitigation

### **Cost Control**

- **Hard limits**: Never exceed budget
- **Usage alerts**: 80% warning, 95% critical
- **Auto-pause**: Stop service at 100% limits

### **Data Security**

- **Daily backups**: Automated to S3
- **User exports**: Full data portability
- **No vendor lock-in**: Easy migration

### **Technical Risks**

- **Vercel reliability**: 99.9% uptime
- **Database backups**: Multiple strategies
- **Error tracking**: Sentry monitoring

This approach ensures you stay in control of costs while building a valuable service for the n8n community. The no-LLM approach keeps costs predictable and the freemium model provides clear value to users.

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
