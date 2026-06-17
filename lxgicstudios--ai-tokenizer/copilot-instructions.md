## ai-tokenizer

> Count and manage tokens for LLMs. Accurate tokenization for GPT-4, Claude, Llama, and more.

# Token Counter

Count and manage tokens for LLMs. Accurate tokenization for GPT-4, Claude, Llama, and more.

## Quick Start

```bash
npx ai-tokenizer count "Your text here"
```

## What It Does

- Count tokens in text or files
- Truncate text to fit token limits
- Chunk documents into token-sized pieces
- Manage token budgets for prompts
- Compare token counts across models

## Usage

```bash
# Count tokens
npx ai-tokenizer count "Hello world"

# Truncate to 100 tokens
npx ai-tokenizer truncate "long text" --tokens 100

# Chunk a file
npx ai-tokenizer chunk --file doc.txt --tokens 1000

# Analyze text
npx ai-tokenizer analyze --file doc.txt

# Compare models
npx ai-tokenizer compare "text" --models gpt-4,claude-3-sonnet
```

## Part of the LXGIC Dev Toolkit

One of 110+ free developer tools from LXGIC Studios. No paywalls, no sign-ups.

**Find more:**
- GitHub: https://github.com/lxgicstudios
- Twitter: https://x.com/lxgicstudios
- Substack: https://lxgicstudios.substack.com
- Website: https://lxgicstudios.com

## License

MIT. Free forever.

---
> Source: [lxgicstudios/ai-tokenizer](https://github.com/lxgicstudios/ai-tokenizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
