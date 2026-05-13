## httpmediacache

> - 这是 `HTTPMediaCache`，一个 Swift-only 的本地 HTTP 多媒体缓存库。

# AGENTS.md

本文件适用于当前目录及所有子目录。

## 项目概览

- 这是 `HTTPMediaCache`，一个 Swift-only 的本地 HTTP 多媒体缓存库。
- 核心库位于 `Sources/HTTPMediaCache`，通过 Swift Package Manager 构建。
- 测试位于 `Tests/HTTPMediaCacheTests`，使用 `XCTest`。
- 示例工程位于 `Example/HTTPMediaCacheExample.xcodeproj`，包含 iOS 与 macOS 示例。
- 主要运行环境：Swift 5.10+、iOS 15.0+、macOS 12.0+。
- 当前核心第三方依赖为 `swift-nio`，主要用于本地 HTTP 代理服务。

## 回复与文案

- 默认使用简体中文回复。
- 代码注释、文档和 Git commit 信息默认使用简体中文，除非用户明确要求其他语言。
- 代码里的错误信息描述和日志信息默认使用英文，避免运行时输出混用中文。
- 用户是有经验开发者，不解释基础语法或入门概念。
- 优先给出高质量方案，减少无谓往返。
- 复杂任务先给 1-3 个方案与权衡，确认后再实现；简单或中等任务可以直接处理。

## Git 与变更边界

- 只有在用户明确同意后才执行 `git commit`。
- 禁止执行 `git push` 和 `git push --force`。
- 未经明确要求，避免破坏性 Git 操作，例如 `git reset --hard`、`git clean -f`、`git rebase`、`git commit --amend`。
- 不要回退或覆盖用户已有改动；遇到脏工作区时只处理当前任务相关文件。
- 删除文件、修改公共 API、新增第三方依赖前必须先确认。
- 避免无关重构、无关格式化和大范围机械改动。

## Swift 代码规范

- 新建或修改 `.swift` 文件后，必须只对该文件执行 `swiftformat <file>`。
- 禁止执行目录级或全项目格式化，例如 `swiftformat .`、`swiftformat Sources`、`swiftformat Tests`。
- 新建 Swift 文件必须包含完整文件头，作者或组织信息使用项目约定或实际贡献者信息，不要硬编码个人信息；日期使用创建当天实际日期：

  ```swift
  //
  //  <FileName>.swift
  //  HTTPMediaCache
  //
  //  Created by <AuthorOrOrganization> on <YYYY/M/D>.
  //
  ```

- 公开 API 需要保持中文文档注释清晰，描述行为、参数和重要边界。
- 优先保持现有 async/await、actor、Swift concurrency 风格，不引入不必要的回调式封装。
- 核心缓存、Range、HLS、代理和下载链路改动要保持职责边界清晰：
  - `API` 暴露公共入口和模型。
  - `Core` 放通用运行时、Range、URL 编解码等核心能力。
  - `Storage` 负责缓存索引、文件存储和缓存单元。
  - `Network` 负责下载抽象和 URLSession 下载实现。
  - `Pipeline` 负责读取、规划和组合缓存/远端数据。
  - `Proxy` 负责本地代理、请求路由和响应写入。
  - `HLS` 负责 playlist 解析、改写和预加载策略。
- 示例工程代码位于 `Example`，不要把示例层逻辑反向引入核心库。

## 测试与验证

- 默认测试命令：

  ```bash
  swift test
  ```

- 修改具体模块时优先运行相关测试，最后根据影响范围决定是否运行完整 `swift test`。
- 行为变更必须补充或更新测试，尤其是：
  - Range 解析与响应。
  - 缓存索引、缓存区间合并和文件存储。
  - HLS playlist 改写、segment/key/init map 处理。
  - 预加载策略、取消、并发限制和进度。
  - 本地代理请求路由和 NIO 响应。
  - 下载配置、请求头、content type 和错误记录。
- 测试中需要临时存储时优先使用 `FileManager.default.temporaryDirectory` 下的唯一目录，并做好清理。
- 不要依赖真实公网媒体资源作为单元测试前提；优先使用测试替身、固定数据和本地可控输入。

## 构建与示例

- 打开示例工程：

  ```bash
  open Example/HTTPMediaCacheExample.xcodeproj
  ```

- 示例工程包含以下共享或平台代码：
  - `Example/Shared`
  - `Example/iOS`
  - `Example/macOS`
- 修改示例 UI 或播放链路时，确认 iOS 与 macOS 是否都受影响。

## 文档要求

- README 示例应与真实 API 保持一致。
- 修改公共 API、安装方式、预加载语义、缓存清理语义或平台要求时，同步更新 README。
- 变更用户可见行为时，视情况更新 `CHANGELOG.md`。
- PR 描述应包含变更内容、验证方式和兼容性影响。

---
> Source: [sunimp/HTTPMediaCache](https://github.com/sunimp/HTTPMediaCache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
