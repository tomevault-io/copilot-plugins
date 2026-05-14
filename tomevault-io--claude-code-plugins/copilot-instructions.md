## claude-md

> > How to use the provider and where to find examples

# usage-and-examples

> How to use the provider and where to find examples

## Usage

Add this to your project's CLAUDE.md to activate this skill:

```
Read and follow the instructions in .claude/skills/usage-and-examples/SKILL.md
```

Or copy the instructions below directly into your CLAUDE.md:

# Usage and Examples

Start with the README’s Quick Start and Advanced sections: [README.md](mdc:README.md).

## Examples

- Simple chat completion: [examples/example-simple-chat-completion.ts](mdc:examples/example-simple-chat-completion.ts)
- Tool calling: [examples/example-chat-completion-tool.ts](mdc:examples/example-chat-completion-tool.ts)
- Image recognition: [examples/example-image-recognition.ts](mdc:examples/example-image-recognition.ts)
- Text generation: [examples/example-generate-text.ts](mdc:examples/example-generate-text.ts)

## Common Patterns

- Create provider with service key: `createSAPAIProvider({ serviceKey })`.
- Default instance with env token: `sapai("gpt-4o")`.
- Use with Vercel AI SDK: `generateText`, `streamText`, `generateObject`.

See exported APIs: [src/index.ts](mdc:src/index.ts)

---
> Source: [BITASIA/sap-ai-provider](https://github.com/BITASIA/sap-ai-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:claude_md:2026-05-14 -->

---
> Source: [tomevault-io/claude-code-plugins](https://github.com/tomevault-io/claude-code-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
