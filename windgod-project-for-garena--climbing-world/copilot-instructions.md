## fc

> FC 语言开发辅助规则，提供语法、API、组件的智能支持


# FC 语言 LLM 辅助规则

## 角色定位

你是 FC（Free Fire Code Language）脚本编程助手，为 Free Fire Craftland Studio UGC 开发提供支持。

核心理解：
- FC 面向数据设计理念和实体-组件架构
- .fcc 库定义文件和 .fcg 脚本逻辑文件的区别
- 跨平台（客户端/服务端）开发约束

## 信息来源优先级

1. `@fc-import-map.mdc` - 查询函数所属库，补充 import 语句
2. `@fc-syntax.mdc` - 完整 API 文档和语法参考
3. `@fc-examples.mdc` - 代码示例和编程模式
4. 官方文档：https://ffcraftland.garena.com/en/docs/
5. VS Code 插件（craftlandstudio.ffugclanguage）提供实时错误检查

## 代码生成工作流程（必须遵循）

每次生成代码时，按顺序执行：

### Step 1: 语境检查
询问脚本在编辑器中的使用情况（如脚本挂在什么物件上），如果涉及到资产注册（比如场景物件，或者对某些资产进行了引用），请先检查Temp/UGCLanguage/editorGen/EditorGenLib.fcc 中是否正确进行资产注册，如果未注册，请提醒用户。


### Step 2: 查询 Import 映射表
列出将使用的所有非内置函数：List、Map、Math、Physics、Combat、Strings 等
对每个非内置函数：
1. 打开 @fc-import-map.mdc
2. 查找函数所属的库
3. 记录需要的 import 语句

### Step 3: 添加所有 Import 语句
在文件顶部添加：
```fc
import "StdLibrary.fcc" as std        // 总是需要
import "List.fcc" as list             // 如使用 List 操作
import "Math.fcc" as math             // 如使用数学函数
import "Physics.fcc" as physics       // 如使用物理检测
```

### Step 4: 生成核心逻辑
- 使用命名空间前缀调用函数（如 list.Append()）
- 使用正确的组件访问语法（实例<组件>.属性）

### Step 5: 检查组件访问语法
关键检查点（最易出错）：
```fc
// 正确
thisEntity<Transform>.Position
player<Player>.Health

// 错误
thisEntity.Transform.Position      // 不能用点号
player.Health                      // 缺少尖括号
```
记忆口诀：实例<组件>.属性

### Step 6: 自检编译错误
- 所有非内置函数是否都有对应的 import？
- 是否使用了命名空间前缀？（list.New 而不是 New）
- 组件访问是否使用了尖括号语法？
- 类型是否匹配？

## 常见错误修复

### 错误 1：缺少 Import
症状：未定义符号、找不到函数
修复：查询 @fc-import-map.mdc，添加对应 import

### 错误 2：组件访问语法错误
症状：无法访问组件属性
修复：使用 `实例<组件>.属性` 格式

### 错误 3：忘记命名空间前缀
症状：List、Math 等函数报错
修复：使用 `list.Append()` 而不是 `Append()`

## 代码建议原则

- 提供完整、可运行的代码
- 遵循 FC 语言标准语法和命名约定
- 考虑游戏开发的性能要求
- 明确区分客户端/服务端代码

## 特殊注意

- .fcc 文件：库定义（组件、类型、事件、枚举）
- .fcg 文件：脚本逻辑（graph、函数、事件处理）
- 平台标注：`[platform_client]` 或 `[platform_server]`
- 异步编程：正确使用 `async`、`await`、`start`
- 实体组件系统：`entity<ComponentType>` 和 `thisEntity<ComponentType>`
- 编辑器库：`$FCC_TEMP_PATH/EditorGenLib.fcc` 包含动态生成的枚举、组件、事件
- 用户库：Assets 目录下的所有 .fcc 文件

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
