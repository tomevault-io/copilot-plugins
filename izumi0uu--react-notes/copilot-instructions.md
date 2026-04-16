## react-notes

> - 在我进行ai提问时, 如果.cursorrules文件生效, 则在每个我对你的提问中开始说明".cursorrules文件生效"

# Check .cursorrules file
- 在我进行ai提问时, 如果.cursorrules文件生效, 则在每个我对你的提问中开始说明".cursorrules文件生效"

## Project structure

```
react-notes/
├── app/                   # Next.js App Router directory
│   ├── [locale]/          # Internationalization routes
│   │   ├── layout.js      # Root layout
│   │   ├── page.js        # Home page
│   │   └── globals.css    # Global styles
│   │   └── page.module.css    
│   │   └── actions.js     # Server actions
│   └── note/              # Note related pages
│       ├── [id]/          # Dynamic route: note detail
│       │   ├── page.jsx
│       │   └── loading.jsx
│       └── edit/          # Edit functionality
│            ├── [id]/
│            │   └── page.jsx
│            └── page.jsx
│            └── loading.jsx
│  
├── components/            # Shared components
│   ├── button/
│   ├── editButton/
│   ├── logo/
│   ├── nextIntlClientProvider/
│   ├── note/
│   ├── noteEditor/
│   ├── notePreview/
│   ├── sidebar/
│   ├── sidebarNoteItemContent/
│   ├── sidebarNoteItemHeader/
│   ├── sidebarNoteList/
│   ├── sidebarNoteListFilter/
│   ├── sidebarNoteListSkeleton/
│   └── sidebarSearchField/
├── i18n/                  # Internationalization configuration
│   ├── request.js        # Request config for next-intl
│   └── routing.js        # Routing config for next-intl
├── lib/                   # Utility functions
│   └── redis.js          # Redis database operations
├── messages/             # Internationalization messages
│   ├── en.json          # English translations
│   └── zh.json          # Chinese translations
├── public/               # Static files
├── .env                  # Environment variables
├── .eslintrc.json       # ESLint configuration
├── .gitignore           # Git ignore rules
├── config.js            # Global configuration
├── middleware.js        # Next.js middleware
├── next.config.mjs      # Next.js configuration
├── package.json         # Project dependencies
└── README.md            # Project documentation
```

## Next.js: Config

- 使用next-intl作为国际化插件
- 使用nextjs 15
- 使用react 19
- 使用react-dom 19
- 使用dayjs 1.11.13
- 使用ioredis 5.4.1
- 使用marked 15.0.3
- 使用sanitize-html 2.13.1
- 使用zod 3.24.1
- 使用next-auth 5.0.0-beta.25

## Next.js: Prioritize Route Handlers

- **Implement all API endpoints using Route Handlers.**
- **Use Route Handlers for complex server operations like database operations, external API calls, and authentication.**
- **Use Server Actions only for simple form submissions or basic data processing.**

## Next.js: Module Import Optimization

- **Use Precise Module Imports:**
  ```javascript
  // ✅ Recommended: Import specific module
  import { specific } from "package/specific/path"
  
  // ❌ Avoid: Import full package
  import { specific } from "package"
  ```

- **Minimize Dependencies in Server Components:**
  - Import only server-compatible modules
  - Avoid modules containing client-side features (hooks, DOM operations)
  - Check module source for client-side dependencies

- **Optimize for Tree-Shaking:**
  - Use direct paths to specific functionality
  - Import from submodules when available
  - Consider checking module source or documentation for optimal import paths

- **Bundle Size Considerations:**
  - Monitor imports in server components for client-side code inclusion
  - Use package documentation to identify optimized import paths
  - Consider splitting client/server functionality when importing packages

## Before AI MODEL using internet, please check the following rules:

- **Check if the related url is fresh.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/izumi0uu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
