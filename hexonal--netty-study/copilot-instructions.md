## netty-study

> 本规则旨在帮助开发者系统性地阅读 Netty 源码，建议按照以下顺序和结构逐步深入：

# Netty 源码阅读规则

本规则旨在帮助开发者系统性地阅读 Netty 源码，建议按照以下顺序和结构逐步深入：

## 1. 总体结构
- Netty 采用多模块结构，核心模块包括：`buffer`、`common`、`transport`、`handler`、`resolver`、`codec` 等。
- 推荐先关注 `netty/transport`、`netty/buffer`、`netty/common`，再逐步扩展到 `handler`、`resolver`、`codec`。

## 2. 推荐阅读顺序（入口优先）
- 强烈建议"从入口类开始，逐步深入"地阅读源码，先理解整体流程和主线，再关注细节实现。

1. **入口与引导（优先阅读）**
   - [Bootstrap.java](mdc:netty/transport/src/main/java/io/netty/bootstrap/Bootstrap.java)
   - [ServerBootstrap.java](mdc:netty/transport/src/main/java/io/netty/bootstrap/ServerBootstrap.java)
   - [AbstractBootstrap.java](mdc:netty/transport/src/main/java/io/netty/bootstrap/AbstractBootstrap.java)
2. **核心通道与事件循环**
   - [Channel.java](mdc:netty/transport/src/main/java/io/netty/channel/Channel.java)
   - [EventLoop.java](mdc:netty/transport/src/main/java/io/netty/channel/EventLoop.java)
   - [EventLoopGroup.java](mdc:netty/transport/src/main/java/io/netty/channel/EventLoopGroup.java)
   - [ChannelFuture.java](mdc:netty/transport/src/main/java/io/netty/channel/ChannelFuture.java)
   - [ChannelPromise.java](mdc:netty/transport/src/main/java/io/netty/channel/ChannelPromise.java)
   - [ChannelFutureListener.java](mdc:netty/transport/src/main/java/io/netty/channel/ChannelFutureListener.java)
3. **管道与处理器**
   - [ChannelPipeline.java](mdc:netty/transport/src/main/java/io/netty/channel/ChannelPipeline.java)
   - [ChannelHandler.java](mdc:netty/transport/src/main/java/io/netty/channel/ChannelHandler.java)
4. **地址解析与连接**
   - [AddressResolver.java](mdc:netty/resolver/src/main/java/io/netty/resolver/AddressResolver.java)
   - [AddressResolverGroup.java](mdc:netty/resolver/src/main/java/io/netty/resolver/AddressResolverGroup.java)
   - [DefaultAddressResolverGroup.java](mdc:netty/resolver/src/main/java/io/netty/resolver/DefaultAddressResolverGroup.java)
   - [NameResolver.java](mdc:netty/resolver/src/main/java/io/netty/resolver/NameResolver.java)
5. **常用工具与基础设施**
   - [ObjectUtil.java](mdc:netty/common/src/main/java/io/netty/util/internal/ObjectUtil.java)
   - [InternalLogger.java](mdc:netty/common/src/main/java/io/netty/util/internal/logging/InternalLogger.java)

## 3. 阅读建议
- 建议先通读每个类的 Javadoc 注释，理解其设计意图。
- 结合 Netty 官方文档和 Wiki（https://netty.io/wiki/）进行辅助理解。
- 阅读源码时，关注接口与实现的分离、异步事件驱动模型、管道与 handler 链的设计。
- 推荐结合实际 Demo（见 `example` 目录）进行调试和断点分析。

## 4. 进阶阅读
- 理解 `buffer`、`codec`、`handler` 等模块的实现细节。
- 关注 Netty 的内存管理、线程模型、零拷贝等高性能设计。

## 5. 注释与日志规范
- 在完善源码注释时，所有涉及日志（logger）的注释和用法需符合 Netty 规范：
  - 日志应使用 `InternalLogger`，并通过 `InternalLoggerFactory` 获取实例。
  - 注明日志级别（如 debug/info/warn/error），并说明典型使用场景。
  - 异常日志应包含异常堆栈，便于排查。
  - 日志内容应简洁明了，避免泄露敏感信息。
  - 推荐参考 Netty 源码中的日志写法和注释风格。
  - 对于重要的节点，你需要增加一些详细的注释
  - 注释需要变更为中文

## 6. 边界说明

- 本规则仅适用于源码注释的补充、完善和调整，不涉及对任何代码实现、结构或逻辑的优化与修改。
- 所有操作仅限于注释层面，严禁对源码本身做出任何功能性变更。

---
如需深入某一模块，可在本规则基础上补充更细致的阅读路径。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hexonal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
