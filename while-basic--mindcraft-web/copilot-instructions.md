## mindcraft-web

> "format_on_save": true,

{
  "line_length": 100,
  "format_on_save": true,
  "javascript": {
    "formatter": "prettier",
    "semi": true,
    "singleQuote": true,
    "trailingComma": "es5"
  },
  "typescript": {
    "formatter": "prettier",
    "semi": true,
    "singleQuote": true,
    "trailingComma": "es5",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitAny": true,
    "exactOptionalPropertyTypes": true
  },
  "tailwind": {
    "sortClasses": true,
    "classRegex": "^[a-zA-Z0-9-]+$"
  },
  "css": {
    "sortProperties": true,
    "indentation": 2
  },
  "markdown": {
    "trailingSpaces": "trim",
    "unorderedListStyle": "-"
  },
  "git": {
    "ignorePatterns": [
      "node_modules",
      ".next",
      "build",
      "*.log",
      ".env*"
    ]
  },
  "imports": {
    "order": [
      "react",
      "next",
      "external-libraries",
      "components",
      "utils",
      "styles"
    ],
    "newlinesBetweenGroups": 1
  },
  "components": {
    "namingConvention": "PascalCase",
    "propsInterface": true,
    "propsNaming": "ComponentNameProps"
  },
  "functions": {
    "namingConvention": "camelCase",
    "maxParameters": 3,
    "maxLines": 50
  },
  "variables": {
    "namingConvention": "camelCase",
    "preferConst": true
  },
  "testing": {
    "required": true,
    "coverage": 80,
    "naming": "*.test.tsx?"
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/while-basic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
