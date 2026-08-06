## retro-console-museum

> 纯网页复古游戏机博物馆:Three.js 展厅 + 14 台可玩复古游戏机模拟器。

# AGENTS.md · 维护者指南

纯网页复古游戏机博物馆:Three.js 展厅 + 14 台可玩复古游戏机模拟器。
Vite + 原生 JavaScript,无前端框架。

## 常用命令

```bash
npm run dev        # 开发服务器(已内置跨域隔离头)
npm run build      # 生产构建
npm run preview    # 预览构建产物(冒烟测试默认测它)
npm test           # node --test tests/*.test.mjs
npm run smoke      # puppeteer 冒烟:真加载各核心跑测试 ROM;--dev 改测 dev 服务器
npm run gen:model  # 重新生成 14 台主机的 GLB(scripts/generate-*-model.mjs)
node scripts/screenshot-docs.mjs  # 重拍 README 配图(需先 npm run build)
```

冒烟 / 探针 / 截图脚本依赖本机 Chrome,默认按平台探测安装路径,找不到设
`CHROME_PATH` 环境变量(见 `scripts/chrome-path.mjs`);测试 ROM 放 `.test-roms/`
(已 gitignore,勿随仓库分发)。

## 目录结构速览

- `src/main.js`:总编排——设备切换、ROM 入口、渲染循环。
- `src/config/console-configs.js`:设备数据中心(模型 / 按键 / 镜头 / 展签文案)
  + 模拟器懒加载工厂(首次给 ROM 才 import 对应适配层)。
- `src/render/`:Three.js 场景、相机、渲染器初始化。
- `src/performance/`:RGB565→RGBA、音频重采样等高频路径工具。
- `src/*-emulator.js`:各模拟器核心的适配层,一个文件一台主机。
- `public/emulator/`:各上游预构建核心运行时(`mgba.js` / `mgba.wasm` 在根,
  其余按主机分目录);升级依赖后需手动同步拷贝。
- `public/models/`:14 台主机的 GLB,全部由建模脚本程序化生成,勿手改。
- `scripts/`:建模脚本(generate-*-model.mjs)+ 冒烟 / 探针脚本。
- `tests/`:node:test 单元测试,只覆盖纯逻辑高频路径。

## 代码与注释风格

- 注释用简洁中文,只写"为什么"(约束 + 违反后果),不复述代码在做什么;
  一行能说完不写两行。
- 踩坑得来的技术事实是资产,删除功能时也要保留这些注释。
- 最小改动:不做推测性抽象,不留死代码和无主 TODO。

## 关键约束(改动前先读)

- **跨域隔离(COOP/COEP)**:mGBA 用 SharedArrayBuffer 多线程,页面必须带
  `Cross-Origin-Opener-Policy: same-origin` 和 `Cross-Origin-Embedder-Policy:
  require-corp`。vite dev / preview 已在 `vite.config.js` 配置;静态托管须自行加,
  否则 GB / GBC / GBA 核心无法启动。
- **模拟器鸭子契约**:适配层统一暴露 `init` / `loadRom` / `pause` / `resume` /
  `press` / `unpress` / `screens` / `onFrame`,main.js 与 game-console.js 只认这套
  接口,新增核心必须对齐。
- **画面上屏三形态**:① 核心写 2D canvas,CanvasTexture 逐帧 `needsUpdate`
  (大多数核心);② EJS_Runtime 核心自渲染 WebGL,走 `captureStream` → 离屏 video
  → 2D canvas 镜像(街机 / DC / PSP);③ 未载 ROM 时挂 idle 待机屏(PSP 有专属
  演示屏)。
- **矩阵冻结与阴影按需**:静态节点一律 `matrixAutoUpdate = false`,只有可按压件
  恢复 true;`shadowMap.autoUpdate = false`,展品入场 / 灯光变化时
  `needsUpdate = true` 补拍一次,勿改回逐帧重绘。
- **GLB 节点名 ↔ config 映射链**:console-configs.js 的
  `visual.buttons / dpad / screens` 是逻辑名→GLB 节点名的映射,console3d.js 依此做
  射线检测、按压动画和屏幕纹理绑定;建模脚本里改节点名必须同步改 config,否则
  按键与屏幕静默失效。
- **EJS_Runtime 只能 callMain 一次**:第二次调用画面能加载但主循环已死,街机 /
  PSP 换 ROM 必须销毁旧 runtime、重建新实例。
- **MD 变分辨率重挂纹理**:源 canvas resize 会使已绑定的 GL 纹理失效(MD 核心
  256×192 → 320×224),game-console.js 检测尺寸变化后重新 attachScreen。
- **active 与 controlled 双状态**:active = 当前选中的展品(ROM 入口、直播墙);
  controlled = 最后运行游戏的主机(全局键盘 / 手柄输入路由给它)。二者可以不同,
  读写时勿混。
- **默认漫步视角**:启动即进漫步,点击展品只切换选中、不再飞行聚焦(仅导览
  保留逐台飞行);`M` / 🚶 / Esc 退回轨道视角。手柄 L3/R3 循环切换选中展品。

---
> Source: [mewamew/retro-console-museum](https://github.com/mewamew/retro-console-museum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
