## react-next-eslint-plugin

> When working on ESLint rules in [lib/rules/](mdc:lib/rules/), follow these established patterns from the codebase.


# ESLint Rule Development Patterns

When working on ESLint rules in [lib/rules/](mdc:lib/rules/), follow these established patterns from the codebase.

## 🏛️ Rule Structure Template

Every rule follows this standard ESLint rule format:

```javascript
export default {
  meta: {
    type: 'problem', // 'problem', 'suggestion', or 'layout'
    docs: {
      description: 'Brief description of what the rule does',
      category: 'Possible Errors', // or appropriate category
      recommended: true, // boolean
      url: 'https://github.com/laststance/react-next-eslint-plugin/blob/main/docs/rules/rule-name.md',
    },
    fixable: null, // 'code', 'whitespace', or null
    hasSuggestions: false, // boolean
    schema: [], // JSON Schema for rule options
    messages: {
      messageId: 'Error message with {{placeholder}} support',
    },
  },

  create(context) {
    // Rule implementation logic
    return {
      // AST node visitors
    }
  },
}
```

## 🔍 Common AST Node Types for React

Based on existing rules like [no-jsx-without-return.js](mdc:lib/rules/no-jsx-without-return.js):

- **JSX Elements**: `JSXElement`, `JSXFragment`
- **React Hooks**: Look for `CallExpression` with specific callee names
- **Component Functions**: `FunctionDeclaration`, `ArrowFunctionExpression`, `FunctionExpression`
- **Context Providers**: `JSXElement` with name matching `*.Provider`

## 🚨 Error Reporting Best Practices

1. **Use descriptive message IDs** with placeholders for dynamic content
2. **Track reported nodes** to avoid duplicate reports (see Set usage in existing rules)
3. **Provide context-specific messages** for different scenarios
4. **Point to the exact problematic node** in context.report()

## 🧪 Testing Requirements

Each rule must have a corresponding test file in [tests/lib/rules/](mdc:tests/lib/rules/) with:
- Valid code examples that should NOT trigger the rule
- Invalid code examples that SHOULD trigger the rule  
- Proper error message and location testing

## 🎯 Rule Focus Areas

This plugin specializes in:
- **React Performance**: memo, useCallback, stable references
- **Next.js Compatibility**: Server Components, proper API usage
- **Code Quality**: proper returns, semantic naming, best practices
- **Anti-patterns**: common mistakes that lead to bugs or performance issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laststance)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/laststance)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
