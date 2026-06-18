## runtime

> 运行与调试

# 运行与调试

- 开发：`yarn start`（LangGraph CLI，端口 2025，配置 [langgraph.json](mdc:langgraph.json)）。
- TypeScript 直跑（如需要）：`nodemon --exec "node --loader ts-node/esm --experimental-specifier-resolution=node" src/bin/www.ts`。
- 生产：`npm run build` → `node dist/...`。
- 无扩展名导入：通过 `tsconfig.json` 的 `moduleResolution: bundler` + Node `--experimental-specifier-resolution=node` 支持。

---
> Source: [iamouyang21/DeepResearch-Langgraph](https://github.com/iamouyang21/DeepResearch-Langgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
