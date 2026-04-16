## blockarizedailab-poc

> **CRITICAL**: Full technical port preserving ALL complexity - NO simplifications.

# Windsurf IDE Rules - BlockarizedAILab-POC

## 🎯 Project: Complete Blockarized AI-Lab Technical Showcase

**CRITICAL**: Full technical port preserving ALL complexity - NO simplifications.

---

## Core Rules

### TypeScript
- NEVER use `any` types
- All types explicitly defined
- Files under 500 lines

### PowerShell (Windows)
- Use `;` for chaining (not `&&`)
- Native PowerShell cmdlets

### Git
- Checkpoint before changes: `git add .; git commit -m "checkpoint: before [change]"`

---

## Project Patterns

### Database
```typescript
// Service layers for DynamoDB/OpenSearch
import { getSongById } from '@/lib/db/dynamodb';
import { searchSimilarLines } from '@/lib/search/opensearch';
```

### LLM
```typescript
// Unified multi-provider service
import { callLLM } from '@/lib/llm-service';
```

### Porting
- Copy from `Projects-25/BlockarizedLyrics/app/` verbatim
- Preserve all useState hooks, validations, complexity
- Reference `PRD.md` for specifications

---

## Key Files
- `PRD.md` - Complete spec
- `CLAUDE.md` - Detailed guidance
- `.github/copilot-instructions.md` - Full rules

---

**Framework**: Next.js 15 + TypeScript

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lmnhd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
