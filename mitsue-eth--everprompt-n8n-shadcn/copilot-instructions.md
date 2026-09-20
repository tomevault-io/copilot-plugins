## n8n-workflow-parser

> n8n workflow JSON parser and prompt extraction strategy


# n8n Workflow Parser & Prompt Extraction

## Core Concept

**Simple User Flow**: User completes n8n workflow → uploads JSON → EverPrompt extracts prompts → creates organized collection

## Workflow Parser Architecture

### 1. **JSON Parser Interface**

```typescript
interface N8nWorkflowParser {
  parseWorkflow(json: string): ParsedWorkflow;
  extractPrompts(workflow: ParsedWorkflow): ExtractedPrompt[];
  createCollection(
    prompts: ExtractedPrompt[],
    workflowName: string
  ): PromptCollection;
}

interface ParsedWorkflow {
  name: string;
  nodes: WorkflowNode[];
  connections: WorkflowConnection[];
  metadata: WorkflowMetadata;
}

interface WorkflowNode {
  id: string;
  name: string;
  type: string;
  position: [number, number];
  parameters: Record<string, any>;
  credentials?: Record<string, any>;
}
```

### 2. **Prompt Extraction Logic**

```typescript
interface ExtractedPrompt {
  nodeId: string;
  nodeName: string;
  nodeType: string;
  promptType: "system" | "user" | "template";
  content: string;
  variables: string[];
  position: [number, number];
  metadata: {
    model?: string;
    temperature?: number;
    maxTokens?: number;
    credentials?: string;
  };
}

class N8nPromptExtractor {
  extractFromNode(node: WorkflowNode): ExtractedPrompt[] {
    const prompts: ExtractedPrompt[] = [];

    // Extract from different node types
    switch (node.type) {
      case "n8n-nodes-base.perplexity":
        prompts.push(...this.extractPerplexityPrompts(node));
        break;
      case "@n8n/n8n-nodes-langchain.openAi":
        prompts.push(...this.extractOpenAIPrompts(node));
        break;
      case "n8n-nodes-base.chatGpt":
        prompts.push(...this.extractChatGPTPrompts(node));
        break;
      // Add more node types as needed
    }

    return prompts;
  }
}
```

## Supported Node Types

### 1. **Perplexity Nodes**

```typescript
extractPerplexityPrompts(node: WorkflowNode): ExtractedPrompt[] {
  const prompts: ExtractedPrompt[] = [];
  const params = node.parameters;

  if (params.messages?.message) {
    params.messages.message.forEach((msg: any, index: number) => {
      prompts.push({
        nodeId: node.id,
        nodeName: node.name,
        nodeType: 'perplexity',
        promptType: msg.role === 'system' ? 'system' : 'user',
        content: msg.content,
        variables: this.extractVariables(msg.content),
        position: node.position,
        metadata: {
          model: params.model,
          temperature: params.temperature,
          searchRecency: params.options?.searchRecency
        }
      });
    });
  }

  return prompts;
}
```

### 2. **OpenAI/LangChain Nodes**

```typescript
extractOpenAIPrompts(node: WorkflowNode): ExtractedPrompt[] {
  const prompts: ExtractedPrompt[] = [];
  const params = node.parameters;

  if (params.messages?.values) {
    params.messages.values.forEach((msg: any, index: number) => {
      prompts.push({
        nodeId: node.id,
        nodeName: node.name,
        nodeType: 'openai',
        promptType: msg.role === 'system' ? 'system' : 'user',
        content: msg.content,
        variables: this.extractVariables(msg.content),
        position: node.position,
        metadata: {
          model: params.modelId?.value,
          temperature: params.temperature,
          maxTokens: params.maxTokens
        }
      });
    });
  }

  return prompts;
}
```

### 3. **ChatGPT Nodes**

```typescript
extractChatGPTPrompts(node: WorkflowNode): ExtractedPrompt[] {
  const prompts: ExtractedPrompt[] = [];
  const params = node.parameters;

  if (params.messages?.message) {
    params.messages.message.forEach((msg: any, index: number) => {
      prompts.push({
        nodeId: node.id,
        nodeName: node.name,
        nodeType: 'chatgpt',
        promptType: msg.role === 'system' ? 'system' : 'user',
        content: msg.content,
        variables: this.extractVariables(msg.content),
        position: node.position,
        metadata: {
          model: params.model,
          temperature: params.temperature,
          maxTokens: params.maxTokens
        }
      });
    });
  }

  return prompts;
}
```

## Variable Extraction

### 1. **n8n Expression Parser**

```typescript
extractVariables(content: string): string[] {
  const variables: string[] = [];

  // Extract n8n expressions like {{ $json.field }}
  const expressionRegex = /\{\{\s*\$([^}]+)\s*\}\}/g;
  let match;

  while ((match = expressionRegex.exec(content)) !== null) {
    variables.push(match[1].trim());
  }

  // Extract function calls like =function()
  const functionRegex = /=\s*([a-zA-Z_][a-zA-Z0-9_]*)\s*\(/g;
  while ((match = functionRegex.exec(content)) !== null) {
    variables.push(match[1]);
  }

  return [...new Set(variables)]; // Remove duplicates
}
```

### 2. **Variable Types**

```typescript
interface N8nVariable {
  name: string;
  type: "json" | "function" | "constant";
  path?: string; // For JSON variables like $json.field
  functionName?: string; // For function calls
  description?: string;
}
```

## Collection Creation

### 1. **Workflow Collection**

```typescript
interface WorkflowCollection {
  id: string;
  name: string;
  description: string;
  workflowId: string;
  workflowName: string;
  prompts: ExtractedPrompt[];
  metadata: {
    nodeCount: number;
    connectionCount: number;
    createdAt: Date;
    tags: string[];
  };
}
```

### 2. **Collection Builder**

```typescript
class WorkflowCollectionBuilder {
  createCollection(
    workflow: ParsedWorkflow,
    prompts: ExtractedPrompt[]
  ): WorkflowCollection {
    return {
      id: generateId(),
      name: `${workflow.name} - Prompts`,
      description: `Extracted prompts from ${workflow.name} workflow`,
      workflowId: workflow.id,
      workflowName: workflow.name,
      prompts: prompts,
      metadata: {
        nodeCount: workflow.nodes.length,
        connectionCount: Object.keys(workflow.connections).length,
        createdAt: new Date(),
        tags: this.generateTags(workflow, prompts),
      },
    };
  }

  private generateTags(
    workflow: ParsedWorkflow,
    prompts: ExtractedPrompt[]
  ): string[] {
    const tags = new Set<string>();

    // Add node type tags
    prompts.forEach((prompt) => {
      tags.add(prompt.nodeType);
    });

    // Add workflow complexity tags
    if (workflow.nodes.length > 10) tags.add("complex");
    if (workflow.nodes.length < 5) tags.add("simple");

    // Add AI model tags
    prompts.forEach((prompt) => {
      if (prompt.metadata.model) {
        tags.add(prompt.metadata.model);
      }
    });

    return Array.from(tags);
  }
}
```

## UI Components

### 1. **Workflow Upload Component**

```typescript
interface WorkflowUploadProps {
  onWorkflowParsed: (collection: WorkflowCollection) => void;
  onError: (error: string) => void;
}

function WorkflowUpload({ onWorkflowParsed, onError }: WorkflowUploadProps) {
  const handleFileUpload = async (file: File) => {
    try {
      const json = await file.text();
      const parser = new N8nWorkflowParser();
      const workflow = parser.parseWorkflow(json);
      const prompts = parser.extractPrompts(workflow);
      const collection = parser.createCollection(prompts, workflow.name);

      onWorkflowParsed(collection);
    } catch (error) {
      onError("Failed to parse workflow JSON");
    }
  };

  return (
    <div className="workflow-upload">
      <input
        type="file"
        accept=".json"
        onChange={(e) =>
          e.target.files?.[0] && handleFileUpload(e.target.files[0])
        }
      />
      <p>Upload your n8n workflow JSON to extract prompts</p>
    </div>
  );
}
```

### 2. **Workflow Collection View**

```typescript
interface WorkflowCollectionViewProps {
  collection: WorkflowCollection;
  onPromptSelect: (prompt: ExtractedPrompt) => void;
}

function WorkflowCollectionView({
  collection,
  onPromptSelect,
}: WorkflowCollectionViewProps) {
  return (
    <div className="workflow-collection">
      <header>
        <h2>{collection.name}</h2>
        <p>{collection.description}</p>
        <div className="metadata">
          <span>{collection.metadata.nodeCount} nodes</span>
          <span>{collection.prompts.length} prompts</span>
        </div>
      </header>

      <div className="prompts-grid">
        {collection.prompts.map((prompt) => (
          <PromptCard
            key={prompt.nodeId}
            prompt={prompt}
            onClick={() => onPromptSelect(prompt)}
          />
        ))}
      </div>
    </div>
  );
}
```

## Implementation Phases

### Phase 1: Basic Parser (Week 1-2)

- [ ] JSON workflow parser
- [ ] Basic prompt extraction for Perplexity nodes
- [ ] Simple collection creation
- [ ] File upload component

### Phase 2: Extended Support (Week 3-4)

- [ ] OpenAI/LangChain node support
- [ ] ChatGPT node support
- [ ] Variable extraction
- [ ] Collection metadata

### Phase 3: Advanced Features (Week 5-6)

- [ ] More node types (Claude, Anthropic, etc.)
- [ ] Prompt categorization
- [ ] Workflow visualization
- [ ] Export back to n8n

### Phase 4: Community Features (Week 7-8)

- [ ] Public workflow collections
- [ ] Workflow sharing
- [ ] Community templates
- [ ] Workflow marketplace

## Benefits of This Approach

### 1. **Simplicity**

- No complex API integrations
- No authentication with n8n instances
- Works with any n8n workflow

### 2. **Immediate Value**

- Users see all their prompts organized instantly
- No setup required
- Works offline

### 3. **Future Reusability**

- Prompts become reusable across workflows
- Easy to modify and improve prompts
- Version control for prompt evolution

### 4. **Community Building**

- Users can share workflow collections
- Learn from others' prompt strategies
- Build a library of proven workflows

## Example Workflow Analysis

Based on your JSON example:

**Workflow**: "Perplexity Powered AI News Search"
**Extracted Prompts**:

1. **Perplexity System Prompt**: "REDACTED" (instructions for AI behavior)
2. **Perplexity User Prompt**: "Find and summarize the most recent..." (search query)
3. **OpenAI System Prompt**: "You're a helpful formatter Agent" (formatting instructions)
4. **OpenAI User Prompt**: "Here is today's AI news list..." (formatting task)

**Collection Created**: "Perplexity AI News Workflow"

- 4 linked prompts
- Tags: [perplexity, openai, news, automation]
- Metadata: 11 nodes, 6 connections, complex workflow

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
