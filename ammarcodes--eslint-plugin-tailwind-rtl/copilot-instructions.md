## eslint-plugin-tailwind-rtl

> npm test          # Run all tests (node --test, built-in runner)

# AGENTS.md

## Commands

```bash
npm test          # Run all tests (node --test, built-in runner)
npm run lint      # Lint repo with own plugin
node --test tests/rules/tailwind/no-physical-classes.test.js  # Run single test
```

No build step. Source is published directly.

## Architecture

ESLint plugin (`eslint-plugin-tailwind-rtl`) that catches physical direction classes/properties and suggests logical alternatives.

```
src/
  index.js          # Entry: exports { meta, rules, configs }
  index.d.ts        # TypeScript declarations (keep in sync with index.js)
  maps/
    tailwind.js     # Physical -> logical Tailwind class Map
    css-in-js.js    # Physical -> logical CSS property object
  rules/
    tailwind/       # Tailwind class rule
    css-in-js/      # CSS-in-JS property rule
tests/
  rules/            # Mirrors src/rules/ structure
```

## Adding a New Rule

1. Create `src/rules/{category}/{rule-name}.js` with `{ meta, create(context) }` shape
2. Register in `src/index.js` under `rules` and add to configs if recommended
3. Add mapping data to `src/maps/` if needed
4. Create `tests/rules/{category}/{rule-name}.test.js` using RuleTester
5. Update `src/index.d.ts` with new rule type
6. Update README.md with new rule documentation

## Rule Structure

```js
module.exports = {
  meta: {
    type: "suggestion",
    docs: { description: "...", recommended: true },
    fixable: "code",
    messages: { ruleId: "message with {{placeholder}}" },
    schema: [],
  },
  create(context) {
    return {
      /* AST visitors */
    };
  },
};
```

## Test Pattern

Uses `node:test` + ESLint `RuleTester` (NOT Jest/Mocha):

```js
const { describe, it } = require("node:test");
const assert = require("node:assert");
const { RuleTester } = require("eslint");

describe("rule-name", () => {
  it("should work", () => {
    const ruleTester = new RuleTester();
    ruleTester.run("rule-name", rule, {
      valid: ["valid code"],
      invalid: [
        { code: "bad", output: "fixed", errors: [{ messageId: "ruleId" }] },
      ],
    });
  });
});
```

## Key Conventions

- **CommonJS only** -- `require`/`module.exports` throughout
- **Plugin namespace**: `"tailwind-rtl"`, rule keys use slashes (`"tailwind/no-physical-classes"`)
- **Full rule name in configs**: `"tailwind-rtl/tailwind/no-physical-classes"`
- **Self-linting**: eslint.config.js loads own plugin, enables tailwind rule at "error"
- **eslint.config.js ignores**: `node_modules/**`, `src/maps/**`, `tests/**`
- **Node engine**: >= 22.12.0

## CI/Release

- **CI** (PR + push to main): test + lint + commitlint (PR only)
- **Release**: semantic-release on push to main, npm publish with provenance
- **Commit format**: Conventional Commits (commitlint enforces)

## Update Checklist

When making changes that affect the plugin's public API or rules:

- [ ] Update `src/index.d.ts` if types change
- [ ] Update `README.md` if rules/configs change
- [ ] Update `SKILL.md` if one exists and relevant

---
> Source: [AmmarCodes/eslint-plugin-tailwind-rtl](https://github.com/AmmarCodes/eslint-plugin-tailwind-rtl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
