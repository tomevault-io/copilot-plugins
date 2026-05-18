## 09-package-management

> - This project uses **Yarn** for frontend package management. Use `yarn add`, `yarn install`, `yarn remove`, etc., instead of `npm`.


### Package Management

- This project uses **Yarn** for frontend package management. Use `yarn add`, `yarn install`, `yarn remove`, etc., instead of `npm`.
- Upon deploying, Databricks Apps detects if `package.json` is present and calls `npm install`, `npm install -r requirements.txt` (if present), and `npm run build` in that case. This should build the static JavaScript content dynamically and there is no need to ship that explicitly.

---
> Source: [databrickslabs/ontos](https://github.com/databrickslabs/ontos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
