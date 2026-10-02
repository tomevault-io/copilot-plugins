## godb

> 本文件用于约束 AI Agent（含 Claude Code、Trae 等）在本仓库内的工作方式。所有 Agent 在执行任何任务前必须先读取本文件并严格遵守。

# AGENTS.md

本文件用于约束 AI Agent（含 Claude Code、Trae 等）在本仓库内的工作方式。所有 Agent 在执行任何任务前必须先读取本文件并严格遵守。

---

## 1. 提交规范（Commit）

### 1.1 小步提交
- 每个最小可验证的小需求（或一个独立的功能切片）提交一次 commit。
- 不要把多个不相关的小需求塞进同一个 commit。
- 单次 commit 应能独立编译通过、独立回滚，不破坏既有功能。

### 1.2 只 commit，不 push
- Agent 只能执行 `git add` + `git commit`，**严禁**执行 `git push`、`git push --force`、`git push --force-with-lease`。
- 推送由用户人工完成。Agent 在完成任务后可提示用户"已提交到本地，待你确认后推送"。

### 1.3 用户名与邮箱
- 提交时必须使用本仓库 `.git/config` 中已配置的 `user.name` 与 `user.email`：
  - `user.name  = mmm`
  - `user.email = mm@m.m`
- **禁止**通过 `git config` 修改仓库或全局的用户名/邮箱配置。
- 如果 `git commit` 默认已读取到上述配置，则无需额外传参；否则显式传参：
  ```bash
  git -c user.name="mmm" -c user.email="mm@m.m" commit -m "..."
  ```

### 1.4 Commit Message 规范
- 中文描述，简洁明了，首行不超过 50 字。
- 格式：`<类型>: <简述>`，类型包括：`feat`（新功能）、`fix`（修复）、`refactor`（重构）、`test`（测试）、`docs`（文档）、`chore`（杂项）、`style`（格式）。
- 示例：`feat: 新增消息服务器回执CMD处理`
- 较复杂的改动在正文（空一行后）补充说明动机与关键改动点，使用 HEREDOC 形式提交：
  ```bash
  git commit -m "$(cat <<'EOF'
  feat: 新增消息送达回执上报

  - 接收方收到消息后调用 message/delivered 接口上报
  - 发送方收到 messageDelivered CMD 后更新 push_status
  EOF
  )"
  ```

### 1.5 暂存范围
- 优先按文件精确 `git add <file1> <file2>`，避免 `git add .` / `git add -A`。
- 严禁提交 `.env`、密钥、token、本地配置等敏感文件。
- 严禁提交构建产物（`build/`、`dist/`、`*.apk`、`*.ipa` 等），除非用户明确要求。

---

## 2. 工作流程：先计划，TDD 测试先行

### 2.1 先计划后编码
- 接到需求后，**先输出实现计划**，包括：
  1. 需求拆解（分成哪些小需求 / 小切片）
  2. 每个切片的改动范围（涉及哪些端：后端 / Android / iOS / 桌面 / 文档）
  3. 每个切片的验收标准
  4. 依赖关系与执行顺序
- 计划须用 `TodoWrite` 工具落成 todo 列表，逐项推进，完成一项立即标记 completed。

### 2.2 TDD 测试先行
- 每个功能切片必须遵循 **Red-Green-Refactor**：
  1. **Red**：先写测试（单元测试 / 集成测试），运行确认失败。
  2. **Green**：写最小实现让测试通过。
  3. **Refactor**：在测试保护下重构。
- 测试覆盖范围：
  - 后端 Go：使用项目已有的测试框架（如 `testing` / `testify`），`go test ./...` 必须通过。
  - Android：JUnit / Espresso，`./gradlew test` 或 `./gradlew connectedCheck`。
  - iOS：XCTest，`xcodebuild test`。
  - 桌面端 TS：Jest / Vitest 等，`npm test` 或对应脚本。
- 纯 UI 调整、纯配置类改动若无法写单元测试，须在 commit message 中说明原因并给出人工验证步骤。

### 2.3 测试位置
- 后端：与被测包同目录的 `_test.go` 文件。
- Android：`src/test/java/...`（单元测试）或 `src/androidTest/java/...`（仪器测试）。
- iOS：`XCTest` target，文件名 `XxxTest.m` / `XxxTest.swift`。
- 桌面端：与源码同名的 `*.test.ts` / `*.spec.ts`，放在同级或 `__tests__` 目录。

---

## 3. 代码规范

### 3.1 注释（尽量每行加注释）
- **尽量为每一行代码加注释**，特别是：
  - 业务逻辑分支（`if` / `switch` / 循环）
  - 外部 API 调用、网络请求、数据库操作
  - 异步处理、回调、闭包
  - 魔法数字、阈值、正则表达式
- 注释语言：**中文**（与用户沟通语言一致）。
- 注释风格示例：

  Go：
  ```go
  // 查询该频道内 message_seq 大于 offset 的未推送消息
  messages, err := d.queryMessagesAfterSeq(channelID, channelType, offset)
  if err != nil { // 查询失败则返回错误
      return nil, err
  }
  ```

  Java：
  ```java
  // 通过 clientMsgNo 查询本地消息
  WKMsg msg = WKIM.getInstance().getMsgManager().getWithClientMsgNO(clientMsgNo);
  if (msg == null) { // 本地不存在则跳过
      return;
  }
  ```

  Objective-C：
  ```objc
  // 通过 clientMsgNo 查询消息
  WKMessage *message = [[WKMessageDB shared] getMessageWithClientMsgNo:clientMsgNo];
  if (!message) { // 消息不存在则跳过
      return;
  }
  ```

  TypeScript：
  ```typescript
  // 接收方收到消息后上报送达回执
  if (!message.header.noPersist && message.fromUID !== WKApp.loginInfo.uid) {
    APIClient.shared.post("message/delivered", { ... }); // 调用送达回执接口
  }
  ```

### 3.2 语言一致性
- 代码注释、commit message、计划文档均使用**中文**。
- 变量、函数、类名使用英文，遵循各端命名规范。

### 3.3 最小改动
- 只改与当前需求相关的代码，不做无关重构。
- 不主动新增文档文件（`*.md`），除非用户明确要求（本 AGENTS.md 即为用户明确要求）。
- 不主动添加未被要求的功能、配置项、容错分支。

---

## 4. 安全与禁区

- **禁止**执行 `git push` 及任何形式的强推。
- **禁止**执行破坏性 git 操作：`reset --hard`、`checkout .`、`clean -fd`、`branch -D`、`rebase -i`，除非用户明确要求。
- **禁止**修改 `.git/config`。
- **禁止**提交密钥、token、`.env` 等敏感信息。
- **禁止**在未读文件的情况下直接编辑（先 Read 再 Edit）。
- 长时间运行命令（构建、测试）使用非阻塞方式执行，避免阻塞会话。

---

## 5. 多端协同

本仓库为多端工程，根目录下各端代码位置：

| 端 | 目录 | 主要语言 |
| --- | --- | --- |
| 后端 API | `win-chat-api/` | Go |
| IM Server | `win-chat-server/` | Go |
| Android | `win-chat-android/` | Java |
| iOS | `win-chat-ios/` | Objective-C |
| 桌面端 | `win-chat-webpc/` | TypeScript / React |
| 管理后台 | `win_chat_webadmin/` | - |
| 小程序/Uniapp | `chat-uniapp/` | - |

- 跨端需求须在计划阶段明确列出每端改动点，逐端实现、逐端编译验证。
- 每端编译通过后再进入下一端；若某端编译失败，就地修复，不跳过。
- 各端编译命令：
  - 后端：`cd win-chat-api && go build ./...`（及 `go test ./...`）
  - Android：`cd win-chat-android && ./gradlew assembleDebug`
  - iOS：`cd win-chat-ios && xcodebuild -workspace TangSengDaoDaoiOS.xcworkspace -scheme TangSengDaoDaoiOS -sdk iphonesimulator -configuration Debug build`
  - 桌面端：`cd win-chat-webpc && npm run build`

---

## 6. 执行检查清单

每次完成任务提交前，Agent 须自检：

- [ ] 计划已输出并落成 todo
- [ ] 测试先行（Red → Green → Refactor）
- [ ] 每行关键代码已加中文注释
- [ ] 相关端编译通过
- [ ] 相关端测试通过
- [ ] 按文件精确 `git add`
- [ ] commit 使用仓库默认用户名邮箱（mmm / mm@m.m）
- [ ] 仅 commit 未 push
- [ ] commit message 符合规范

---
> Source: [stylesmile/godb](https://github.com/stylesmile/godb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
