## starkweb

> - When importing types, always use relative imports like `import type { Block } from '../../types/block.js'`

# Cursor AI Rules

## Import Rules
- When importing types, always use relative imports like `import type { Block } from '../../types/block.js'` 
- Never use absolute imports like `import type { Block } from 'src/types/block.js'`

## Code Generation Rules
- When generating or modifying imports for types, ensure paths are relative to the file location
- Convert any instances of `src/types/block.js` to the appropriate relative path (usually `../../types/block.js` in decorators) 

---
> Source: [NethermindEth/starkweb](https://github.com/NethermindEth/starkweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
