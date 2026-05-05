## koishi-plugin-opencode

> > 📝 **查看变更日志**: [English](./CHANGELOG.md) | [简体中文](./CHANGELOG.zh-CN.md)

# Version Management

> 📝 **查看变更日志**: [English](./CHANGELOG.md) | [简体中文](./CHANGELOG.zh-CN.md)

## 代码修改建议

> [!IMPORTANT]
> 在修改消息发送或处理逻辑时，必须注意代码中的**多分支**情况。例如在 `opencode` 插件中：
> - **流式输出**：存在 Native 模式（编辑消息）和 Segment 模式（分段发送）的分支。
> - **非流式输出**：存在指令直接返回、超时兜底发送、工具调用/Fallback 路径等。
> 类似的改动必须同步应用到所有相关分支路径，确保功能行为的一致性。

## Koishi Element 资源处理规范

> [!IMPORTANT]
> 在处理本地资源（图片、音频、视频、文件）分发时，必须遵循以下标准以确保跨平台兼容性：
>
> 1. **存储路径**：资源应统一复制到 `data/opencode/tmp/` 目录。
> 2. **URL 格式**：使用标准的 `file:///` 绝对路径（RFC 8089）。
>    - **推荐写法**：`decodeURI(pathToFileURL(absPath).href)`，确保路径符合规范且解析时显示原始字符。
> 3. **多逻辑分支覆盖**：消息发送逻辑的修改必须同步覆盖以下路径：
>    - **流式输出**：`Native`（编辑消息）与 `Segment`（分段发送）分支。
>    - **非流式输出**：Action 直接返回、超时兜底发送、工具调用及 Fallback 路径。
> 4. **大多数适配器都不支持将普通消息编辑为文件发送消息**：请避免进行此类修改。

## 更新插件版本
> [!IMPORTANT]
> 此命令必须在 `koishi-app` 根目录运行（即工作区根目录），而不是插件目录。

初始创建的插件版本号为 1.0.0。当你修改过插件后，你需要更新版本号才能重新发布。在 **应用目录** 运行下面的命令以更新版本号：

```bash
npm run bump [...name] -- [-1|-2|-3|-p|-v <ver>] [-r]
```

- `name`: 要更新的插件列表，缺省时表示全部 (此处 name 不包含 koishi-plugin- 前缀，而是你的工作区目录名)
- `-1, --major`: 跳到下一个大版本，例如 3.1.4 -> 4.0.0
- `-2, --minor`: 跳到下一个中版本，例如 3.1.4 -> 3.2.0
- `-3, --patch`: 跳到下一个小版本，例如 3.1.4 -> 3.1.5
- `-p, --prerelease`: 跳到下一个预览版本，具体行为如下
  - 如果当前版本是 alpha.x，则跳到 beta.0
  - 如果当前版本是 beta.x，则跳到 rc.0
  - 如果当前版本是 rc.x，则移除 prerelease 部分
  - 其他情况下，跳到下一个大版本的 alpha.0
- `-v, --version`: 设置具体的版本号
- `-r, --recursive`: 递归更新依赖版本
  - 缺省情况：按照当前版本的最后一位递增

当进行此操作时，其他相关插件的依赖版本也会同步更新，确保所有工作区内依赖的插件版本一致。进一步，如果你希望更新了依赖版本的插件也同时更新自身的版本，那么可以附加 `-r, --recursive` 选项。

## Publishing

After updating version, push to trigger automated release:

```bash
git push origin main
git push origin --tags
```

The `publish.yml` workflow will automatically:
1. Build the project
2. Publish to npm (using Trusted Publisher OIDC)
3. Generate provenance

## 工作区开发规范

> [!IMPORTANT]
> 本节中介绍的命令行都需要在 `koishi-app` 应用目录（即工作区根目录）下运行。

### 初始化工作区
如果还没有 `koishi-app` 目录，请运行以下命令初始化：

```bash
npm create koishi@latest koishi-app
```

后续所有操作均在该目录中进行。

### 创建新插件
在应用目录运行下面的命令以创建一个新的插件工作区：

```bash
npm run setup [name] -- [-c] [-m] [-G]
```
- `name`: 插件的包名，缺省时将进行提问
- `-c, --console`: 创建一个带控制台扩展的插件
- `-m, --monorepo`: 创建 monorepo 的插件
- `-G, --no-git`: 跳过 git 初始化

### 构建源代码
在生产模式下使用或发布到插件市场前，需要构建源代码。在应用目录运行：

```bash
npm run build [...name]
```
- `name`: 要构建的插件列表，缺省时表示全部插件

例如 `example` 插件：
- 后端代码输出到 `external/example/lib`
- 前端代码输出到 `external/example/dist`

### 添加依赖
如果需要添加其他依赖，使用 `-w` 参数指定工作区：

```bash
npm install [...deps] -w koishi-plugin-[name]
```
- `name`: 你的插件名称 (e.g. `opencode`)
- `deps`: 要添加的依赖列表
- 添加 `devDependencies` 或 `peerDependencies` 需加上 `-D` 或 `-P` 参数。

### 更新依赖版本
批量更新所有工作区的依赖版本：

```bash
npm run dep
```
这将按照每个 `package.json` 中声明的依赖版本进行更新（例如 `^1.1.4` -> `^1.2.3`）。

### 二次开发
与其他插件协同工作或调试时，可以将其他仓库克隆到 `external` 目录：

```bash
npm run clone [repo]
# 例如: npm run clone koishijs/koishi-plugin-forward
```

## Documentation

- [SDK Type Definitions](./docs/SDK_TYPES.md)

## Links

- [Koishi Plugin Registry](https://registry.koishi.chat/)
- [Koishi Development Guide](https://koishi.chat/zh-CN/guide/develop/workspace.html)
- [Koishi Element (Message Format)](https://koishi.chat/zh-CN/guide/basic/element.html)
- [npm Documentation](https://docs.npmjs.com/trusted-publishers)

---
> Source: [DoiiarX/koishi-plugin-opencode](https://github.com/DoiiarX/koishi-plugin-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
