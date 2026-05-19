## coding-standards

> ElegooSlicer 编码规范 - 编辑代码文件时应用


# 编码规范

重要原则：这是旧项目，修改现有代码时优先保持文件内风格一致，新代码遵循本规范。

## 命名与结构

- 类/结构/枚举：`PascalCase`；函数/方法/变量：`camelCase`（函数名以动词开头）
- 成员变量：`m`+CamelCase（如 `mPrinterList`）；全局：`g`+PascalCase；静态：`s`+PascalCase
- 常量/宏：`ALL_CAPS`；布尔变量：动词前缀（`isReady`、`hasError`、`canUpdate`）
- 头文件用 `#pragma once`，尽量前置声明减少依赖
- include 顺序（组间空行）：配对头文件 → 项目头文件 → 第三方库 → 标准库
- 模块依赖遵循单向原则：底层模块（Utils/Core）不依赖上层模块（GUI/Business）
- 跨模块调用优先通过接口/回调，避免直接依赖具体实现
- 路径处理用 `std::filesystem` 或 `boost::nowide` 支持中文

## 函数与类

- 每个函数/类专注单一职责，函数名/类名准确描述功能
- 布尔返回用 `is/has/can`，void 用 `execute/save/update`；早返回减少嵌套
- 标量/小对象按值，大对象用 `const T&`，修改用 `T&`，可选用 `std::optional<T>`
- 输出参数优先返回值（结构体/`tuple`/`optional`/`expected`），引用输出用 `outXYZ`
- 缓冲区用 `std::span<T>`，只读字符串用 `std::string_view`
- 裸指针表达"可空且无所有权"，所有权转移用 `unique_ptr`/`shared_ptr`
- 指针或引用参数函数入口应校验，用 `const` 限定不可修改对象
- 优先组合而非继承；能 `const` 则 `const`，能 `constexpr` 则 `constexpr`
- 成员优先私有提供 getter/setter，成员函数不修改状态时加 `const`

## 内存与错误

- 遵循 RAII，除 GUI 控件及第三方库接口外避免裸 `new/delete`，优先用 `unique_ptr`/`shared_ptr`
- wxWidgets 控件由父子关系管理生命周期，自定义控件需明确所有权
- 不要在循环中频繁分配释放内存，考虑复用或预分配
- 预期失败优先用 `optional`/`expected`/错误码，不可恢复错误用标准异常并添加上下文
- 输入参数应在函数边界校验，错误信息包含上下文/路径/参数值（避免泄露隐私）

## 现代 C++

- `auto` 仅用于复杂迭代器/lambda，基础类型（`int`/`double`/`bool`）显式声明
- 优先 C++17（范围 for、结构化绑定），`enum class` 枚举，`nullptr` 空指针
- 避免 C 风格转换，改用 `static_cast`/`dynamic_cast`；容器优先 `vector`/`map`/`unordered_map`
- 使用 `std::move` 减少拷贝
- 避免不必要的堆分配，尽可能优先使用基于栈的对象
- 使用 `<algorithm>` 中的算法优化循环（例如，`std::sort`，`std::for_each`）
- 避免全局变量，谨慎使用单例模式
- 在类中分离接口和实现
- 明智地使用模板和元编程来实现通用解决方案

## 并发与线程

- UI 更新必须在主线程，后台线程通过 `CallAfter`/事件队列通信，避免直接操作控件
- 使用 `thread`/`mutex`/`lock_guard`/`atomic` 保证线程安全，共享资源明确所有权与锁策略
- 跨线程传递数据优先值拷贝或移动，避免共享指针；长时间操作使用线程池

## 注释与文档

- 代码与文档全部使用英文，变量/参数/返回值显式声明类型
- 为类、方法和关键逻辑编写清晰的注释。
- 英文注释说明"为什么"而非"做什么"，避免复述代码
- 注释只描述**代码意图/业务约束/资源所有权/线程约束**等客观信息，不要包含与协作过程相关的背景（例如“根据和 AI 的讨论/用户要求/聊天记录”之类表述）
- 修复类注释如需说明原因，使用**可追溯信息**（如 `// Fix: <brief reason> (ISSUE-ID)`），避免写成对话解释或情绪化描述
- 公共类/函数用 Doxygen：`/** @brief ... @param ... @return ... */`
- TODO 格式：`// TODO(OWNER|ISSUE-ID): description`

## 避免事项

- 不要在头文件中 `using namespace std` 或实现非模板函数
- 不要用 C 风格字符串（`char*`）和数组，改用 `std::string`/`std::vector`
- 不要忽略编译器警告，修复所有 warning

---
> Source: [ELEGOO-3D/ElegooSlicer](https://github.com/ELEGOO-3D/ElegooSlicer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
