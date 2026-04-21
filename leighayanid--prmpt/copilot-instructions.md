## prmpt

> > This file provides context for Claude when working on this codebase.

# CLAUDE.md - AI Prompt Engine

> This file provides context for Claude when working on this codebase.

## Project Overview

**AI Prompt Engine** is a visual interface that transforms user selections into production-ready prompts for AI code generation. Instead of generating code directly, it generates expertly-crafted prompts that users can take to any AI model (Claude, GPT, Gemini) for consistent, high-quality results.

### Core Value Proposition
- **Model-agnostic**: Generated prompts work with any AI
- **Zero API costs**: Runs entirely client-side
- **Reusable**: Save and share prompts as templates
- **Educational**: Users learn prompt engineering

## Tech Stack

| Component | Technology | Purpose |
|-----------|------------|---------|
| Framework | Next.js 14 (App Router) | Server Components, routing |
| UI Library | Shadcn/UI | Toggle groups, cards, copy button, toasts |
| State | React Hook Form + Zod | Form state and validation |
| Styling | Tailwind CSS | Utility-first styling |
| Storage | localStorage | Persist user configs and saved prompts |
| Icons | Lucide React | Consistent iconography |

## Project Structure

```
src/
├── app/
│   ├── page.tsx              # Main prompt builder interface
│   ├── library/
│   │   └── page.tsx          # Saved prompts library
│   └── layout.tsx            # Root layout with providers
├── components/
│   ├── ui/                   # Shadcn components
│   ├── stack-selector.tsx    # Tech stack toggle groups
│   ├── spec-kit-editor.tsx   # Standards/rules editor
│   ├── prompt-preview.tsx    # Generated prompt display
│   ├── prompt-actions.tsx    # Copy, download, share buttons
│   └── model-hints.tsx       # AI model optimization tips
├── lib/
│   ├── prompt-builder.ts     # Core prompt generation logic
│   ├── templates/            # Prompt section templates
│   │   ├── role.ts
│   │   ├── context.ts
│   │   ├── spec-kit.ts
│   │   ├── task.ts
│   │   ├── constraints.ts
│   │   ├── output-format.ts
│   │   └── verification.ts
│   ├── presets.ts            # Pre-configured stack combinations
│   └── storage.ts            # localStorage utilities
├── hooks/
│   ├── use-prompt-config.ts  # Form state management
│   └── use-saved-prompts.ts  # Library CRUD operations
└── types/
    └── prompt-config.ts      # TypeScript interfaces
```

## Code Conventions

### File Naming
- **Files**: `kebab-case` (e.g., `stack-selector.tsx`, `prompt-builder.ts`)
- **Components**: `PascalCase` (e.g., `StackSelector`, `PromptPreview`)
- **Functions**: `camelCase` (e.g., `buildPrompt`, `getStoredConfigs`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_CONFIG`, `STACK_OPTIONS`)

### Component Patterns
```tsx
// Prefer named exports for components
export function StackSelector({ config, onChange }: StackSelectorProps) {
  // Component logic
}

// Colocate types with components when specific to that component
interface StackSelectorProps {
  config: PromptConfig;
  onChange: (config: Partial<PromptConfig>) => void;
}
```

### State Management
- Use React Hook Form for the main configuration form
- Use Zod schemas for runtime validation
- Persist to localStorage on config changes
- No global state library needed (prop drilling is fine for this scale)

## Core Data Types

```typescript
// types/prompt-config.ts

interface PromptConfig {
  // Stack Selections
  frontend: "nextjs" | "vite" | "remix" | "astro";
  backend: "node" | "bun" | "go" | "python";
  database: "drizzle" | "prisma" | "none";
  auth: "clerk" | "authjs" | "supabase" | "none";
  styling: "tailwind" | "css-modules" | "styled-components";
  
  // Project Details
  projectName: string;
  projectType: "saas" | "ecommerce" | "blog" | "api" | "custom";
  description?: string;
  
  // Spec Kit (User's Standards)
  specKit: SpecKit;
  
  // Output Preferences
  outputFormat: "json" | "markdown" | "files-list";
  includeExamples: boolean;
  targetModel: "claude" | "gpt" | "gemini" | "any";
}

interface SpecKit {
  fileNaming: "kebab-case" | "camelCase" | "PascalCase";
  componentNaming: "PascalCase" | "camelCase";
  folderStructure: "feature-based" | "type-based" | "hybrid";
  customRules: string[];
}

interface SavedPrompt {
  id: string;
  name: string;
  config: PromptConfig;
  generatedPrompt: string;
  createdAt: string;
  updatedAt: string;
}
```

## Prompt Architecture

Generated prompts follow this structure:

1. **Role Definition** - Sets the AI's persona
2. **Context Block** - Project type, tech stack, requirements
3. **Standards (Spec Kit)** - Naming conventions, folder structure, rules
4. **Task Specification** - What to generate
5. **Constraints** - What NOT to do, edge cases
6. **Output Format** - Expected response structure (JSON, markdown, etc.)
7. **Examples** - Optional few-shot examples
8. **Verification** - Self-check instructions for the AI

### Prompt Builder Implementation

```typescript
// lib/prompt-builder.ts

export function buildPrompt(config: PromptConfig): string {
  const sections = [
    buildRoleSection(config),
    buildContextSection(config),
    buildSpecKitSection(config),
    buildTaskSection(config),
    buildConstraintsSection(config),
    buildOutputSection(config),
    config.includeExamples ? buildExamplesSection(config) : "",
    buildVerificationSection(config)
  ];
  
  return sections.filter(Boolean).join("\n\n");
}
```

## UI Components Guide

### Stack Selector
- Use Shadcn `ToggleGroup` for each category
- Show compatibility indicators (green dot = works well together)
- Include preset buttons for common stacks (T3, MERN, etc.)

### Spec Kit Editor
- Dropdowns for naming conventions and folder structure
- Dynamic list for custom rules (add/remove)
- Import button for .eslintrc/.prettierrc parsing (future)

### Prompt Preview
- Syntax-highlighted markdown display
- Collapsible sections for each prompt part
- Character/token count display

### Action Buttons
- **Copy**: Use `navigator.clipboard.writeText()` with toast feedback
- **Open in Claude**: Link to `https://claude.ai/new?q={encodedPrompt}`
- **Download**: Generate `.md` file with `URL.createObjectURL()`
- **Share**: Encode config in URL params for sharing

## Model-Specific Optimizations

When `targetModel` is set, add these hints to the generated prompt:

### Claude
- Use XML tags for structure (`<context>`, `<task>`, `<output>`)
- Mention "use extended thinking for complex decisions"
- Prefer explicit JSON schemas in output format

### GPT-4
- Structure as system + user message pair
- Add "Respond only with valid JSON" for JSON output
- Use numbered steps for complex tasks

### Gemini
- Leverage longer context window mentions
- Good for multimodal inputs (mention if relevant)
- Use clear section headers

## localStorage Schema

```typescript
// Keys used in localStorage
const STORAGE_KEYS = {
  CURRENT_CONFIG: 'prompt-engine:current-config',
  SAVED_PROMPTS: 'prompt-engine:saved-prompts',
  SPEC_KIT_PRESETS: 'prompt-engine:spec-kit-presets',
  USER_PREFERENCES: 'prompt-engine:preferences'
};
```

## Common Tasks

### Adding a New Stack Option
1. Update the type in `types/prompt-config.ts`
2. Add UI option in `components/stack-selector.tsx`
3. Add context in `lib/templates/context.ts`
4. Update any compatibility mappings

### Adding a New Prompt Section
1. Create template in `lib/templates/`
2. Import and add to `buildPrompt()` in `lib/prompt-builder.ts`
3. Add toggle in UI if optional

### Adding a Preset Stack
1. Add to `lib/presets.ts`
2. Include in preset selector UI

## Testing Checklist

When modifying prompt generation:
- [ ] Generated prompt is valid markdown
- [ ] All selected options appear in context
- [ ] Spec Kit rules are properly formatted
- [ ] Output format matches selection
- [ ] Copy to clipboard works
- [ ] Saved prompts persist after refresh

## Commands

```bash
# Development
npm run dev

# Build
npm run build

# Lint
npm run lint

# Type check
npm run typecheck
```

## Future Considerations

### Phase 2 Features (Not Yet Implemented)
- Supabase integration for team sharing
- Import from existing project configs
- Prompt version history

### Phase 3 Features (Not Yet Implemented)
- Public prompt gallery
- Upvoting and ratings
- Claude Projects / GPT Custom Instructions integration

## Notes for Claude

When working on this codebase:

1. **Keep it client-side**: No API routes needed for MVP. All logic runs in the browser.

2. **Prompt quality is paramount**: The generated prompts should be expertly crafted. Test outputs with actual AI models.

3. **UX matters**: One-click copy, instant feedback, smooth animations. This is a developer tool—make it delightful.

4. **Extensibility**: New stack options and prompt sections will be added. Keep the architecture modular.

5. **No over-engineering**: localStorage is fine. No need for Redux, Zustand, or complex state management.

6. **Accessibility**: Ensure keyboard navigation works for all interactive elements.

7. **Mobile-friendly**: The interface should work on tablets (developers do use them sometimes).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leighayanid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
