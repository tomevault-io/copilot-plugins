## velvet

> 1. **API Route Pattern**: Always use `app/api/chat/route.ts` for AI logic.

# .cursorrules

## API Route Pattern
1. **API Route Pattern**: Always use `app/api/chat/route.ts` for AI logic.

## OpenRouter Requirement
2. **OpenRouter Requirement**: 
   - NEVER pass the raw `messages` array directly to `streamText`.
   - ALWAYS map over `messages` and flatten `content` arrays into strings. 
   - OpenRouter models throw 400 errors if they receive content arrays.
   - Handle both `parts` format (from @ai-sdk/react) and `content` format.

## Tool Calling
3. **Tool Calling**: 
   - Use `tools` inside `streamText` for capabilities like Image Generation.
   - For Fal.ai image generation, use the REST API directly (https://fal.run/fal-ai/z-image/turbo) with FAL_KEY.
   - Always wrap tool execution in try-catch blocks.

## Imports
4. **Imports**:
   - Use `import { streamText } from 'ai'` (Vercel AI SDK 3.0+).
   - Do NOT import `StreamingTextResponse` (deprecated).
   - Use `return result.toDataStreamResponse()` at the end of the route.
   - For Fal.ai, use the REST API directly with fetch() and FAL_KEY environment variable.

## Message Sanitization
5. **Message Sanitization**:
   - Always sanitize messages before passing to `streamText`.
   - Handle both `parts` (array) and `content` (string or array) formats.
   - Filter out non-text parts when flattening arrays.
   - Ensure final content is always a string, never an array.

---
> Source: [Zeramental/velvet](https://github.com/Zeramental/velvet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
