## bot

> > **注意**: 本规范适用于 `aumate-app/` 和 `crates/` 下的 Tauri 应用开发。

# Aumate App 开发规范

> **注意**: 本规范适用于 `aumate-app/` 和 `crates/` 下的 Tauri 应用开发。
> 对于 Node.js automation library (`packages/`)，请参考 `CLAUDE.md`。

## 🏗️ 架构原则

### DDD 分层架构

本项目采用 Domain-Driven Design (DDD) 分层架构，严格遵循依赖倒置原则：

```
API Layer (Tauri Commands)
    ↓ 调用
Application Layer (Use Cases)
    ↓ 依赖接口
Domain Layer (Ports - Trait接口)
    ↑ 实现接口
Infrastructure Layer (Adapters)
```

### 核心规则

1. **依赖方向**
   - Domain Layer 定义接口（Ports），不依赖任何其他层
   - Infrastructure Layer 实现接口（Adapters）
   - Application Layer 通过接口调用，不直接依赖具体实现
   - API Layer 仅做参数验证和调用 Use Cases

2. **单一职责**
   - API Layer: 参数验证、错误转换
   - Application Layer: 业务流程编排
   - Domain Layer: 领域模型和业务规则
   - Infrastructure Layer: 技术实现细节

3. **命名约定**
   - Port接口: `xxxPort` (如 `ScreenCapturePort`)
   - Adapter实现: `xxxAdapter` (如 `ScreenCaptureAdapter`)
   - Use Case: `xxxUseCase` (如 `CaptureScreenUseCase`)
   - DTO: `xxxDto` / `xxxRequest` / `xxxResponse`

## 📦 代码组织

### Crate 结构

```
crates/
├── core/
│   ├── shared/       # 共享类型、错误定义
│   ├── domain/       # 领域模型
│   └── traits/       # Port 接口定义
├── application/      # Use Cases + DTOs
└── infrastructure/   # Adapters + Services

aumate-app/src-tauri/ # API Layer (应用特定)
└── src/
    ├── commands/     # Tauri Commands
    ├── state.rs      # AppState
    └── setup.rs      # 依赖注入
```

### 新功能实现流程

当添加新功能时，按照以下顺序实现：

#### 1. Domain Layer - 定义接口

**位置**: `crates/core/traits/src/`

```rust
// crates/core/traits/src/my_feature.rs
use async_trait::async_trait;
use aumate_core_shared::InfrastructureError;

#[async_trait]
pub trait MyFeaturePort: Send + Sync {
    async fn do_something(&self) -> Result<MyResult, InfrastructureError>;
}
```

**导出**: 在 `crates/core/traits/src/lib.rs` 中添加：
```rust
pub mod my_feature;
pub use my_feature::MyFeaturePort;
```

#### 2. Infrastructure Layer - 实现 Adapter

**位置**: `crates/infrastructure/src/adapters/`

```rust
// crates/infrastructure/src/adapters/my_feature.rs
use async_trait::async_trait;
use aumate_core_traits::MyFeaturePort;

pub struct MyFeatureAdapter {
    // 字段
}

impl MyFeatureAdapter {
    pub fn new() -> Self {
        Self {}
    }
}

#[async_trait]
impl MyFeaturePort for MyFeatureAdapter {
    async fn do_something(&self) -> Result<MyResult, InfrastructureError> {
        // 实现逻辑
        // 如果需要平台特定代码，使用：
        #[cfg(target_os = "macos")]
        {
            // macOS 实现
        }
        
        #[cfg(target_os = "windows")]
        {
            // Windows 实现
        }
    }
}
```

**平台特定代码** (如需要):
- 位置: `crates/infrastructure/src/platform/macos/` (或 `windows/`, `linux/`)
- 在 Adapter 中调用平台特定函数

**导出**: 在 `crates/infrastructure/src/adapters/mod.rs` 中添加：
```rust
pub mod my_feature;
pub use my_feature::MyFeatureAdapter;
```

#### 3. Application Layer - Use Case + DTO

**Use Case** (`crates/application/src/use_cases/my_feature.rs`):
```rust
use aumate_core_shared::UseCaseError;
use aumate_core_traits::MyFeaturePort;
use std::sync::Arc;

pub struct MyFeatureUseCase {
    feature: Arc<dyn MyFeaturePort>,
}

impl MyFeatureUseCase {
    pub fn new(feature: Arc<dyn MyFeaturePort>) -> Self {
        Self { feature }
    }

    pub async fn execute(&self) -> Result<MyResultDto, UseCaseError> {
        log::info!("[MyFeatureUseCase] Executing");

        let result = self.feature
            .do_something()
            .await
            .map_err(|e| e.into())?;

        // 转换为 DTO
        Ok(result.into())
    }
}
```

**DTO** (`crates/application/src/dto/my_feature.rs`):
```rust
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct MyResultDto {
    // 字段
}

// 实现 From trait 进行转换
impl From<DomainType> for MyResultDto {
    fn from(value: DomainType) -> Self {
        Self {
            // 转换逻辑
        }
    }
}
```

**导出**: 
- `crates/application/src/use_cases/mod.rs` 添加 `pub mod my_feature;` 和 `pub use my_feature::*;`
- `crates/application/src/dto/mod.rs` 添加 `pub mod my_feature;` 和 `pub use my_feature::*;`

#### 4. API Layer - Tauri Command (仅在应用层)

**注意**: API commands 只在 `aumate-app/src-tauri/src/commands/` 中，不在 crates 里

**位置**: `aumate-app/src-tauri/src/commands/my_feature.rs`

```rust
use crate::state::AppState;
use aumate_application::dto::MyResultDto;
use aumate_core_shared::ApiError;
use tauri::State;

#[tauri::command]
pub async fn do_my_feature(
    state: State<'_, AppState>,
) -> Result<MyResultDto, String> {
    log::info!("API: do_my_feature called");

    state.my_feature_use_case
        .execute()
        .await
        .map_err(|e| {
            let api_error: ApiError = e.into();
            api_error.to_string()
        })
}
```

**注册命令**: 在 `aumate-app/src-tauri/src/lib.rs` 的 `invoke_handler` 中添加

#### 5. 依赖注入 - AppState

**State** (`aumate-app/src-tauri/src/state.rs`):
```rust
pub struct AppState {
    // ... existing fields ...
    pub my_feature_adapter: Arc<MyFeatureAdapter>,
    pub my_feature_use_case: Arc<MyFeatureUseCase>,
}
```

**Setup** (`aumate-app/src-tauri/src/setup.rs`):
```rust
pub fn setup_application() -> AppState {
    // 1. 创建 Adapter
    let my_feature_adapter = Arc::new(MyFeatureAdapter::new());
    
    // 2. 创建 Use Case
    let my_feature_use_case = Arc::new(MyFeatureUseCase::new(
        my_feature_adapter.clone()
    ));
    
    // 3. 返回 AppState
    AppState {
        // ... existing fields ...
        my_feature_adapter,
        my_feature_use_case,
    }
}
```

## ⚠️ 重要禁止事项

### ❌ 不要在 Commands 中直接实现业务逻辑

**错误示例**:
```rust
#[tauri::command]
pub async fn get_window_elements() -> Result<Vec<WindowElement>, String> {
    // ❌ 错误：直接在 command 中实现逻辑
    #[cfg(target_os = "macos")]
    {
        use core_graphics::window::CGWindowListCopyWindowInfo;
        // ... 大量实现代码 ...
    }
}
```

**正确做法**:
```rust
#[tauri::command]
pub async fn get_window_elements(
    state: State<'_, AppState>
) -> Result<Vec<WindowElementDto>, String> {
    // ✅ 正确：仅调用 Use Case
    state.get_window_elements
        .execute()
        .await
        .map_err(|e| e.into())
}
```

### ❌ 不要跨层直接依赖

- Domain Layer 不能依赖 Infrastructure/Application/API
- Application Layer 不能直接依赖 Adapter 实现（只能依赖 Port 接口）
- Infrastructure Layer 不能依赖 Application/API

### ❌ 不要在错误的地方定义类型

- 领域模型: `core/domain/` (如 `Image`, `Screenshot`)
- 共享类型: `core/shared/` (如 `Rectangle`, `Point`, `MonitorId`)
- DTO: `application/dto/` (用于 API 响应)
- Port 接口: `core/traits/`

## 🔍 代码审查清单

提交代码前，检查：

- [ ] 新功能是否按照 DDD 分层实现？
- [ ] Port 接口是否在 `core/traits` 定义？
- [ ] Adapter 是否在 `infrastructure/adapters` 实现？
- [ ] Use Case 是否在 `application/use_cases` 实现？
- [ ] DTO 是否在 `application/dto` 定义？
- [ ] API Command 是否只做参数验证和调用 Use Case？
- [ ] 是否更新了模块导出 (`mod.rs`, `lib.rs`)?
- [ ] 是否更新了 `AppState` 和 `setup_application`？
- [ ] 是否在 `lib.rs` 中注册了新的 command？
- [ ] 是否更新了相关文档 (README.md, ARCHITECTURE.md)?
- [ ] 代码是否通过 `cargo check --workspace` 检查？
- [ ] 是否添加了必要的测试？

## 📝 常用命令

```bash
# 检查整个工作区
cargo check --workspace

# 检查特定 crate
cargo check --package aumate-infrastructure

# 运行测试
cargo test --workspace

# 格式化代码
cargo fmt --all

# Lint 检查
cargo clippy --workspace -- -D warnings

# 编译发布版本
cd aumate-app && pnpm tauri build
```

## 🔗 相关文档

- **项目总览**: `CLAUDE.md` - 包含 Tego Bot 和 Aumate App 的完整信息
- **架构详解**: `crates/docs/ARCHITECTURE.md` - Aumate App DDD 架构说明
- **项目概览**: `crates/docs/README.md` - Crates 结构和模块说明
- **最新变更**: `HOTKEY_CHANGES.md` - 全局快捷键 DDD 实现示例

## 📈 最新进展 (2024-12)

### ✅ 已完成
1. **统一依赖管理**
   - 所有依赖版本集中在根 `Cargo.toml` 的 `workspace.dependencies`
   - 子 crate 使用 `workspace = true` 引用
   - 解决了版本冲突问题

2. **Windows 平台修复**
   - 添加 `uiautomation` 和 `windows` crate 到 infrastructure
   - 修复 `Rectangle` API 变更
   - 实现跨平台 `MonitorInfo`

3. **全局快捷键管理** (DDD 架构示例)
   - ✅ Port: `GlobalShortcutPort` (`crates/core/traits/src/global_shortcut.rs`)
   - ✅ Adapter: `GlobalShortcutAdapter` (`crates/infrastructure/src/adapters/global_shortcut.rs`)
   - ✅ Use Cases: Register/Unregister/CheckAvailability (`crates/application/src/use_cases/global_shortcut.rs`)
   - ✅ Commands: `register_global_shortcut`, `unregister_global_shortcut`, `check_global_shortcut_availability`
   - ✅ 默认截图热键从 F2 改为 Ctrl+4，避免冲突
   - ✅ 优雅处理热键注册失败（记录警告而非崩溃）

### 📝 开发检查清单
提交代码前，使用此清单确保符合规范：
- [ ] 遵循 DDD 分层架构（Port → Adapter → Use Case → Command）
- [ ] 所有依赖使用 `workspace = true`
- [ ] 通过 `cargo check --workspace` 检查
- [ ] 更新了相关模块导出 (`mod.rs`, `lib.rs`)
- [ ] 在 `AppState` 和 `setup.rs` 中正确注入依赖
- [ ] 在 `lib.rs` 中注册了新的 Tauri commands
- [ ] 添加了必要的文档注释

---

**记住**: 保持架构清晰，代码组织有序，遵循 DDD 原则，让代码更易维护和扩展！

---
> Source: [tegojs/bot](https://github.com/tegojs/bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
