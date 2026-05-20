## vercel-ai-sdk

> Reference guide for the Vercel AI SDK, including links to documentation, core concepts, API reference, and usage examples for integrating AI models like Gemini.

# Vercel AI SDK Documentation & Usage

This rule provides quick access to documentation and key information about the Vercel AI SDK, used for integrating AI models into applications.

## Key Documentation Links

-   **Main SDK Docs:** [https://sdk.vercel.ai/docs](mdc:https:/sdk.vercel.ai/docs)
-   **Core Concepts:** [https://sdk.vercel.ai/docs/concepts](mdc:https:/sdk.vercel.ai/docs/concepts)
-   **API Reference:** [https://sdk.vercel.ai/docs/api-reference](mdc:https:/sdk.vercel.ai/docs/api-reference)
-   **Supported Models/Providers (including Google Gemini):** [https://sdk.vercel.ai/docs/providers](mdc:https:/sdk.vercel.ai/docs/providers)
-   **Next.js Integration:** [https://sdk.vercel.ai/docs/integrations/nextjs](mdc:https:/sdk.vercel.ai/docs/integrations/nextjs)

## Core Functions/Hooks

-   `streamText`: For streaming text responses from an API route.
-   `generateText`: For non-streaming text generation.
-   `streamObject`: For streaming structured JSON objects.
-   `generateObject`: For non-streaming structured object generation.
-   `useChat` (React Hook): For building chat interfaces.
-   `useCompletion` (React Hook): For text completion interfaces.

## Provider Integration

The SDK uses provider-specific packages (e.g., `@ai-sdk/google`, `@ai-sdk/openai`) or compatible layers (`createOpenAICompatible`) to interact with different LLMs. Remember to install the necessary provider package (e.g., `bun add @ai-sdk/google` for Gemini). 

---
> Source: [RayFernando1337/vibestamps](https://github.com/RayFernando1337/vibestamps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
