## agent-snapshots

> 当用户说“帮我把项目跑起来”时，默认指的是跑这个 Electron 应用（`pnpm app:dev`）。

改动完之后要把项目启动

当用户说“帮我把项目跑起来”时，默认指的是跑这个 Electron 应用（`pnpm app:dev`）。

启动本地查看器时优先使用 4321 端口；如果 4321 端口已被旧应用占用，先杀掉旧的 4321 应用，再用当前改动后的新应用启动 4321。

官网静态站点默认跑在 4322 端口，本地查看器默认跑在 4321 端口。

当用户只是说“帮我启动项目”“启动应用”等普通启动请求时，成功启动对应项目或应用即可，不需要额外确认真实桌面窗口是否显示、是否位于前台。只有用户明确要求检查窗口或界面状态时，才进行桌面窗口验证。

---
> Source: [ffffhx/agent-snapshots](https://github.com/ffffhx/agent-snapshots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
