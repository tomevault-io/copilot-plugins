## money-babystep

> 你是一名资深经验的flutter开发者，并且熟悉kotlin & swift 对android&ios的原生开发，你可以独立完成android&ios的app开发。

# background
你是一名资深经验的flutter开发者，并且熟悉kotlin & swift 对android&ios的原生开发，你可以独立完成android&ios的app开发。
包括app UI的设计，功能开发，以及原生开发。

# 项目背景
本项目是一个记账app，用户可以快捷的记录自己的收支情况，并且可以查看自己的收支情况。对于收支情况的展示则提供丰富的图表，以及金融会计指标，来让用户快速了解到其
个人财务状况。

# 项目架构
1. 本项目使用flutter来作为移动端ui的实现，使用rust来作为具体的业务逻辑实现，其中将flutter与rust进行通信，使用rinf(rust in flutter)来实现。

# 项目要求
1. 能够以flutter来实现app的UI设计
2. 出色的rust开发经验，使用rust来实现app的逻辑处理
3. 必要时，能够使用kotlin & swift 对android&ios的原生开发，来针对打开速度，内存控制，打包体积等方面进行优化。
4. 项目顶层UI的设计可以参考微信app的设计，如此来使得用户可以上手即可使用本app。
5. 项目中的flutterUI设计符合flutter的最佳实践，确保ui组件的高效与复用。
6. rust也以rust工程最佳实践提供建议与帮助。


# 项目层级架构
请参考下述的代码层级构建，来为我进行项目代码层级构建
```shell
├── native  # 业务逻辑均通过rust来实现，rust与dart则使用基于事件驱动模式的 rinf
│   ├── README.md
│   └── hub
│       ├── Cargo.toml
│       ├── migrations
│       └── src
│           ├── biz # 定义数据repo需求，以及具体的业务逻辑实现(基本上业务逻辑均在这一层实现)
│           │   └── mod.rs
│           ├── data # 实现biz定义的repo需求，当前数据存储使用的持久化方案是sqlite
│           │   └── mod.rs
│           ├── lib.rs
│           ├── server # 主要定义该服务如何启动起来，其准备服务的init
│           │   └── mod.rs
│           ├── service # 封装侨接的服务层,其调用biz，来通过组合实现业务逻辑api的支持
│           │   └── mod.rs
```
Riverpod: 用于状态管理


# 项目ai命令提示
若出现了以@flutter开头，则实现仅聚焦于flutter UI层面的实现。


# flutter实现细节
1. 当前项目使用rinf，会通过rinf gen来生成dart的结构，尽可能来减少数据模型的创建，以此来优化性能

---
alwaysApply: false
keywords: ["@flutter", "flutter", "dart", "ui"]
---

# Flutter 开发指南

## 项目特定架构

### 1. 目录结构规范

```
lib/
├── main.dart                    # 应用入口
├── constants/                   # 常量定义
│   ├── app_constants.dart      # 颜色、样式常量
│   └── app_strings.dart        # 文本常量（支持国际化）
├── models/                      # 数据模型（逐步废弃，改用 Rinf 生成）
├── pages/                       # 页面级组件
│   ├── main_frame_page.dart    # 主框架（底部导航）
│   ├── bookkeeping_page.dart   # 记账页
│   ├── asset_page.dart         # 资产页
│   └── profile_page.dart       # 个人中心
├── widgets/                     # 可复用组件
│   ├── transaction_input_sheet.dart  # 记账输入弹窗
│   ├── category_selector.dart        # 分类选择器
│   └── bottom_nav_bar.dart          # 底部导航栏
├── providers/                   # Riverpod 状态管理
│   ├── bill_provider.dart      # 账单状态（使用 Rinf）
│   ├── category_provider.dart  # 分类状态
│   └── navigation_provider.dart # 导航状态
├── services/                    # 服务层（逐步废弃）
│   └── database_service.dart   # 旧 SQLite 服务
└── src/bindings/               # Rinf 自动生成（不要手动修改）
    └── signals/                # Rust 信号绑定
```

### 2. 状态管理规范

**使用 Riverpod 作为唯一状态管理方案**

#### Provider 命名规范：
```dart
// 数据提供者
final billRepositoryProvider = Provider<BillRepository>((ref) => ...);

// 状态控制器
final createBillProvider = StateNotifierProvider.autoDispose<
  CreateBillNotifier, CreateBillState>((ref) => ...);

// 简单状态
final selectedCategoryProvider = StateProvider<int?>((ref) => null);
```

#### 状态类定义（使用 freezed）：
```dart
@freezed
class CreateBillState with _$CreateBillState {
  const factory CreateBillState({
    @Default(CreateBillStatus.initial) CreateBillStatus status,
    Bill? bill,
    String? errorMessage,
  }) = _CreateBillState;
}

enum CreateBillStatus { initial, loading, success, error }
```

### 3. Rinf 信号通信规范

#### 发送信号到 Rust：
```dart
// 1. 创建请求对象（使用 Rinf 生成的类）
CreateBillReq(
  userId: BigInt.from(userId),
  bookId: BigInt.from(bookId),
  amount: amount,
  tagIdLv1: BigInt.from(tagIdLv1),
  tagIdLv2: BigInt.from(tagIdLv2),
).sendSignalToRust();

// 2. 监听响应
CreateBillResp.rustSignalStream.listen((rustSignal) {
  final message = rustSignal.message;
  // 处理响应
});
```

#### 在 Provider 中使用：
```dart
class CreateBillNotifier extends StateNotifier<CreateBillState> {
  CreateBillNotifier() : super(const CreateBillState()) {
    _listenToRustSignals();
  }

  void _listenToRustSignals() {
    CreateBillResp.rustSignalStream.listen((rustSignal) {
      final bill = rustSignal.message.bill;
      state = state.copyWith(
        status: CreateBillStatus.success,
        bill: bill,
      );
    });
  }

  Future<void> createBill({...}) async {
    state = state.copyWith(status: CreateBillStatus.loading);
    
    CreateBillReq(...).sendSignalToRust();
  }
}
```

### 4. UI 组件规范

#### 页面组件（Page）：
- 继承 `ConsumerStatefulWidget` 或 `ConsumerWidget`
- 负责页面级别的布局和状态监听
- 使用 `ref.watch()` 监听状态，`ref.read()` 调用方法

#### 可复用组件（Widget）：
- 尽量设计为 StatelessWidget
- 通过构造函数接收参数
- 使用回调函数向上传递事件

#### 底部弹窗规范：
```dart
// 显示方式
showModalBottomSheet(
  context: context,
  isScrollControlled: true,
  backgroundColor: Colors.transparent,
  builder: (context) => const TransactionInputSheet(),
);

// 弹窗内部结构
class TransactionInputSheet extends ConsumerStatefulWidget {
  @override
  Widget build(BuildContext context) {
    // 监听状态变化
    ref.listen<CreateBillState>(createBillProvider, (previous, next) {
      if (next.status == CreateBillStatus.success) {
        Navigator.of(context).pop(); // 关闭弹窗
        // 显示成功提示
      }
    });
    
    return Container(
      decoration: BoxDecoration(
        borderRadius: BorderRadius.vertical(top: Radius.circular(20)),
      ),
      child: SafeArea(child: ...),
    );
  }
}
```

### 5. 样式规范

**所有颜色、尺寸、样式必须使用 `AppConstants`**

```dart
// ✅ 正确
Container(
  color: AppConstants.backgroundColor,
  child: Text(
    'Title',
    style: TextStyle(
      color: AppConstants.textPrimaryColor,
      fontSize: 16,
    ),
  ),
)

// ❌ 错误 - 不要硬编码颜色
Container(
  color: Color(0xFFF5F5F5),
  child: Text('Title', style: TextStyle(color: Colors.black)),
)
```

### 6. 国际化规范

```dart
// 使用 AppStrings 常量
Text(AppStrings.appName)

// 需要参数的文本，使用函数
String greetingText(String name) => '你好，$name';
```

### 7. 表单验证规范

```dart
bool get _canSubmit {
  return _amountController.text.isNotEmpty &&
      double.tryParse(_amountController.text) != null &&
      _selectedPrimaryCategoryId != null;
}

// 在按钮中使用
ElevatedButton(
  onPressed: (_canSubmit && !isSubmitting) ? _submit : null,
  child: ...,
)
```

### 8. 错误处理规范

```dart
// 在 Provider 中捕获错误
try {
  // 调用 Rust API
} catch (e) {
  state = state.copyWith(
    status: CreateBillStatus.error,
    errorMessage: e.toString(),
  );
}

// 在 UI 中显示错误
ref.listen<CreateBillState>(createBillProvider, (previous, next) {
  if (next.status == CreateBillStatus.error) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('操作失败: ${next.errorMessage}'),
        backgroundColor: Colors.red,
      ),
    );
  }
});
```

### 9. 性能优化规范

#### 使用 `autoDispose`：
```dart
// 页面销毁时自动清理状态
final myProvider = StateNotifierProvider.autoDispose<MyNotifier, MyState>(
  (ref) => MyNotifier(),
);
```

#### 避免不必要的重建：
```dart
// ✅ 正确 - 只监听需要的字段
final status = ref.watch(createBillProvider.select((s) => s.status));

// ❌ 错误 - 监听整个状态
final state = ref.watch(createBillProvider);
```

#### 使用 const 构造函数：
```dart
const Text('静态文本')
const SizedBox(height: 16)
const Icon(Icons.add)
```

### 10. 代码组织规范

#### 文件内组织顺序：
1. Imports
2. Widget 类定义
3. State 类定义（如果是 StatefulWidget）
4. 生命周期方法（initState, dispose 等）
5. 事件处理方法（_onXxx）
6. 工具方法（_buildXxx, _validateXxx）
7. build 方法
8. 私有组件类（_MyPrivateWidget）

#### 命名规范：
- 文件名：`snake_case.dart`
- 类名：`PascalCase`
- 变量/方法：`camelCase`
- 私有成员：`_leadingUnderscore`
- 常量：`SCREAMING_SNAKE_CASE`（在 AppConstants 中）

### 11. 依赖注入规范

```dart
// 在 main.dart 中初始化
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

### 12. 导航规范

```dart
// 使用 navigation_provider 管理底部导航
final navigationProvider = StateProvider<int>((ref) => 0);

// 页面跳转
Navigator.of(context).push(
  MaterialPageRoute(builder: (context) => DetailPage()),
);

// 弹窗关闭并传递结果
Navigator.of(context).pop(result);
```

## 代码示例模板

### 新建页面模板：
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../constants/app_constants.dart';

class MyPage extends ConsumerStatefulWidget {
  const MyPage({super.key});

  @override
  ConsumerState<MyPage> createState() => _MyPageState();
}

class _MyPageState extends ConsumerState<MyPage> {
  @override
  void initState() {
    super.initState();
    // 初始化逻辑
  }

  @override
  void dispose() {
    // 清理资源
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('页面标题'),
      ),
      body: Container(
        color: AppConstants.backgroundColor,
        child: const Center(
          child: Text('内容'),
        ),
      ),
    );
  }
}
```

### 新建 Provider 模板：
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:freezed_annotation/freezed_annotation.dart';
import '../src/bindings/signals/my_signal.dart';

part 'my_provider.freezed.dart';

@freezed
class MyState with _$MyState {
  const factory MyState({
    @Default(MyStatus.initial) MyStatus status,
    String? data,
    String? errorMessage,
  }) = _MyState;
}

enum MyStatus { initial, loading, success, error }

class MyNotifier extends StateNotifier<MyState> {
  MyNotifier() : super(const MyState()) {
    _listenToRustSignals();
  }

  void _listenToRustSignals() {
    MyResp.rustSignalStream.listen((rustSignal) {
      // 处理响应
    });
  }

  Future<void> doSomething() async {
    state = state.copyWith(status: MyStatus.loading);
    MyReq().sendSignalToRust();
  }
}

final myProvider = StateNotifierProvider.autoDispose<MyNotifier, MyState>(
  (ref) => MyNotifier(),
);
```

## 常见问题

### Q: 什么时候使用 rinf 生成的模型，什么时候自定义模型？
A: 
- **使用 Rinf 生成**：所有与 Rust 通信的数据结构
- **自定义模型**：仅 UI 内部使用的临时数据结构

### Q: Provider 什么时候使用 autoDispose？
A: 
- 页面级状态：使用 `autoDispose`
- 全局共享状态：不使用 `autoDispose`

### Q: 如何处理 BigInt？
A:
```dart
// Int → BigInt
BigInt.from(intValue)

// BigInt → Int
bigIntValue.toInt()

// 显示
Text('${bigIntValue.toString()}')
```


---
alwaysApply: false
keywords: ["@rust", "rust", "rinf", "signal", "api"]
---

# Rust API 开发指南

## Rinf 信号定义规范

### 1. 信号结构定义

```rust
// filepath: native/hub/src/service/my_service.rs

use rinf::{DartSignal, RustSignal};
use serde::{Deserialize, Serialize};

// 请求信号（Dart → Rust）
#[derive(Deserialize, DartSignal, Debug)]
pub struct MyActionReq {
    pub user_id: u64,
    pub param1: String,
    pub param2: f64,
}

// 响应信号（Rust → Dart）
#[derive(Serialize, RustSignal, Debug)]
pub struct MyActionResp {
    pub success: bool,
    pub data: MyData,
}
```

### 2. 服务层实现

```rust
impl<B, T> MySvc<B, T>
where
    B: MyRepo,
    T: OtherRepo,
{
    pub async fn handle_my_action(&self) {
        let recv = MyActionReq::get_dart_signal_receiver();
        
        while let Some(signal_pack) = recv.recv().await {
            let req = signal_pack.message;
            info!("Received MyActionReq: {:?}", req);

            match self.biz.do_something(req.user_id, req.param1, req.param2).await {
                Ok(data) => {
                    MyActionResp {
                        success: true,
                        data,
                    }.send_signal_to_dart();
                }
                Err(e) => {
                    error!("Failed to process: {:?}", e);
                    // 发送错误响应
                }
            }
        }
    }
}
```

### 3. 运行 rinf gen

```bash
# 每次修改 Rust 信号定义后运行
rinf gen
```

## Flutter 调用 Rust API 完整流程

### 步骤 1：定义 Rust 信号（在 `native/hub/src/service/`）

### 步骤 2：运行 `rinf gen` 生成 Dart 绑定

### 步骤 3：创建 Flutter Provider

### 步骤 4：在 UI 中使用

## 注意事项

1. **不要手动修改** `lib/src/bindings/` 下的文件
2. **类型映射**：
   - Rust `u64` → Dart `BigInt`
   - Rust `f64` → Dart `double`
   - Rust `String` → Dart `String`
3. **错误处理**：Rust 端必须捕获所有错误，避免崩溃
4. **日志记录**：使用 `tracing::info!` 和 `tracing::error!`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BeWhyZ) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
