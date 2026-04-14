## card5

> - 使用 [QFramework.cs](mdc:Assets/Scripts/Core/QFramework.cs) 作为基础架构

# 架构指南

## 基础工具
- 使用 [QFramework.cs](mdc:Assets/Scripts/Core/QFramework.cs) 作为基础架构
    - 将模块分为 `Model` 、 `System` 、 `Controller` 、 `Utility`
    - 大多数 `Monobehaviour` 都应该实现 `IController` 来接入QFramework
    - 在 Controller 中使用 Command 对 `Model` 进行操作
    - 在 Controller 中使用 Query 对 `Model` 信息进行查询
    - 在 Controller 中注册 Event 来实现事件回调
    - Controller 中只能注册 Event，不能发送 Event
    - 在 System、Model、Command中完成操作时发送 Event 通知 Controller
- 使用 `UniTask` 处理异步操作
    - 尽量使用异步操作代替协程、 `Update` 等
    - 使用 `Forget()` 处理不需要等待的异步操作
    - 及时合理的位置释放 `AsyncOperationHandle`
- 使用Unity的 `Addressables` 进行资源管理
    - 使用 `AddressablesManager.Release` 释放异步资源加载产生的handle
    - 使用 `Addressables.InstantiateAsync` 加载 Prefab
    - 使用 `Addressables.ReleaseInstance` 卸载实例
- 使用 Unity 的新输入系统 InputSystem
    - 在 `OnEnable` 中注册输入事件，在 `OnDisable` 中注销
- 在开发中使用 `Odin Inspector` 构建方便使用的Editor UI
- 使用 `SerializedScriptableObject` 或 `ScriptableObject` 作为主要的数据配置类
- 使用 `PrimeTween` 进行动画和补间操作

## UI

- UI脚本所需组件应设为 `SerializeField`，并在 `OnValidate` 中检查并尝试创建
- 可使用 Unity MCP 进行UI创建与布局

## 系统

- 输入系统
- 存档系统
- 卡牌系统

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IshmaelHeathcliff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IshmaelHeathcliff)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
