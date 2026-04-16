## blog

> "name": "Next.js 專案結構",

{
  "version": 1,
  "rules": [
    {
      "name": "Next.js 專案結構",
      "description": "確保專案遵循 Next.js 的最佳實踐結構",
      "patterns": [
        "src/app/**/*.{ts,tsx}",
        "src/components/**/*.{ts,tsx}",
        "src/lib/**/*.{ts,tsx}",
        "public/**/*"
      ]
    },
    {
      "name": "部落格文章",
      "description": "管理部落格文章內容",
      "patterns": [
        "posts/**/*.md",
        "posts/**/*.mdx"
      ]
    },
    {
      "name": "配置文件",
      "description": "專案配置文件",
      "patterns": [
        "next.config.ts",
        "tsconfig.json",
        "package.json",
        "postcss.config.mjs",
        "eslint.config.mjs"
      ]
    },
    {
      "name": "組件文件",
      "description": "React 組件文件",
      "patterns": [
        "src/components/**/*.{ts,tsx}",
        "src/app/**/components/**/*.{ts,tsx}"
      ]
    },
    {
      "name": "工具函數",
      "description": "工具函數和輔助方法",
      "patterns": [
        "src/lib/**/*.{ts,tsx}",
        "src/utils/**/*.{ts,tsx}"
      ]
    },
    {
      "name": "類型定義",
      "description": "TypeScript 類型定義文件",
      "patterns": [
        "src/types/**/*.d.ts",
        "**/*.d.ts"
      ]
    },
    {
      "name": "樣式文件",
      "description": "CSS 和樣式相關文件",
      "patterns": [
        "src/**/*.css",
        "src/**/*.scss",
        "src/**/*.module.css"
      ]
    }
  ],
  "ignore": [
    "node_modules/**",
    ".next/**",
    ".git/**",
    "public/**",
    "**/*.lock",
    "**/*.lockb",
    ".DS_Store"
  ]
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/haru5386) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
