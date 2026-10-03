## threejs-scene-modules

> 博物馆 Three 展厅（museumScene）dispose 与 composable 协作约定


# 博物馆 Three 展厅规范

## 架构

- 场景宿主：`MuseumScene`（`src/utils/museumScene.ts`）— camera / renderer / scene / controls / loaders / rAF
- 页面生命周期：`useMuseumHall`（PC / 移动共用），在 composable 内用 plain `let` 持有实例
- 热点：`artifactHotspots.ts` + `museumHotspots.ts`
- 资源释放：`src/utils/utils.ts` 的 `disposeMaterial` / `disposeScene` / `disposeTextures`

## 硬性要求

- 传给 Three API 的对象先 `toRaw()`，避免 Vue Proxy
- 卸载 / 路由离开时必须调用场景 `dispose`，移除事件监听与 rAF
- 用户可见错误用 `ElMessage`（中文）
- 改光、环陈半径、漫游位姿等集中在 `museumScene.ts`；文案与分类改 `museumArtifacts.ts`

## 导入

```ts
import * as THREE from 'three';
import { toRaw } from 'vue';
import MuseumScene from '@/utils/museumScene';
```

addons：`three/addons/...`（本仓库现用 WebGLRenderer，非 WebGPU/TSL，除非任务明确要求）。

---
> Source: [zhangbo126/cultural-relics-museum](https://github.com/zhangbo126/cultural-relics-museum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
