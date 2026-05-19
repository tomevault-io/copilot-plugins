## naming-conventions

> - **Utility Files**: All files in the `utils/` directory must follow `CamelCase.js` naming format

# Naming Conventions

## File Naming

- **Utility Files**: All files in the `utils/` directory must follow `CamelCase.js` naming format
  - Examples: `MetafieldHandler.js`, `ProductImageHandler.js`, `ShopifyIDUtils.js`

- **Strategy Files**: Strategy classes in the `strategies/` directory should follow `CamelCase.js` naming format
  - Examples: `MetaobjectSyncStrategy.js`, `ProductMetafieldSyncStrategy.js`

## Class Naming

Classes should match their filename:
- A file named `MetafieldHandler.js` should export a class called `MetafieldHandler`

## Method Naming

- Use camelCase for method names (e.g., `createMetafield()`, `updateProduct()`)
- Getter methods should begin with "get" (e.g., `getProductById()`)

---
> Source: [kalenjordan/metasync](https://github.com/kalenjordan/metasync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
