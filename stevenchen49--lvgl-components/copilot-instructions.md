## lvgl-components

> 因为可能存在多个可选方案，要做出正确的决策，需要足够的依据。

# AI助手核心规则

## 三阶段工作流

### 阶段一：分析问题

**声明格式**：`【分析问题】`

**目的**
因为可能存在多个可选方案，要做出正确的决策，需要足够的依据。

**必须做的事**：
- 理解我的意图，如果有歧义请问我
- 搜索所有相关代码
- 识别问题根因

**主动发现问题**
- 发现重复代码
- 识别不合理的命名
- 发现多余的代码、类
- 发现可能过时的设计
- 发现过于复杂的设计、调用
- 发现不一致的类型定义
- 进一步搜索代码，看是否更大范围内有类似问题

做完以上事项，就可以向我提问了。

**绝对禁止**：
- ❌ 修改任何代码
- ❌ 急于给出解决方案
- ❌ 跳过搜索和理解步骤
- ❌ 不分析就推荐方案

**阶段转换规则**
本阶段你要向我提问。
如果存在多个你无法抉择的方案，要问我，作为提问的一部分。
如果没有需要问我的，则直接进入下一阶段。

### 阶段二：制定方案
**声明格式**：`【制定方案】`

**前置条件**：
- 我明确回答了关键技术决策。

**必须做的事**：
- 列出变更（新增、修改、删除）的文件，简要描述每个文件的变化
- 消除重复逻辑：如果发现重复代码，必须通过复用或抽象来消除
- 确保修改后的代码符合DRY原则和良好的架构设计

如果新发现了向我收集的关键决策，在这个阶段你还可以继续问我，直到没有不明确的问题之后，本阶段结束。
本阶段不允许自动切换到下一阶段。

### 阶段三：执行方案
**声明格式**：`【执行方案】`

**必须做的事**：
- 严格按照选定方案实现
- 修改后运行类型检查

**绝对禁止**：
- ❌ 提交代码（除非用户明确要求）
- 启动开发服务器

如果在这个阶段发现了拿不准的问题，请向我提问。

收到用户消息时，一般从【分析问题】阶段开始，除非用户明确指示。

---

# Components 子模块代码风格规范

## 1. 文件组织

### 1.0 项目根目录结构
- gui
  - iface          # 标准接口 .h，及标准接口的实现 .cpp, forward LVGL 必要声明，但不暴露任何 LVGL 具体定义和函数
    - gui          # 实现 #include "gui/View.h"， 体现这个头文件 gui 相关
      - View.h     # 文件名一般与内部的主要对象同名 
      - Adaptor.h  # 对接 impls 中的 AdaptorLv8.cpp 实现，暴露接口函数
  - impls          # 兼容 LVGL8/LVGL9 的实现
    - lv8          # LVGL8 实现
      - AdaptorLv8.cpp # LVGL8 强相关函数
    - lv9          # LVGL9 实现
     
### 1.1 头文件
- 使用 `#pragma once` 作为头文件保护
- 按顺序包含头文件：项目库，系统库，第三方库
- 使用 `namespace gui` 包装所有 GUI 组件
- 允许为了一大类继续拆分命名空间，比如 `namespace gui::style` 和 `namespace gui::adaptor`
- 放置在 `gui/iface/gui` 目录中
- 尽可能避免暴露过多头文件，只保留必要的头文件，提升编译速度
- 头文件中的复杂实现放到 `gui/iface/gui` 目录的 cpp 文件中

### 1.2 源文件
- 避免太长的代码行长度，一般不超过 120 字符
- 头文件中的复杂实现放到 `gui/iface/gui` 目录中
- LVGL8 相关实现在 `gui/impls/lv8` 目录中
- 保持接口和实现分离

## 2. 命名规范

### 2.1 类名
- 使用 PascalCase：`Button`, `View`, `ViewStack`
- 清晰描述性的名称，比如 `Button` 表示按钮，`View` 表示视图，`VStack` 表示垂直视图堆栈，`HStack` 表示水平视图堆栈，`ZStack` 表示叠视图堆栈
- 对象使用 class 定义，以数据结构为主的用 struct 定义，比如 `struct Size`

### 2.2 方法名
- 使用 camelCase：`backgroundColor()`, `foregroundColor()`, `text()`
- 使用动词前缀：`set`, `get`, `create`, `apply`, `show`, `hide`
- 利用修饰器实现流畅接口，比如 `Modifier` 实现修改方法，使用 `Modifier` 时，返回 `*this`, 注意返回左右值两个版本(Decorator 设计模式)
- 如果遇到要调用 LVGL 函数，使用 `_lv` 前缀，比如 `_lvCreateButton`， 声明在 `gui/iface/gui/Adaptor.h` 中，实现在 `gui/impls/lv8/AdaptorLv8.cpp` 中
- 如果方法内含 lv_obj_t* 对象的操作，则方法名以 `_` 开头，比如 `_setSize`， 提醒用户注意 UI render 非线程安全

### 2.3 变量名
- 成员变量使用 `m` 前缀：`mButtonConfig`, `mButtonText`
- 局部变量使用 camelCase：`buttonConfig`, `parentObj`
- 静态局部变量使用 `s` 前缀：`sButtonConfig`, `sParentObj`
- 静态全局变量使用 `g` 前缀：`gButtonConfig`, `gParentObj`
- 常量使用 google C++ 风格： `kBlack`, `kWhite`

### 2.4 文件名
- 头文件：`Button.h`, `View.h`, `ProgressBar.h`
- 源文件：`Button.cpp`, `View.cpp`, `ProgressBar.cpp`

## 3. 代码格式

### 3.1 缩进
- 使用 4 个空格（不使用 Tab）
- 在 `.editorconfig` 中配置

### 3.2 括号风格
- K&R 风格（开括号在同一行）
- 函数,结构体和类定义换行

### 3.3 空格使用
- 操作符前后加空格：`a + b`
- 逗号后加空格：`func(a, b, c)`
- 关键字后加空格：`if (condition)`

### 3.4 行长度
- 保持不超过 120 字符
- 适当换行长行

## 4. 文档风格

### 4.1 文件头
```cpp
/**
 * Copyright (c) 2025 XXXXXX AI All Rights Reserved.
 */
```

### 4.2 类文档
```cpp
/**
 * Copyright (c) 2025 XXXXXX AI All Rights Reserved.
 */
```

### 4.3 方法文档
```cpp
/**
 * @brief Set button text
 * @param[in] text Button display text
 * @param[out] this Reference to this object for chaining
 * @return Reference to this object for chaining
 */
```

### 4.4 代码块注释
- 使用 `// ==================== Section Title ====================` 格式
- 保持注释简洁，使用英文
- 不在 Doxygen 中写使用例子（按需单独提供）

## 5. C++ 特定规则

### 5.1 类设计
- 使用 RAII 原则
- 提供构造函数和析构函数
- 使用 `override` 关键字标记虚函数
- 支持流畅接口，返回 `*this`
- 支持私有成员变量，使用 `m` 前缀， 使用英文且形象地描述成员变量
- 支持静态成员变量，使用 `s` 前缀
- C++20 标准特性, 但要注意 g++9.3 只支持部分特性

### 5.2 内存管理
- 优先使用智能指针
- 避免裸指针，除了 LVGL 对象（`lv_obj_t*`）
- 在析构函数中清理资源
- 优先使用移动语义，避免拷贝，但是要注意移动导致的双析构
- 注意智能指针和 lambda 导致的循环引用、捕获计数问题、生命周期问题、内存泄露问题

#### 5.2.1 智能指针最佳实践
- **优先使用 unique_ptr**：避免不必要的共享所有权
- **使用 weak_ptr 处理循环引用**：特别是在事件系统中
- **避免在 lambda 中捕获智能指针**：除非确实需要延长生命周期
- **使用 enable_shared_from_this**：当需要从 this 创建 shared_ptr 时

#### 5.2.2 Lambda 和智能指针问题解决方案
```cpp
// ❌ 错误：循环引用
class Button {
    std::shared_ptr<Button> mSelf;
    std::function<void()> mCallback;
    
    void setCallback() {
        mCallback = [this]() { /* 可能导致循环引用 */ };
    }
};

// ✅ 正确：使用 weak_ptr
class Button : public std::enable_shared_from_this<Button> {
    std::function<void()> mCallback;
    
    void setCallback() {
        mCallback = [weakSelf = std::weak_ptr<Button>(shared_from_this())]() {
            if (auto self = weakSelf.lock()) {
                // 安全地使用 self
            }
        };
    }
};

// ✅ 正确：使用原始指针（当生命周期明确时）
class View {
    View* mParent;  // 不管理生命周期
    std::vector<std::function<void()>> mCallbacks;
    
    void addCallback() {
        mCallbacks.push_back([this]() {
            // 确保 View 对象在使用期间有效
        });
    }
};
```

### 5.3 异常处理
- 使用返回值表示错误状态
- 避免异常，使用错误码或布尔返回值，如果不可避免，请使用 `throw` 关键字，并提供详细的错误信息
- 使用 `try-catch` 关键字捕获异常，并提供详细的错误信息

### 5.4 方法组织
```cpp
class ViewBase 
{
public:
    // Constructor
    ViewBase(const std::string& name);
    virtual ~ViewBase();

protected:
    std::shared_ptr<std::atomic<bool>> mDestroyed;
    lv_obj_t* mLvParent;
    lv_obj_t* mLvObj;
    std::string mName;
};

template <typename Derived>
class Modifier 
{
public:
    Modifier() = default;
    virtual ~Modifier() = default;
};

template <typename Derived>
class View : public ViewBase, public Modifier<Derived>
{
public:
    View(const std::string& name = "") : ViewBase(name) {}
    virtual ~View() = default;
};

class Button : public View 
{
public:
    // Constructor
    Button(const std::string& text);
    virtual ~Button();

    // ==================== Button specific methods ====================
    Button& onClick(OnClickCallback callback);

    // ==================== Convenient methods ====================
    Button& hide();

protected:
    // Protected helper methods
    void applyButtonStyle();

protected:
    // Pay attention to the direct access to lv_obj_t*
    lv_obj_t* _build(lv_obj_t* parent) override;

private:
    // Private member variables
    std::string mText;        
};
```

## 6. 配置类

### 6.1 结构设计
```cpp
struct Layout
{
    enum class HorizontalAlign {
        Left,
        Center,
        Right
    };
    
    enum class VerticalAlign {
        Top,
        Center,
        Bottom
    };

    int x;
    int y;
    HorizontalAlign hAlign = HorizontalAlign::Left;
    VerticalAlign vAlign = VerticalAlign::Top;
};
```

### 6.2 配置模式
- 使用结构体进行配置
- 提供默认值
- 包含常用操作的便捷方法
- 支持流畅接口

## 7. LVGL 集成

### 7.1 全局方法
- 使用 `_lv` 前缀：`_lvCreateButton()`, `_lvSetButtonText()`
- 声明存放在 `Adaptor.h` 头文件中, 实现存放在 `impls/lv8/AdaptorLv8.cpp` 中
- 不直接在 iface 头文件中暴露 lvgl.h 头文件

### 7.2 对象管理
- 在成员变量中存储 `lv_obj_t*`
- 需要时延迟创建对象
- 创建对象后应用配置

## 8. 最佳实践

### 8.1 代码组织
- 接口和实现分离
- 使用结构化的 struct 管理参数
- 为常见用例提供工厂方法

### 8.2 错误处理
- 验证参数
- 提供有意义的错误消息
- 使用日志输出调试信息

### 8.3 性能
- 避免不必要的对象拷贝
- 对大对象使用常量引用
- 适当使用 `const` 限定符
- 使用移动语义优化性能
- 避免在热点路径中使用虚函数调用

## 9. 工具配置

### 9.1 EditorConfig
```
[*.{c,h,cpp,hpp}]
indent_style = space
indent_size = 4
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
```

### 9.2 编译器警告
- 启用所有警告：`-Wall -Wextra`
- 将警告视为错误：`-Werror`
- 使用特定警告标志提高代码质量

---
> Source: [stevenchen49/lvgl-components](https://github.com/stevenchen49/lvgl-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
