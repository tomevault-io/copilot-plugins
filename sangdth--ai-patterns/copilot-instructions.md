## ui-components

> Implementing designs and building UI components

# UI Components Rule

- Reuse existing UI components from `@/components/ui`, these are the primitives we can build with
- Ask the human how they want to proceed when there are missing components and designs

## Finding Existing Components

- search for the closest `./components/` directory first,
- if you can't find suitable components, try to search further up the directory tree
- up until you find the `@/components/` directory

## Creating New Components

- perform the "Finding Existing Components" steps first
- only create new component if you can't find any existing components that solves the problem
- put the new component in the closest `components` directory that needs it
- only put into `@/components/<new-name>` if it's common enough to be reused in other components

## AI-Related Components

- AI-specific components (chat interfaces, message bubbles, streaming indicators, etc.) are located in `components/ai-elements`
- Check `components/ai-elements` first when building AI/chat-related features
- These components are designed to work with AI streaming, message handling, and conversational interfaces
- Reuse and compose from `components/ai-elements` before creating new AI components

---
> Source: [sangdth/ai-patterns](https://github.com/sangdth/ai-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
