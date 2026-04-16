## flykylin

> C++20编码标准和最佳实践


<coding_standards>

# 语言和技术栈
- 项目使用 C++20 标准
- UI框架使用 Qt 6.5+
- 跨平台支持：Windows x64 和 RK3566 Linux ARM64
- AI推理使用 ONNX Runtime（Windows DirectML / RK3566 NPU）

# 命名规范
- **类名**：PascalCase（如 `PeerDiscovery`、`AIEngine`）
- **接口类**：`I_` 前缀（如 `I_Accelerator`、`I_CommunicationService`）
- **成员变量**：`m_` 前缀 + camelCase（如 `m_userName`、`m_socket`）
- **局部变量**：camelCase（如 `localCount`、`tempName`）
- **常量**：`k` 前缀 + PascalCase（如 `kDefaultPort`、`kMaxConnections`）
- **函数**：camelCase（如 `startDiscovery()`、`checkNsfw()`）
- **获取器/设置器**：`get`/`set` 前缀（如 `getUserName()`、`setPort()`）
- **布尔函数**：`is`/`has` 前缀（如 `isConnected()`、`hasPermission()`）

# 内存管理
- 优先使用智能指针：`std::unique_ptr`、`std::shared_ptr`
- Qt对象使用父子关系自动管理内存
- 跨线程Qt对象使用 `QPointer` 弱引用
- 禁止裸指针 `new`/`delete`，除非必要
- 遵循RAII原则：资源在构造函数获取，析构函数释放

# 代码格式
- 使用 clang-format 自动格式化（配置已提供）
- 缩进：4空格，禁止Tab
- 行宽：最大100列
- 大括号：K&R风格（开括号不换行）
- 指针和引用：靠左对齐（`int* ptr` 而非 `int *ptr`）

# 错误处理
- 使用异常处理不可恢复的错误
- 使用 `std::optional` 表示可能失败的返回值
- 检查所有可能失败的操作返回值
- Qt操作失败时使用 `qCritical()` 记录错误

# 线程安全
- 多线程访问共享数据必须使用 `std::mutex` 或 `QMutex`
- 跨线程信号槽使用 `Qt::QueuedConnection`
- 使用 `std::lock_guard` 或 `QMutexLocker` 自动管理锁

# C++20特性使用
- 使用 Concepts 约束模板参数
- 使用 Ranges 简化STL算法
- 使用 `auto` 类型推导（当类型明显时）
- 使用结构化绑定（如 `auto [key, value] = pair;`）
- 使用 `constexpr` 和 `consteval` 进行编译期计算

# Qt最佳实践
- 信号槽连接优先使用新语法（函数指针）
- 使用 `Q_OBJECT` 宏启用元对象系统
- 使用 `Q_PROPERTY` 声明属性用于QML绑定
- 长时间操作使用 `QtConcurrent::run()` 或 QThread
- UI操作必须在主线程

# 注释规范
- 使用 Doxygen 风格注释
- 公共API必须添加完整文档注释
- 复杂算法添加实现说明
- TODO/FIXME/NOTE标记待办事项
- 避免无意义的注释（代码应自解释）

# 性能考虑
- 传递大对象使用 `const&` 或移动语义
- 循环中避免不必要的拷贝
- 使用 `reserve()` 预分配容器空间
- 避免过早优化，先保证正确性
- RK3566平台注意内存和CPU限制

# 安全规范
- 输入验证：检查所有外部输入
- 缓冲区：使用Qt容器避免越界
- NSFW检测：图片传输前双端检测
- 网络通信：使用Protobuf序列化避免注入

</coding_standards>

<code_review_checklist>

## 提交前检查
- [ ] 编译无警告（`-Wall -Wextra`）
- [ ] 通过 clang-format 格式化
- [ ] 通过 clang-tidy 静态分析
- [ ] 单元测试覆盖率 ≥ 80%
- [ ] 添加必要的文档注释
- [ ] 无内存泄漏（Valgrind验证）

## Code Review要点
- [ ] 功能正确实现需求
- [ ] 符合架构设计
- [ ] 遵循编码规范
- [ ] 线程安全（如适用）
- [ ] 错误处理完善
- [ ] 性能无明显问题

</code_review_checklist>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Triwalt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
