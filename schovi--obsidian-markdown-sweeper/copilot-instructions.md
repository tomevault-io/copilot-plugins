## obsidian-markdown-sweeper

> Obsidian plugin for cleaning up messy markdown. Shows diff preview before applying changes.

# Sweeper

Obsidian plugin for cleaning up messy markdown. Shows diff preview before applying changes.

## Quick Reference

| Command | Description |
|---------|-------------|
| `npm run dev` | Build in watch mode |
| `npm run build` | Type-check + production build |
| `npm run test` | Run all tests |
| `npm run test:watch` | Tests in watch mode |
| `npx vitest run src/rules/[name].test.ts` | Single test file |

## Architecture

### Entry Points

- `src/main.ts` - Plugin class, commands, ribbon icon
- `src/CleanupModal.ts` - Diff preview UI using `diff` library
- `src/settings.ts` - Settings tab + persistence
- `src/presets.ts` - Preset tier logic (minimal/standard/aggressive)

### Rules System (`src/rules/`)

Each rule file exports a `RuleDefinition`:

```typescript
interface RuleDefinition {
  id: string;           // camelCase identifier
  name: string;         // Human-readable name
  group: RuleGroup;     // Category for UI grouping
  tier: PresetTier;     // minimal | standard | aggressive
  example: string;      // Before → After example
  fn: (content: string) => RuleResult;
}

interface RuleResult {
  content: string;      // Transformed content
  changesCount: number; // Number of modifications made
}
```

**Key files:**

- `types.ts` - RuleDefinition, RuleResult, RuleGroup, PresetTier
- `registry.ts` - Ordered list of all rules (execution order matters)
- `index.ts` - `applyAllRules()` orchestration

### Adding a New Rule

1. Create `src/rules/yourRule.ts`:

```typescript
import { RuleDefinition, RuleResult } from "./types";

function yourRuleLogic(content: string): RuleResult {
  let changesCount = 0;
  const result = content.replace(/pattern/g, () => {
    changesCount++;
    return "replacement";
  });
  return { content: result, changesCount };
}

export const yourRule: RuleDefinition = {
  id: "yourRule",
  name: "Your Rule Name",
  group: "formatting",
  tier: "standard",
  example: "before → after",
  fn: yourRuleLogic,
};
```

2. Create `src/rules/yourRule.test.ts`:

```typescript
import { describe, it, expect } from "vitest";
import { yourRule } from "./yourRule";

describe("yourRule", () => {
  it("transforms X to Y", () => {
    const result = yourRule.fn("input");
    expect(result.content).toBe("expected");
    expect(result.changesCount).toBe(1);
  });

  it("handles no changes", () => {
    const result = yourRule.fn("unchanged");
    expect(result.changesCount).toBe(0);
  });
});
```

3. Register in `src/rules/registry.ts`:

```typescript
import { yourRule } from "./yourRule";
// Add to rules array in appropriate position
```

4. Update `README.md` rules table with the new rule

**When modifying or removing rules:** Always update tests and README.md to match.

### Rule Groups

- `blankLines` - Empty line handling
- `whitespace` - Spaces, tabs, indentation
- `lists` - List markers, checkboxes, numbering
- `formatting` - Quotes, emphasis, links, code fences
- `headings` - Heading syntax
- `html` - HTML tags and entities
- `blockElements` - Blockquotes
- `obsidian` - Obsidian-specific (tags)

### Common Patterns

**Protecting code blocks:**

When a rule shouldn't modify content inside code blocks, use `processOutsideCode`:

```typescript
function processOutsideCode(content: string, processor: (text: string) => string): string {
  const parts: string[] = [];
  const codePattern = /(```[\s\S]*?```|`[^`\n]+`)/g;
  // splits content, processes non-code parts only
}
```

Used by: emphasis, htmlEntities, htmlTags, htmlCleanupAggressive, tagNormalization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/schovi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/schovi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
