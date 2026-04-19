## quicktap-backend

> IngestSelector Node implementation and specification guidelines

# IngestSelector Node Guidelines

## Overview
The IngestSelectorNode is a critical component that orchestrates document processing using LLM-powered tool-based search. It uses AI SDK tools to perform intelligent document selection and clarification.

## File Structure
- Main implementation: [libs/agent/src/nodes/ingest-selector/node.ts](mdc:libs/agent/src/nodes/ingest-selector/node.ts)
- Type definitions: [libs/agent/src/nodes/ingest-selector/types.ts](mdc:libs/agent/src/nodes/ingest-selector/types.ts)
- Zod schemas: [libs/agent/src/nodes/ingest-selector/schemas.ts](mdc:libs/agent/src/nodes/ingest-selector/schemas.ts)
- Prompts: [libs/agent/src/nodes/ingest-selector/prompts.ts](mdc:libs/agent/src/nodes/ingest-selector/prompts.ts)

## Core Architecture

### 1. Tool-Based Search Pattern
- Use AI SDK's `tool()` helper with Zod schemas for tool definitions
- Define tools in the node implementation, not in separate service files
- Use `generateObjectWithTools` from LLMService for tool execution

### 2. Prompt Management
- All prompts must be in English (as per user requirements)
- Store prompts in [prompts.ts](mdc:libs/agent/src/nodes/ingest-selector/prompts.ts)
- Use existing `getToolBasedSearchInstructions` for all clarification scenarios
- Never create separate prompt functions for clarification messages

### 3. Schema Design
- Use Zod schemas for AI SDK tool parameters and return types
- Keep schemas in [schemas.ts](mdc:libs/agent/src/nodes/ingest-selector/schemas.ts)
- Provide English descriptions for all schema fields
- Export both TypeScript interfaces and Zod schemas

## Implementation Guidelines

### 1. Error Handling
- Throw exceptions with English messages only
- Avoid using `getLocalizedMessage` or localized strings
- Use descriptive error messages that explain what went wrong
- Log errors with appropriate log levels (debug, warn, error)

### 2. Clarification Logic
- Use LLM to generate clarification messages in user's preferred language
- Leverage existing prompt's clarification strategy
- Support internet search suggestions when `useInternet` is true
- Provide specific, actionable guidance to users

### 3. Search Strategy
- Use vector similarity search via EmbeddingsService
- Implement score-based relevance filtering (score > 0.5 is good)
- Support both `full_ingest` and `chunk_summarize` strategies
- Group results by document and process accordingly

### 4. State Management
- Return partial state updates following BaseNodeService pattern
- Include reasoning logs for debugging and transparency
- Handle both successful processing and clarification scenarios
- Maintain type safety with proper interfaces

## Tool Definition Pattern

```typescript
const searchDocumentsTool = tool({
  description: 'Search for relevant documents and chunks using vector similarity search',
  parameters: ToolSearchParametersSchema,
  execute: async ({ query, limit = 50, scoreThreshold = 0.2, documentIds }) => {
    // Implementation with proper error handling and logging
  }
});
```

## Clarification Strategy

### When No Files Available
- Use mock search tool that returns empty results
- Let LLM analyze context and generate appropriate message
- Consider user notes, available options, and preferred language
- Suggest file upload or internet search alternatives

### When No Relevant Content Found
- Analyze search results and scores
- Generate specific suggestions for query refinement
- Offer alternative search terms or approaches
- Mention internet search if available

## Dependencies
- LLMService: For AI SDK tool calls and text generation
- EmbeddingsService: For vector similarity search
- ConfigurationService: For model selection and settings
- NestJS Logger: For structured logging

## Testing Considerations
- Test both successful document processing and clarification scenarios
- Verify tool-based search functionality
- Ensure proper error handling and fallback messages
- Test with different source selection modes and strategies

## Common Patterns

### 1. Processing Search Results
```typescript
private processSearchResults(
  relevantChunks: SearchResultItem[], 
  ingestStrategy: 'full_ingest' | 'chunk_summarize'
): { processedDocuments: IngestDocument[], processedChunks: IngestChunk[] }
```

### 2. Combining Chunks for Full Ingest
```typescript
private combineChunksContent(chunks: SearchResultItem[]): string
```

### 3. Mapping Clarification Types
```typescript
private mapClarificationType(llmType?: 'no_files' | 'no_results' | 'insufficient_relevance'): 'no_files' | 'no_results' | 'error'
```

## Anti-Patterns to Avoid
- Don't create separate service files for node logic
- Don't use localized error messages
- Don't duplicate prompt logic across multiple functions
- Don't hardcode clarification messages
- Don't use `as any` type casts
- Don't query database directly (use DB services)

## Performance Considerations
- Use appropriate temperature settings (0 for structured output)
- Implement proper logging for debugging
- Handle search timeouts and errors gracefully
- Optimize chunk processing for large document sets

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tundraray) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
