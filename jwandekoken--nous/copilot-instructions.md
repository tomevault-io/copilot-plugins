## node-pnpm

> Use this rule for all node.js package management operations


# Node Package Manager: pnpm

All Node.js package management operations **must** use `pnpm` instead of `npm` or `yarn`.

## ✅ Always use `pnpm`

- Never use `npm install`, `npm run`, or other npm commands directly
- Never use `yarn add`, `yarn install`, or other yarn commands
- Always prefix commands with `pnpm`

## 🔁 Correct usage

```bash
# Instead of: npm install
pnpm install

# Instead of: npm run build
pnpm build

# Instead of: npm run dev
pnpm dev

# Instead of: yarn add vue-router
pnpm add vue-router
```

## ❌ Incorrect usage

```bash
# Don't do this:
npm install
yarn add lodash
npm run build
```

This ensures consistency across the project and leverages pnpm's superior performance and disk efficiency.

---
> Source: [jwandekoken/nous](https://github.com/jwandekoken/nous) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
