## evo

> - `src/grammar/` - 语法定义系统（AST、数据类型、规则）

# Cursor Rules for Evo-lang Project

## 项目结构 / Project Structure

### 核心目录
- `src/grammar/` - 语法定义系统（AST、数据类型、规则）
- `src/parser/` - 解析器（自适应解析、自然语言理解）
- `src/runtime/` - 运行时（解释器、JIT编译器）
- `src/evolution/` - 进化引擎（15个模块，包括分析、生成、优化等）
- `src/python/` - Python互操作层（PyO3集成）
- `src/poetry/` - 诗歌理解模块
- `docs/` - 文档目录
- `examples/` - 示例代码
- `modules/` - Evo-lang模块文件（.evo）

### 关键文档
- `README.md` - 项目主文档，包含项目结构、特性、使用示例
- `ARCHITECTURE.md` - 架构概览，模块索引，数据流
- `docs/API-REFERENCE.md` - API快速参考，函数签名
- `docs/getting-started.md` - 快速入门指南
- `docs/syntax-reference.md` - 语法参考

## 优先读取顺序 / Priority Reading Order

当需要理解或修改代码时，按以下顺序查找信息：

1. **入口文档**
   - `README.md` 的项目结构部分
   - `ARCHITECTURE.md` 的模块索引部分

2. **API参考**
   - `docs/API-REFERENCE.md` - 查找函数签名和接口

3. **模块文档**
   - 相关模块的 `mod.rs` 文件中的文档注释
   - 模块级注释通常包含导航信息

4. **具体实现**
   - 根据索引定位到具体 `.rs` 文件
   - 查看关键方法和数据结构

5. **示例代码**
   - `examples/` 目录中的 `.evo` 文件
   - `examples/README.md` 了解示例用途

## 代码风格 / Code Style

### Rust代码
- 遵循标准 `rustfmt` 风格
- 关键函数必须有文档注释（使用 `///`）
- 模块级文档注释（使用 `//!`）应包含：
  - 模块职责说明
  - 快速导航信息
  - 关键数据结构和方法

### 文档注释格式
```rust
//! # Module Name
//! 
//! ## 职责 / Responsibility
//! Brief description of what this module does.
//! 
//! ## 快速导航 / Quick Navigation
//! - `file1.rs` - Description
//! - `file2.rs` - Description

/// Function description
/// 
/// # Arguments
/// * `arg` - Argument description
/// 
/// # Returns
/// Return value description
pub fn function(arg: Type) -> ReturnType {
    // ...
}
```

## 常用模式 / Common Patterns

### 错误处理
- 使用 `Result<T, E>` 返回错误
- 自定义错误类型，提供详细的错误信息
- 在关键路径进行错误传播

### 类型系统
- `Value` - 运行时值（在 `grammar/core.rs` 定义）
- `GrammarElement` - AST节点（在 `grammar/core.rs` 定义）
- `EvolutionEvent` - 进化事件（在 `evolution/tracker.rs` 定义）

### 模块导入
```rust
// 优先使用模块路径，避免过度使用 pub use
use evo::grammar::core::GrammarElement;
use evo::parser::AdaptiveParser;
use evo::runtime::Interpreter;
```

## 修改建议 / Modification Guidelines

### 添加新功能时
1. 先查看 `ARCHITECTURE.md` 找到合适的模块位置
2. 检查 `docs/API-REFERENCE.md` 了解相关接口
3. 查看现有实现，遵循相同模式
4. 更新相关文档和API参考

### 修改核心类型时
- `grammar/core.rs` 中的 `Value` 和 `GrammarElement` 是核心类型
- 修改这些类型会影响整个系统，需要谨慎
- 确保更新所有相关模块

### 添加新分析工具时
- 放在 `src/evolution/` 目录
- 在 `evolution/mod.rs` 中导出
- 遵循现有分析工具的接口模式
- 考虑集成到 `EvolutionEngine` 中

## 测试和示例 / Testing and Examples

- 示例代码在 `examples/` 目录
- 每个 `.evo` 文件展示特定功能
- 运行示例：`cargo run`
- Python集成示例：查看 `python/README.md` 和 `test_python.py`

## 关键路径 / Key Paths

### 解析流程
```
grammar/core.rs → parser/adaptive.rs → runtime/interpreter.rs
```

### 进化流程
```
evolution/learning.rs → evolution/knowledge.rs → evolution/engine.rs → evolution/tracker.rs
```

### Python集成
```
src/lib.rs (PyO3导出) → python/bridge.rs → parser/ + runtime/
```

## 注意事项 / Notes

1. **自进化特性** - 这是核心特性，语言能够自我改进，不要破坏这个机制
2. **模块化设计** - 保持模块职责单一，接口清晰
3. **向后兼容** - 修改API时考虑对现有代码的影响
4. **文档同步** - 修改代码时同步更新相关文档
5. **性能考虑** - JIT编译器和优化器关注性能，注意不要引入性能回归

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juoon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
