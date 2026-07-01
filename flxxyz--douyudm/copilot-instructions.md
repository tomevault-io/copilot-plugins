## douyudm

> 斗鱼直播弹幕实时获取库，TypeScript + WebSocket，同时支持 Node.js / 浏览器 / CLI。

# douyudm

斗鱼直播弹幕实时获取库，TypeScript + WebSocket，同时支持 Node.js / 浏览器 / CLI。

详见 `README.md`。本文档只记录在 README 之外、未来 Claude 协作时需要预先了解的非显然信息。

## 命令

```bash
pnpm install                  # 项目使用 pnpm，不是 npm（CI 也是 pnpm）
pnpm run build                # Rollup 4，4 个输出（CJS / ESM / 浏览器 IIFE / CLI bin）
pnpm test                     # Jest，全部测试
pnpm run test:coverage        # 必须 100% 覆盖 src/core/stt.ts 和 src/core/packet.ts，否则 CI 失败
pnpm run test:stt             # 单跑 STT
pnpm run test:packet          # 单跑 Packet
```

## 架构（src/ 五层）

| 目录 | 职责 | 是否含环境依赖 |
|------|------|------|
| `src/core/` | STT 文本协议、Packet 二进制协议、常量 | 否（纯逻辑） |
| `src/types/` | 所有 TS 类型/接口 | 否 |
| `src/events/` | 38 个已知消息事件的默认处理器 | 否 |
| `src/platform/{node,browser}.ts` | Rollup 入口，简单 re-export | 否 |
| `src/cli/cmd.ts` | CLI 入口（commander v14） | Node-only |
| `src/index.ts` | `Client` 主类，构造函数接收 `WebSocketFactory` 注入 | 否（通过 factory 隔离） |

`Client` 通过 `defaultWsFactory` 在运行时检测环境：Node 用 `require('ws')`，浏览器用原生 `WebSocket`。

## Gotchas（必读）

### 1. 不要用 `typeof WebSocket !== 'undefined'` 判断浏览器
Node v21+ 内置 WebSocket（undici），会误判。必须用 `process.versions?.node`。
**原因**：Node 内置 WebSocket 的 `event.data` 是 `Blob` 而不是 `Buffer`，会让 `Packet.decode` 崩溃。详见 `src/index.ts:34-44`。

### 2. 浏览器原生 WS 的 `onmessage` 返回 Blob
需要异步 `.arrayBuffer()` 转换后再传给 `_messageHandle`。详见 `src/index.ts:121-128`。

### 3. CLI 的 shebang 由 Rollup banner 注入
`src/cli/cmd.ts` **不要写** `#!/usr/bin/env node`，否则双 shebang 导致语法错误。banner 在 `rollup.config.ts:66`。

### 4. 礼物事件三选一：用 `gdp`，不要用 `dgb`
- `gdp` = 本房间礼物事件 "xx 赠送了 X xN" — **展示用这个**
- `dgb` = 原始事件，默认处理器为空，不建议直接用
- `spbc` = 全站大额礼物广播，**不限本房间**，含 `nn`（目标主播昵称）

浏览器示例 `examples/browser/index.html` 订阅的就是 `gdp` + `spbc`。

### 5. STT 协议有重复 key 的服务端 quirk
`qausrespond` 和 `rri` 消息含两个 `rid` 字段。JS 对象语义下后者覆盖前者，**round-trip 无法还原**。这是斗鱼服务端协议设计问题，不是 STT 实现 bug。`__tests__/stt.test.ts` 显式断言此行为。

### 6. `close()` 必须显式调用
仅置空 `_ws` 引用不会关闭连接。`close()` 内部会发送 `logout`、清理心跳、关闭 socket。

### 7. WS 端口随机选
`run()` 每次从 `[8501..8506]` 随机选一个，URL 是 `wss://danmuproxy.douyu.com:<port>/`。可传自定义 url 覆盖。

### 8. Jest 配置用 `.js` 不是 `.ts`
`jest.config.js` 故意写成 JS，避免引入 `ts-node` 依赖。

## 协议要点

- **Packet header**：12 字节（`uint32 len` ×2 小端 + `int16 typeCode=689` + `int16 占位=0`）。body 末尾追加 `\0`。
- **STT 转义**：`@` → `@A`，`/` → `@S`。`@A` 必须先于 `@S` 转义/反转义，否则双重处理。
- **STT 嵌套**：value 含 `@=` 时递归反序列化为对象；多条消息用 `//` 分隔为数组。

## 测试 fixtures

`__tests__/stt.test.ts` 顶部的 `fixtures` 是真实抓包数据，新增协议字段时优先在这里加用例，确保 round-trip。

## 发布流程

```bash
git tag v3.x.x
git push origin v3.x.x       # 触发 .github/workflows/publish.yml
```

CI 会用 `jq` 把版本号写回 `package.json` 并提交回 master，**本地 master 需 `git pull --rebase origin master` 后才能继续推**。

注意点：
- npm token 类型必须是 **Automation**，否则 `EOTP` 失败
- 用 `jq` 而不是 `npm version`，避免版本号未变时 `Version not changed` 错误
- `examples/browser/` 通过 `static.yml` 自动部署到 GitHub Pages

## 依赖与外部约束

- 运行时：`ws` (Node) + `commander` (CLI)。两者都是 Rollup 的 `external`，不打进库。
- 浏览器构建：`platform/browser.ts` → IIFE，全局变量名 **`douyudm`（小写）**。
- TS target ES2019，lib `["ES2019","DOM"]`，strict mode。
- `dist/` 和 `coverage/` 在 `.gitignore`。
- `data/*.json` 是历史礼物 ID 表和镜像地址，当前代码未使用，仅作参考。

## 与已有 memory 的关系

- `.claude/memory/project_overview.md` — 部分过时（v2.1.1/JS/lowdb 描述），需以代码和本文件为准
- `.claude/memory/refactor_ts_rollup.md` — v3.0.0 重构方案，仍准确，可作架构参考

---
> Source: [flxxyz/douyudm](https://github.com/flxxyz/douyudm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
