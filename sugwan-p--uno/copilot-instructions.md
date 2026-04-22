## uno

> FE_Rules

{
  "rules": {
    "no-magic-numbers": ["error", { "ignore": [-1, 0, 1], "detectObjects": true }],
    "camelcase": ["error", { "properties": "always" }],
    "function-name-format": ["error", { "format": "camelCase", "prefix": ["get", "set", "handle", "fetch"] }],
    "variable-name-format": ["error", { "format": "camelCase" }],
    "boolean-prefix": ["error", { "prefix": ["is", "has"] }],
    "no-abbreviations": "error",
    "constant-name-format": ["error", { "format": "UPPER_CASE" }],
    "array-suffix": ["error", { "suffix": "List" }],
    "api-function-suffix": ["error", { "suffix": "Api" }],
    "component-file-naming": ["error", { "pattern": "src/**/index.tsx" }],
    "page-folder-structure": ["error", { "pattern": "src/pages/**" }],
    "require-jsdoc": [
      "warn",
      {
        "require": {
          "FunctionDeclaration": true,
          "MethodDefinition": false,
          "ClassDeclaration": true,
          "ArrowFunctionExpression": false,
          "FunctionExpression": false
        }
      }
    ],
    "arrow-function-use": ["error", "always"],
    "early-return": "error",
    "max-parameters": ["error", 3],
    "async-await-only": ["error"],
    "promise-use": ["error", { "allowInAsyncFunctions": false }],
    "axios-style": "error",

    "quotes": ["error", "single"],

    "prettier-settings": {
      "singleQuote": true,
      "semi": false,
      "tabWidth": 2,
      "trailingComma": "none",
      "bracketSpacing": true,
      "jsxSingleQuote": false
    }
  },
  "commitRules": {
    "types": [
      "feat",
      "fix",
      "docs",
      "style",
      "refactor",
      "test",
      "chore"
    ],
    "max-length": 100,
    "subject-case": "lower",
    "body-required": false,
    "footer-required": false
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sugwan-p) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
