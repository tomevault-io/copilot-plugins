## soulwriter

> SoulWriter 是 Mac-native、local-first 的叙事写作工作台。当前主产品为

# SoulWriter 公开快照工程规则

## 项目定位

SoulWriter 是 Mac-native、local-first 的叙事写作工作台。当前主产品为
`apps/mac/SoulWriterMac`；`apps/api`、`apps/web` 与 `infra/docker` 是遗留
Web-first retained surface，不是当前主线。

## 系统契约

`docs/reports/SCRIV-SYSTEM-CONTRACT-1.md` 是唯一系统级真源。本快照内的
设计、实现、测试与 review 均以该契约为最高优先级；与其他说明冲突时，
以该契约为准。

## 事实层与 Agent 边界

- 项目事实通过 `ProjectRepository` / `LocalProjectStore` 写入；视图与
  Agent 不得直接改写 SQLite。
- `BinderNode` 管结构，`TextDocument` 管正文，`DocumentSynopsis` 与
  `InspectorMetadata` 是投影和元数据，不是第二套事实层。
- Agent 只能提出可审查、可接受或拒绝的 `AgentProposal`；应用前必须创建
  `ProjectSnapshot`，不得默认直接改写正文或项目事实。

## 修改后验证

```bash
git diff --check
xcodebuild test -project apps/mac/SoulWriterMac/SoulWriterMac.xcodeproj \
  -scheme SoulWriterMac -destination 'platform=macOS'
bash scripts/mac/verify_no_server_dependency.sh
```

## 公开快照红线

- 禁止提交凭据、用户项目、本机路径、二进制产物或运行证据。
- 许可证见 `LICENSE`：All Rights Reserved，非开源；未经版权所有者书面
  许可，不得复制、修改、分发或商用。

---
> Source: [lynch5mo/SoulWriter](https://github.com/lynch5mo/SoulWriter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
