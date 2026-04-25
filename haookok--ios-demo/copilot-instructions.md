## ios-demo

> Swift 代码规范，确保学习过程中养成良好的编码习惯


# Swift 代码规范（初学者版）

## 命名规则

```swift
// ✅ 正确：驼峰命名，含义清晰
let userName = "小明"
func calculateTotal(price: Double, quantity: Int) -> Double

// ❌ 错误：含义不清、不规范
let a = "小明"
func calc(p: Double, q: Int) -> Double
```

## 代码注释

初学者阶段，**每个新概念**都必须添加中文注释：

```swift
// ✅ 好的注释：解释为什么
// 使用 guard 提前退出，避免深层嵌套
guard let name = optionalName else { return }

// ❌ 坏的注释：重复代码本身
// 声明一个变量
let name = "小明"
```

## 文件组织

```
项目名/
├── Models/          // 数据模型（像是蓝图）
├── Views/           // 界面（用户看到的东西）
├── ViewModels/      // 视图逻辑（连接数据和界面的桥梁）
├── Services/        // 服务（和外部世界打交道）
├── Extensions/      // 扩展（给已有类型加新本领）
└── Resources/       // 资源文件（图片、颜色等）
```

## SwiftUI 视图规范

```swift
// ✅ 视图结构清晰，使用 MARK 分区
struct ContentView: View {
    // MARK: - 属性
    @State private var count = 0

    // MARK: - 视图
    var body: some View {
        VStack {
            Text("计数: \(count)")
            Button("点击") { count += 1 }
        }
    }
}
```

## 错误处理

```swift
// ✅ 使用 do-catch，给出有意义的错误信息
do {
    let data = try loadFile(named: "config")
} catch {
    print("无法加载配置文件：\(error.localizedDescription)")
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Haookok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
