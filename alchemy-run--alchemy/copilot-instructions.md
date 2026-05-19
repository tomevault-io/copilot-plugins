## cloudflare

> When adding a new resource that can be bound to a worker, make sure to update:

When adding a new resource that can be bound to a worker, make sure to update:

1. bindings.ts - add the binding type to the union
2. bound.ts - map the Alchemy resource to the Cloudflare runtime binding type
3. worker.ts - map the binding to the cloduflare metadata api
4. {resource}.ts - add a new file for the resource alchemy/src/cloudflare/{resource}.ts
5. {resource}.test.ts - add a new file for the resource alchemy/test/cloudflare/{resource}.test.ts

---
> Source: [alchemy-run/alchemy](https://github.com/alchemy-run/alchemy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
