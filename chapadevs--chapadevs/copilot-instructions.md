## ai-preview-structure

> AI preview build structure — components, components/ui, pages; multi-file preferred; Sandpack entry index.js → App.js

# AI Preview Build Structure

- **Generated previews** must follow the build structure: `./components`, `./components/ui`, `./pages`.
- **Output format**: Prefer multi-file output (a `files` object with paths like `/App.js`, `/components/Header.js`, `/pages/HomePage.js`, etc.) for new generation. Single `code` string (one `/App.js`) remains supported for backward compatibility.
- **Sandpack**: When `websitePreviewFiles` is present, build Sandpack from the `files` object; otherwise use `websitePreviewCode` for a single `/App.js`. Entry point is always `index.js` → `App.js`.
- **App.js**: In multi-file mode, App.js is the shell only (imports from `./pages/*` and `./components/Header`, `./components/Footer` or `./components/Sidebar`; holds `currentPage` state; nav via `button` + `onClick`).
- **Management Panel template**: When project type is "Management Panel / ERP / CRM" or prompt includes management/panel/erp/crm/admin/dashboard keywords, generated preview uses Sidebar, LoginPage, RegisterPage, DashboardPage, ProductsPage, UsersPage. Extremely simple, beautiful panel UI; frontend-only.

---
> Source: [Chapadevs/Chapadevs](https://github.com/Chapadevs/Chapadevs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
