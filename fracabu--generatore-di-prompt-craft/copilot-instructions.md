## generatore-di-prompt-craft

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a React TypeScript web application for generating structured prompts using the C.R.A.F.T. framework (Context, Role, Action, Format, Target). The application integrates with Google Gemini API and OpenAI to generate high-quality, structured prompts from simple user topics.

## Development Commands

### Essential Commands
- `npm install` - Install dependencies
- `npm run dev` - Start development server (runs on http://localhost:3003)
- `npm run build` - Build for production
- `npm run preview` - Preview production build

### Configuration
- Environment variables are stored in `.env.local` (not committed)
- Required: `VITE_GEMINI_API_KEY` - Google Gemini API key
- API keys are also stored in browser localStorage by users at runtime:
  - `gemini_api_key` - User's Gemini API key
  - `openai_api_key` - User's OpenAI API key
  - `openrouter_api_key` - User's OpenRouter API key
  - `openrouter_model` - Selected OpenRouter model (e.g., 'openai/gpt-4o')

## Architecture Overview

### Core Framework: C.R.A.F.T.
The application is built around the C.R.A.F.T. prompt engineering framework:
- **C**ontesto (Context): Background and situation
- **R**uolo (Role): The expert persona the AI should adopt
- **A**zione (Action): Specific task to perform
- **F**ormato (Format): Output structure
- **T**arget: Target audience

### Application Flow
1. **HomeView** (`/`) - User enters a topic
2. **AI Service Layer** - Transforms topic into structured C.R.A.F.T. prompt using AI
3. **ResultView** (`/result`) - Displays generated C.R.A.F.T. prompt
4. **TestView** (`/test`) - Tests the generated prompt and shows AI response

### Data Flow & State Management
- **No Redux/Context**: Application uses React Router navigation with localStorage for state persistence
- **Navigation Pattern**: Data flows between views via localStorage:
  - `currentTopic` - The user's input topic
  - `currentCraftPrompt` - Generated C.R.A.F.T. prompt (JSON)
  - `selectedProvider` - AI provider ('gemini' | 'openai' | 'openrouter')
  - `testResult` - Test output from AI
  - `savedCraftPrompts` - Array of saved prompts
- **Session Cleanup**: `App.tsx` clears transient localStorage data on mount while preserving API keys (`gemini_api_key`, `openai_api_key`, `openrouter_api_key`) and saved prompts

### Service Architecture

#### AI Service Layer (`services/aiService.ts`)
Primary service that supports multiple AI providers:
- **Triple Provider Support**: Gemini (primary), OpenAI, and OpenRouter
- **OpenRouter Support**: Allows access to 30+ models from various providers (OpenAI, Anthropic, Google, Meta, Mistral, Qwen, DeepSeek, etc.)
- **Structured Output**: Uses JSON schema validation for consistent C.R.A.F.T. structure
- **Key Functions**:
  - `generateCraftPrompt(topic, provider)` - Generates C.R.A.F.T. prompt from topic
  - `testGeneratedPrompt(prompt, provider)` - Tests prompt and returns AI response with truncation detection
- **System Instruction**: Contains comprehensive meta-prompt that teaches AI the C.R.A.F.T. framework with detailed examples
- **Response Format Fallback**: OpenRouter tries `response_format: { type: "json_object" }` first, falls back to plain text if unsupported

#### Gemini Service (`services/geminiService.ts`)
Legacy Gemini-only service (kept for backward compatibility):
- Same functionality as aiService but Gemini-only
- Uses `@google/genai` SDK
- Configuration: `gemini-2.5-flash` model with temperature 0.7 for generation, 0.8 for testing
- Thinking budget: 1024 tokens for complex reasoning

### Component Architecture

#### Views (Main Pages)
- **HomeView**: Entry point with topic input, auto-rotating suggestion carousel (3-second intervals), tutorial modal
- **ResultView**: Displays generated C.R.A.F.T. prompt with save functionality
- **TestView**: Shows test results with markdown rendering

#### Reusable Components
- **Header**: Navigation with API settings modal
- **Footer**: Site footer
- **PromptCard**: Displays individual C.R.A.F.T. prompt sections
- **Button3D**: 3D pushable button with CSS transforms
- **TutorialModal**: Interactive C.R.A.F.T. framework explanation
- **WarningModal**: Error/warning dialog
- **ApiSettings**: Modal for configuring API keys
- **ExamplePromptCard**: Rotating suggestion cards

### Styling System
- **Framework**: Tailwind CSS (loaded via CDN in index.html)
- **Color Scheme**:
  - Primary: Sky blue (`sky-600`)
  - Secondary: Emerald (`emerald-400`)
  - Background: Slate dark (`slate-900`, `slate-800`)
- **3D Effects**: Custom CSS transforms with `transformStyle: 'preserve-3d'`
- **Custom Scrollbars**: Styled in index.html with slate theme
- **Responsive**: Mobile-first design with sm/md/lg breakpoints

### Data Structure

#### TypeScript Types (`types.ts`)
```typescript
CraftPrompt {
  contexto: string
  ruolo: string
  azione: string
  formato: string
  target: string
}

SavedPrompt {
  id: string
  topic: string
  prompt: CraftPrompt
  createdAt: string
}
```

#### Static Data
- **examplePrompts.ts**: 6 pre-built example prompts covering social media, blog posts, technical explanations, video scripts, sales emails, and fiction writing

### Path Alias Configuration
- `@/*` maps to project root (configured in vite.config.ts and tsconfig.json)
- Use for cleaner imports: `import { CraftPrompt } from '@/types'`

## Important Implementation Details

### API Integration
- **Dynamic Instance Creation**: AI instances are created per-request using localStorage keys
- **Error Handling**: Both services throw descriptive errors for missing keys or network failures
- **Browser-Only**: Uses browser localStorage and importmap (CDN imports) - not Node.js compatible

### LocalStorage Usage Patterns
- Keys are checked/updated across components using storage events
- App.tsx listens for `storage` and custom `localStorageChange` events
- Cleanup on app initialization preserves only API keys and saved prompts

### 3D Button Implementation
- Uses CSS `transform: translateY()` for depth effect
- Shadow and front layers animated separately
- Mouse events (enter/leave/down/up) control transform values
- Disabled state maintains visual consistency

### Textarea Auto-resize
- HomeView uses ref-based height calculation
- Limits: min 40px, max 128px
- Updates on topic change via useEffect
- Enter key triggers generation (Shift+Enter for newline)

### Carousel Animation
- 3 cards visible at once
- Rotates every 3 seconds through 6 example prompts
- Icon mapping based on topic keywords (social/blog/video/email)
- Click to populate topic input

## Common Development Patterns

### Adding a New Example Prompt
Add to `data/examplePrompts.ts` with structure:
```typescript
{
  id: 'unique-id',
  topic: 'Display title',
  createdAt: new Date().toISOString(),
  prompt: {
    contexto: '...',
    ruolo: '...',
    azione: '...',
    formato: '...',
    target: '...'
  }
}
```

### Adding a New AI Provider
1. Update `aiService.ts` `getAIInstance()` to support new provider
2. Add provider-specific schema format
3. Implement generation and test logic in try/catch blocks
4. Add localStorage key for API configuration
5. Update ApiSettings component for UI

### Navigation Between Views
Always update localStorage before `navigate()`:
```typescript
localStorage.setItem('currentTopic', topic);
localStorage.setItem('currentCraftPrompt', JSON.stringify(prompt));
navigate('/result');
```

### Error Display Pattern
Use WarningModal with error state:
```typescript
const [error, setError] = useState<string | null>(null);
// In JSX:
{error && <WarningModal title="Attenzione" onClose={() => setError(null)}><p>{error}</p></WarningModal>}
```

## Technology Stack Details

- **React 19.2.0** with TypeScript
- **Vite 6.2.0** as build tool and dev server
- **React Router 7.9.4** for client-side routing
- **@google/genai 1.24.0** for Gemini API
- **openai 6.3.0** for OpenAI API
- **Tailwind CSS** (CDN) for styling
- **Browser ES Modules** via importmap (no bundler for deps in production)

## Build & Deployment Notes

- Vite builds to `dist/` folder
- Server configuration: port 3003, host 0.0.0.0
- Environment variables injected via `define` in vite.config.ts
- Production uses importmap for React/Gemini dependencies from CDN
- Ensure `.env.local` is in .gitignore (contains sensitive API keys)

## OpenRouter Integration Details

### Model Selection
OpenRouter provides access to 30+ models from various providers. The application includes a comprehensive dropdown selector in HomeView with models organized by provider:
- **OpenAI**: GPT-5, GPT-5 Mini, GPT-4o, GPT-4o Mini, GPT-4 Turbo, GPT-4, GPT-3.5 Turbo
- **Anthropic**: Claude Sonnet 4.5, Claude 3.5 Sonnet, Claude 3 Opus/Sonnet/Haiku
- **Google**: Gemini Pro 1.5, Gemini Flash 1.5, Gemini 2.0 Flash (Experimental)
- **Meta Llama**: Llama 3.3 70B, Llama 3.1 405B/70B/8B Instruct
- **Mistral**: Mistral Large/Small, Pixtral Large (Multimodal)
- **Qwen**: Qwen 2.5 72B, QwQ 32B (Reasoning), Qwen3 VL 8B (Vision)
- **DeepSeek**: DeepSeek Chat, DeepSeek R1 (Reasoning)
- **Zhipu AI**: GLM 4.6, GLM 4.5
- **Others**: Perplexity Sonar (Online), Grok Beta (xAI), Cohere Command R+

### Configuration
- Selected model is persisted in `localStorage.getItem('openrouter_model')`
- Default model: `openai/gpt-4o`
- Base URL: `https://openrouter.ai/api/v1`
- Custom headers include `HTTP-Referer` and `X-Title` for tracking

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fracabu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fracabu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
