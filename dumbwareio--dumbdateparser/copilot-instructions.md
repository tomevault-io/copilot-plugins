## dumbdateparser

> "formatter": "prettier",

{
  "javascript": {
    "formatter": "prettier",
    "formatterOptions": {
      "printWidth": 100,
      "tabWidth": 2,
      "singleQuote": true,
      "trailingComma": "es5",
      "bracketSpacing": true,
      "semi": true
    },
    "linter": "eslint",
    "linterOptions": {
      "rules": {
        "no-unused-vars": "warn",
        "no-console": "warn",
        "no-undef": "error",
        "prefer-const": "warn"
      }
    }
  },
  "general": {
    "trimTrailingWhitespace": true,
    "insertFinalNewline": true,
    "formatOnSave": true
  },
  "custom": {
    "parallelFiles": [
      {
        "files": ["src/browser.js", "src/index.js"],
        "rules": {
          "maintainParallelFunctionality": true,
          "synchronizeMethods": true,
          "synchronizeConstants": true,
          "allowedDifferences": [
            "export/global scope handling",
            "environment-specific implementations"
          ]
        }
      }
    ]
  }
} 

---
> Source: [DumbWareio/DumbDateParser](https://github.com/DumbWareio/DumbDateParser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
