## umi

> 1. add a ts file to `packages/preset-umi/src/features/${name}/${name}.ts`, the plugin api doc is in `docs/docs/docs/api/plugin-api.md`.

## How to add a built-in feature plugin?

1. add a ts file to `packages/preset-umi/src/features/${name}/${name}.ts`, the plugin api doc is in `docs/docs/docs/api/plugin-api.md`. 

Sample code:

```ts
import { IApi } from '../../types';
export default (api: IApi) => {
  api.describe({
    key: '404',
  });
};
```

2. Add to `packages/preset-umi/src/index.ts` to the `features` part.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/umijs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
