## aetheragentsos

> handleSuccess(result);

# Gemini API Integration Guide

This document provides comprehensive documentation for integrating with Google's Gemini API in AetherAgentsOS.

## Table of Contents

- [Overview](#overview)
- [Setup](#setup)
- [Models](#models)
- [Basic Usage](#basic-usage)
- [Streaming](#streaming)
- [Search Grounding](#search-grounding)
- [Thinking Budget](#thinking-budget)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## Overview

AetherAgentsOS uses the `@google/genai` SDK to interact with Google's Gemini AI models. The integration provides:

- **Multi-model support**: Flash, Pro, Lite, Image, Imagen
- **Streaming responses**: Real-time text generation
- **Search grounding**: Web-augmented responses
- **Extended thinking**: Deep reasoning capabilities
- **Automatic fallback**: Graceful model degradation

### SDK Version

```json
"@google/genai": "^1.34.0"
```

---

## Setup

### 1. Get API Key

1. Visit [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Create or sign in to your Google account
3. Generate a new API key
4. Copy the key (starts with `AI...`)

### 2. Configure Environment

```bash
# Create .env file
cp .env.example .env

# Add your API key
echo "GEMINI_API_KEY=your_key_here" >> .env
```

### 3. Verify Configuration

The API key is injected via Vite:

```typescript
// vite.config.ts
define: {
  'process.env.API_KEY': JSON.stringify(env.GEMINI_API_KEY)
}
```

### 4. Test Connection

```typescript
import { testApiConnection } from '@/src/services/gemini';

const isConnected = await testApiConnection();
console.log('API Status:', isConnected ? 'Connected' : 'Failed');
```

---

## Models

### Available Models

| Model | ID | Best For | Thinking | Search |
|-------|----|----|----------|--------|
| **Flash** | `gemini-3-flash-preview` | General use, fast responses | Yes | Yes |
| **Pro** | `gemini-3-pro-preview` | Complex reasoning, analysis | Yes | Yes |
| **Lite** | `gemini-flash-lite-latest` | High volume, efficiency | Yes | No |
| **Image** | `gemini-2.5-flash-image` | Image understanding | No | No |
| **Imagen** | `imagen-4.0-generate-001` | Image generation | No | No |

### Model Selection

```typescript
import { ModelType } from '@/src/types';

// Fast responses
config.model = ModelType.FLASH;

// Complex tasks
config.model = ModelType.PRO;

// High volume
config.model = ModelType.LITE;
```

### Fallback Chain

When a model fails, the system automatically tries alternatives:

```
Flash → Pro → Lite
```

```typescript
// Fallback chain configuration
const TEXT_FALLBACK_CHAIN = [
  ModelType.FLASH,
  ModelType.PRO,
  ModelType.LITE,
];
```

---

## Basic Usage

### Simple Generation

```typescript
import { generateAgentResponse } from '@/src/services/gemini';
import { ModelType } from '@/src/types';

const config = {
  name: 'Assistant',
  systemInstruction: 'You are a helpful assistant.',
  model: ModelType.FLASH,
  temperature: 0.7,
  useSearch: false,
  thinkingBudget: 0,
};

const result = await generateAgentResponse(config, 'What is TypeScript?');

console.log(result.text);       // Response text
console.log(result.latency);    // Response time in ms
console.log(result.modelUsed);  // Model that generated response
```

### With System Instruction

```typescript
const config = {
  ...baseConfig,
  systemInstruction: `
    You are a senior React developer.

    TASK: Help with React 19 code.
    STYLE: Concise, practical, with examples.
    CONSTRAINTS: Use TypeScript and hooks only.
  `,
};

const result = await generateAgentResponse(
  config,
  'How do I create a custom hook for form validation?'
);
```

### Temperature Control

```typescript
// Deterministic (same output)
config.temperature = 0.0;

// Balanced (default)
config.temperature = 0.7;

// Creative
config.temperature = 1.0;

// Highly creative
config.temperature = 1.5;
```

---

## Streaming

### Basic Streaming

```typescript
const result = await generateAgentResponse(
  config,
  'Explain React hooks in detail.',
  (chunk) => {
    // Called with accumulated text on each chunk
    console.log('Current text:', chunk);
    setResponseText(chunk);
  }
);
```

### In React Component

```typescript
const ChatView: React.FC = () => {
  const [response, setResponse] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (prompt: string) => {
    setIsLoading(true);
    setResponse('');

    try {
      await generateAgentResponse(
        agentConfig,
        prompt,
        (chunk) => setResponse(chunk) // Update on each chunk
      );
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div>
      {isLoading && <LoadingIndicator />}
      <div className="response">{response}</div>
    </div>
  );
};
```

### Streaming Behavior

- Callback receives full text so far (not individual chunks)
- Called multiple times as response generates
- Final callback contains complete response
- Use for real-time UI updates

---

## Search Grounding

### Enable Search

```typescript
const config = {
  ...baseConfig,
  useSearch: true,  // Enable grounding
  model: ModelType.FLASH, // Must support search
};
```

### Custom Search Query

```typescript
const config = {
  ...baseConfig,
  useSearch: true,
  searchQuery: 'React 19 new features 2024', // Override search
};
```

### Access Grounding Data

```typescript
const result = await generateAgentResponse(config, prompt);

if (result.grounding && result.grounding.length > 0) {
  result.grounding.forEach((chunk) => {
    console.log('Source:', chunk.web?.title);
    console.log('URL:', chunk.web?.uri);
  });
}
```

### Grounding Chunk Structure

```typescript
interface GroundingChunk {
  web?: {
    uri: string;    // Source URL
    title: string;  // Page title
  };
  maps?: {
    uri: string;
    title: string;
  };
}
```

### Supported Models

| Model | Search Support |
|-------|---------------|
| Flash (preview) | Yes |
| Pro (preview) | Yes |
| Lite | No |
| Image | No |
| Imagen | No |

---

## Thinking Budget

### Enable Extended Thinking

```typescript
const config = {
  ...baseConfig,
  thinkingBudget: 4096, // Tokens for reasoning
};
```

### Budget Levels

| Tokens | Use Case |
|--------|----------|
| 0 | Disabled (default) |
| 1024 | Light reasoning |
| 4096 | Moderate analysis |
| 8192 | Deep analysis |
| 16384 | Complex problems |
| 32768 | Maximum reasoning |

### Best Practices

```typescript
// Math/logic problems
thinkingBudget: 8192

// Code analysis
thinkingBudget: 4096

// Simple queries
thinkingBudget: 0

// Research tasks
thinkingBudget: 16384
```

### Thinking Visualization

In the UI, when thinking is active:
- Amber-colored loading indicator
- "Neural Reasoning Engine Active" message
- Progress bar animation

```typescript
{isThinking && (
  <div className="thinking-indicator">
    <BrainCircuit className="text-amber-500 animate-spin" />
    <span>Neural Reasoning Engine Active</span>
  </div>
)}
```

### Supported Models

- Gemini 3 Flash
- Gemini 3 Pro
- Gemini Flash Lite
- Gemini 2.5 models

---

## Error Handling

### Error Types

| Error | Cause | Handling |
|-------|-------|----------|
| `INVALID_API_KEY` | Bad or missing key | Check .env configuration |
| `QUOTA_EXCEEDED` | Rate limit hit | Implement backoff, upgrade |
| `SAFETY_BLOCKED` | Content filtered | Adjust prompt |
| `MODEL_UNAVAILABLE` | Region restricted | Use fallback model |
| `NETWORK_ERROR` | Connection failed | Retry with backoff |

### Error Diagnosis

```typescript
import { diagnoseError } from '@/src/utils';

try {
  const result = await generateAgentResponse(config, prompt);
} catch (error) {
  const diagnosis = diagnoseError(error);

  console.log('Error:', diagnosis.error);
  console.log('Code:', diagnosis.code);
  console.log('Fix:', diagnosis.remediation);
  console.log('Severity:', diagnosis.severity);
  console.log('Docs:', diagnosis.docs_link);
}
```

### Retry with Backoff

```typescript
import { retryWithBackoff } from '@/src/utils';

const result = await retryWithBackoff(
  () => generateAgentResponse(config, prompt),
  3,    // Max retries
  1000  // Base delay (ms)
);
```

### Non-Retryable Errors

These errors skip the retry chain:

```typescript
// Location restrictions
if (error.message.includes('location')) throw error;

// Safety blocks
if (error.message.includes('Safety')) throw error;

// Permission denied
if (error.message.includes('PERMISSION_DENIED')) throw error;
```

---

## Best Practices

### API Key Security

```typescript
// Good: Environment variable
const apiKey = process.env.API_KEY;

// Bad: Hardcoded
const apiKey = 'AIza...'; // Never do this!
```

### Prompt Engineering

```typescript
// Good: Structured instruction
const systemInstruction = `
You are a code reviewer.

TASK: Review TypeScript code.
OUTPUT: Bullet points with severity.
CONSTRAINTS: Focus on type safety.
`;

// Bad: Vague instruction
const systemInstruction = 'Help with code';
```

### Model Selection

```typescript
// Fast responses
useModel(ModelType.FLASH);

// Complex reasoning
useModel(ModelType.PRO);

// Cost efficiency
useModel(ModelType.LITE);
```

### Error Recovery

```typescript
try {
  const result = await generateAgentResponse(config, prompt);
  handleSuccess(result);
} catch (error) {
  const diagnosis = diagnoseError(error);

  if (diagnosis.severity === 'high') {
    notifyUser(diagnosis.remediation);
  } else {
    logWarning(diagnosis);
  }
}
```

### Performance

```typescript
// Reduce latency
config.thinkingBudget = 0; // Disable if not needed

// Stream for UX
await generateAgentResponse(config, prompt, setResponse);

// Cache responses (if applicable)
const cached = cache.get(promptHash);
if (cached) return cached;
```

---

## Troubleshooting

### API Key Issues

```bash
# Check environment
echo $GEMINI_API_KEY

# Verify in browser console
console.log(process.env.API_KEY?.substring(0, 10));
```

### Rate Limiting

```
Error: 429 Too Many Requests
```

**Solutions:**
1. Implement exponential backoff
2. Reduce request frequency
3. Upgrade API quota

### Safety Blocks

```
Error: Response blocked by safety filters
```

**Solutions:**
1. Adjust prompt wording
2. Enable safety filters in config
3. Use different model

### Region Restrictions

```
Error: Model not available in your region
```

**Solutions:**
1. Use different model
2. Check Google AI availability
3. Use VPN (check ToS)

### Network Errors

```
Error: Failed to fetch
```

**Solutions:**
1. Check internet connection
2. Verify API endpoint
3. Check for CORS issues

### Debug Mode

```typescript
// Log all API calls
console.log('Model:', model);
console.log('Config:', generationConfig);
console.log('Prompt:', effectivePrompt);
```

---

## API Reference

### generateAgentResponse

```typescript
async function generateAgentResponse(
  config: AgentConfig,
  prompt: string,
  onStream?: (chunk: string) => void
): Promise<GenerationResult>
```

### GenerationResult

```typescript
interface GenerationResult {
  text: string;              // Generated text
  grounding?: GroundingChunk[]; // Search sources
  latency: number;           // Response time (ms)
  modelUsed: string;         // Model ID used
}
```

### getGeminiClient

```typescript
function getGeminiClient(): GoogleGenAI
```

### testApiConnection

```typescript
async function testApiConnection(): Promise<boolean>
```

---

## Resources

- [Google AI Studio](https://aistudio.google.com)
- [Gemini API Documentation](https://ai.google.dev/gemini-api/docs)
- [@google/genai SDK](https://www.npmjs.com/package/@google/genai)
- [Gemini Cookbook](https://github.com/google-gemini/cookbook)

---

*Last updated: December 2024*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Krosebrook)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Krosebrook)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
