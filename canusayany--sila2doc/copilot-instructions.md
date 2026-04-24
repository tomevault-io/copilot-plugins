## sila2doc

> Generator文件夹是受保护的尽量不要修改



# SiLA2 D3驱动生成工具 - 项目文档

Generator文件夹是受保护的尽量不要修改
---

## 目录

1. [项目概述](#一项目概述)
2. [核心功能](#二核心功能)
3. [UI设计](#三ui设计)
4. [数据模型](#四数据模型)
5. [关键技术点](#五关键技术点)
6. [实施历史与关键改进](#六实施历史与关键改进)
7. [测试覆盖](#七测试覆盖)
8. [使用指南](#八使用指南)

---

## 一、项目概述

### 1.1 项目定位

本工具是"**一站式设备驱动生成工具**"的SiLA2/gRPC协议模块，用于从SiLA2服务器或本地特性文件自动生成符合D3系统规范的驱动代码。

### 1.2 核心架构

```
┌────────────────────────────────────────────────────────────┐
│  SiLA2 特性源                                               │
│  • 在线服务器（mDNS扫描）                                    │
│  • 本地 .sila.xml 文件                                      │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  Tecan Generator (第三方工具)                               │
│  • 生成接口定义 (Interface)                                 │
│  • 生成客户端实现 (Client)                                  │
│  • 生成数据传输对象 (DTOs)                                  │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  代码分析服务 (ClientCodeAnalyzer)                          │
│  • 反射分析生成的代码                                       │
│  • 提取方法、参数、返回值信息                               │
│  • 提取XML文档注释                                          │
│  • 检测数据类型支持情况                                     │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  方法预览与特性调整 (用户交互)                              │
│  • 用户勾选"调度方法" (MethodOperations)                    │
│  • 用户勾选"维护方法" (MethodMaintenance)                   │
│  • 提供批量操作功能                                         │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  D3驱动代码生成 (CodeDOM)                                   │
│  • AllSila2Client.cs - 多特性整合、方法平铺                │
│  • D3Driver.cs - 驱动主类、方法分类、JSON转换               │
│  • Sila2Base.cs - 基类实现                                  │
│  • CommunicationPars.cs - IP/Port配置                       │
└─────────────────┬──────────────────────────────────────────┘
                  ↓
┌────────────────────────────────────────────────────────────┐
│  项目编译                                                   │
│  • 生成 .csproj 和 .sln 文件                                │
│  • 复制依赖库 (reflib)                                      │
│  • 执行 dotnet build                                        │
│  • 生成 DLL 和 XML 文档                                     │
└────────────────────────────────────────────────────────────┘
```

### 1.3 依赖工具

| 工具 | 用途 | 来源 |
|------|------|------|
| **Tecan Generator** | 从.sila.xml生成C# gRPC客户端代码 | 第三方 (Tecan) |
| **BR.PC.Device.Sila2Discovery** | SiLA2服务器的mDNS扫描和连接管理 | Bioyond |
| **Microsoft CodeDOM** | 生成D3驱动封装层代码 | .NET Framework |
| **Newtonsoft.Json** | JSON序列化/反序列化 | NuGet |
| **.NET SDK 8.0** | 项目编译和DLL生成 | Microsoft |

---

## 二、核心功能

### 2.1 完整工作流程

1. **特性选择**
   - 扫描在线服务器或导入本地.sila.xml文件
   - 支持多特性选择（限同一服务器）
   - 父节点三态显示（未选/半选/全选）

2. **设备信息输入**
   - 品牌（Brand）
   - 型号（Model）
   - 设备类型（DeviceType）
   - 开发者（Developer）

3. **客户端代码生成**
   - 使用Tecan Generator生成强类型代码
   - 自动复制必需的DLL（Tecan.Sila2、gRPC、protobuf等）
   - 自动去重重复的类型定义

4. **方法预览与分类**
   - 显示所有检测到的方法
   - 用户勾选"调度方法"和"维护方法"
   - 支持批量操作（全部设为调度/维护、清除特性）

5. **D3驱动代码生成**
   - 自动生成命名空间：`BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}`
   - 生成4个核心文件：`AllSila2Client.cs`、`D3Driver.cs`、`Sila2Base.cs`、`CommunicationPars.cs`
   - 自动处理不支持的数据类型（JSON序列化/反序列化）

6. **项目编译**
   - 生成.csproj和.sln文件
   - 配置XML文档生成
   - 执行dotnet build
   - 输出DLL和XML文档文件

### 2.2 关键特性

#### 2.2.1 灵活的方法标记系统

- **IsIncluded**：是否包含在D3Driver中（默认true）
- **IsOperations**：是否为调度方法（默认false）
- **IsMaintenance**：是否为维护方法（默认根据方法名判断）

方法可以：
- ✅ 同时是调度方法和维护方法
- ✅ 只是调度方法
- ✅ 只是维护方法
- ✅ 两者都不是（不会生成到D3Driver中）

#### 2.2.2 智能类型转换

**支持的类型**（直接使用）：
- 基础类型：int, byte, sbyte, string, DateTime, double, float, bool, byte[], long, short, ushort, uint, ulong, decimal, char
- 枚举类型
- 基础类型的数组和列表

**不支持的类型**（自动转换为JsonString）：
- 复杂的自定义类型
- 嵌套的复合类型
- Tecan.Sila2.DynamicClient.DynamicObjectProperty

**转换示例**：
```csharp
// AllSila2Client.cs - 保持原样
public virtual SetAnyTypeValueResponse SetAnyTypeValue(
    DynamicObjectProperty anyTypeValue) { ... }

// D3Driver.cs - 自动转换
[MethodOperations]
public virtual string SetAnyTypeValue(string anyTypeValueJsonString)
{
    var anyTypeValue = JsonConvert.DeserializeObject<DynamicObjectProperty>(anyTypeValueJsonString);
    var result = this._sila2Device.SetAnyTypeValue(anyTypeValue);
    return JsonConvert.SerializeObject(result);
}
```

#### 2.2.3 代码去重机制

Tecan Generator在生成多个特性时可能产生重复的类型定义，系统会自动：
- 解析所有生成的C#文件
- 识别重复的顶层类型（类、枚举、结构、接口、委托）
- 注释掉重复的定义，保留第一次出现的
- 不影响嵌套类型和不同命名空间的同名类型

#### 2.2.4 DTO构造函数特性

自动为所有DTO类的默认构造函数添加 `[JsonConstructor]` 特性：
```csharp
[JsonConstructor()]
public SetAnyTypeValueRequestDto() { }
```

---

## 三、UI设计

### 3.1 主界面布局（三列式）

```
┌─────────────────────────────────────────────────────────────┐
│  [⋮] 侧边栏                │ │  主区域                       │
│  ┌──────────────────────┐  │ │  ┌─────────────────────────┐│
│  │ 🔍 扫描 📁 添加 📤 导出│  │ │  │ 生成过程信息             ││
│  ├──────────────────────┤  │ │  │ [日志显示区域]           ││
│  │ 在线服务器 (250px)    │  │ │  └─────────────────────────┘│
│  │ ☑ Server 1          │  │ │  ┌─────────────────────────┐│
│  │   ☐ Feature A       │  │ │  │ 项目信息                 ││
│  │   ☐ Feature B       │  │ │  │ 项目：XXX_YYY           ││
│  │ ☐ Server 2          │  │ │  │ 路径：C:\...            ││
│  ├──────────────────────┤  │ │  │ 状态：已编译             ││
│  │ 本地特性 (可变高度)   │  │ │  └─────────────────────────┘│
│  │ ☐ Node1             │  │ │  ┌─────────────────────────┐│
│  │   ☐ file1.sila.xml  │  │ │  │ [按钮区域]               ││
│  └──────────────────────┘  │ │  │ 🗂️打开项目 📦打开DLL    ││
│                             │ │  │ ✨生成D3 🔨编译 🔧调整   ││
│                             │ │  └─────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### 3.2 方法预览与特性调整窗口

```
┌─────────────────────────────────────────────────────────────┐
│  方法预览与特性调整                                    [×]   │
├─────────────────────────────────────────────────────────────┤
│  请勾选需要标记为"调度方法"或"维护方法"的方法              │
├─────────────────────────────────────────────────────────────┤
│  特性名称  │ 方法名称    │ 类型 │ 返回值 │☑调度│☑维护│ 说明│
│  Feature1  │ Method1     │ Cmd  │ void   │ ☐  │ ☑  │ ... │
│  Feature1  │ Method2     │ Cmd  │ int    │ ☑  │ ☐  │ ... │
│  Feature2  │ GetTemp     │ Prop │ double │ ☑  │ ☐  │ ... │
├─────────────────────────────────────────────────────────────┤
│  [全部调度] [全部维护] [清除特性]    [确定] [取消]        │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 操作按钮

| 按钮 | 功能 | 启用条件 |
|------|------|---------|
| 🗂️ 打开项目目录 | 打开生成的项目目录 | 项目已生成 |
| 📦 打开DLL目录 | 打开编译输出的DLL目录 | 项目已编译 |
| ✨ 生成D3项目 | 生成客户端代码和D3驱动代码 | 已选择特性 |
| 🔨 编译D3项目 | 编译已生成的项目 | 项目已生成 |
| 🔧 调整方法特性 | 重新打开方法预览窗口 | 项目已生成 |

---

## 四、数据模型

### 4.1 核心数据结构

#### ClientFeatureInfo
```csharp
public class ClientFeatureInfo
{
    public Type InterfaceType { get; set; }
    public string FeatureName { get; set; }
    public string InterfaceName { get; set; }
    public string ClientName { get; set; }
    public List<MethodGenerationInfo> Methods { get; set; }
}
```

#### MethodGenerationInfo
```csharp
public class MethodGenerationInfo
{
    public string Name { get; set; }
    public Type ReturnType { get; set; }
    public List<ParameterGenerationInfo> Parameters { get; set; }
    public XmlDocumentationInfo XmlDocumentation { get; set; }
    
    // 方法标记
    public bool IsIncluded { get; set; } = true;
    public bool IsOperations { get; set; } = false;
    public bool IsMaintenance { get; set; } = false;
    
    // 类型转换标识
    public bool RequiresJsonReturn { get; set; }
    
    // 其他属性
    public bool IsProperty { get; set; }
    public bool IsObservableCommand { get; set; }
    public string FeatureName { get; set; }
}
```

#### ParameterGenerationInfo
```csharp
public class ParameterGenerationInfo
{
    public string Name { get; set; }
    public Type Type { get; set; }
    public string Description { get; set; }
    public XmlDocumentationInfo XmlDocumentation { get; set; }
    public bool RequiresJsonParameter { get; set; }
}
```

#### D3DriverGenerationConfig
```csharp
public class D3DriverGenerationConfig
{
    public string Brand { get; set; }
    public string Model { get; set; }
    public string DeviceType { get; set; }
    public string Developer { get; set; }
    
    // 自动生成
    public string Namespace { get; set; }  // BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}
    public string OutputPath { get; set; }  // {Temp}/Sila2D3Gen/{Brand}_{Model}_{Timestamp}
    
    public List<ClientFeatureInfo> Features { get; set; }
    
    // 特性来源
    public bool IsOnlineSource { get; set; }
    public string ServerUuid { get; set; }
    public string ServerIp { get; set; }
    public int? ServerPort { get; set; }
    public List<string> LocalFeatureXmlPaths { get; set; }
}
```

### 4.2 DeviceClass特性规则

```csharp
[DeviceClass(brand, model, injectionKey, deviceType, developer)]
public class D3Driver : Sila2Base { }
```

**参数说明**：
- `brand`：品牌（英文、下划线、数字）
- `model`：型号（英文、下划线、数字）
- `injectionKey`：注入键，通常为 `{Brand}{Model}`（无分隔符）
- `deviceType`：设备类型（英文、下划线、数字）
- `developer`：开发者名称

**示例**：
```csharp
[DeviceClass("Bioyond", "MD", "BioyondMD", "Robot", "Name")]
public class D3Driver : Sila2Base { }
```

### 4.3 D3Driver方法生成规范

**同步性要求**（关键约束）：
- ⚠️ 所有D3调用的方法必须是同步的
- ⚠️ 禁止使用 `async/await`
- 可观察命令使用 `command.Response.GetAwaiter().GetResult()` 阻塞等待

**方法特性标记**：
```csharp
// 调度方法
[MethodOperations]
public double GetCurrentTemperature() { ... }

// 维护方法（参数为顺序编号）
[MethodMaintenance(1)]
public bool GetDeviceState() { ... }

[MethodMaintenance(2)]
public void SwitchDeviceState(bool isOn) { ... }

// 同时标记两个特性
[MethodOperations]
[MethodMaintenance(3)]
public void Initialize() { ... }
```

**XML注释要求**：
- 所有带特性标记的方法必须有完整的XML注释
- 注释从Tecan Generator生成的代码中自动提取

---

## 五、关键技术点

### 5.1 三态CheckBox实现

```xaml
<CheckBox IsThreeState="True" 
          IsChecked="{Binding IsPartiallySelected, Mode=TwoWay}"
          Content="{Binding ServerName}"/>
```

**三态逻辑**：
- `null` = 半选（部分子项被选中）
- `false` = 未选（所有子项未选）
- `true` = 全选（所有子项被选）

### 5.2 CheckBox单击响应优化

使用 `DataGridTemplateColumn` 替代 `DataGridCheckBoxColumn`：

```xml
<DataGridTemplateColumn Width="90" Header="调度方法">
    <DataGridTemplateColumn.CellTemplate>
        <DataTemplate>
            <CheckBox 
                IsChecked="{Binding IsOperations, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}"
                HorizontalAlignment="Center"
                VerticalAlignment="Center" />
        </DataTemplate>
    </DataGridTemplateColumn.CellTemplate>
</DataGridTemplateColumn>
```

### 5.3 单服务器选择校验

```csharp
partial void OnIsSelectedChanged(bool value)
{
    if (value)
    {
        var viewModel = GetD3DriverViewModel();
        if (!viewModel.ValidateServerSelection(this))
        {
            // 校验失败，IsSelected已被重置为false
            return;
        }
    }
    
    ParentServer?.UpdateParentSelectionState();
}
```

### 5.4 友好类型名称生成

避免生成带程序集信息的完整限定名称：

```csharp
private string GetFriendlyTypeName(Type type)
{
    if (type.IsGenericType)
    {
        var typeName = type.GetGenericTypeDefinition().FullName;
        var backtickIndex = typeName.IndexOf('`');
        if (backtickIndex > 0)
            typeName = typeName.Substring(0, backtickIndex);
        
        var genericArgs = type.GetGenericArguments();
        var genericArgNames = genericArgs.Select(GetFriendlyTypeName);
        return $"{typeName}<{string.Join(", ", genericArgNames)}>";
    }
    // ...
}
```

**效果对比**：
- ❌ `ICollection`1[[Stream, System.Private.CoreLib, Version=8.0.0.0, ...]]`
- ✅ `System.Collections.Generic.ICollection<System.IO.Stream>`

### 5.5 代码去重策略

只处理顶层类型定义，避免误伤嵌套类型：

```csharp
var classes = root.DescendantNodes().OfType<ClassDeclarationSyntax>()
    .Where(c => c.Parent is NamespaceDeclarationSyntax ||
                c.Parent is FileScopedNamespaceDeclarationSyntax ||
                c.Parent is CompilationUnitSyntax);
```

### 5.6 XML文档生成配置

在.csproj中添加：

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <DocumentationFile>bin\$(Configuration)\$(TargetFramework)\{projectName}.xml</DocumentationFile>
</PropertyGroup>
```

---

## 六、实施历史与关键改进

### 6.1 第一天：核心功能实现（2024-10-23）

**完成内容**：
- ✅ 基础架构搭建（Models、Services、ViewModels）
- ✅ UI实现（三列布局、侧边栏切换、方法预览窗口）
- ✅ 在线服务器扫描和连接
- ✅ 本地特性文件导入和管理
- ✅ Tecan Generator集成
- ✅ ClientCodeAnalyzer实现（动态编译分析）
- ✅ D3驱动代码生成（CodeDOM）
- ✅ 项目编译功能
- ✅ 测试控制台项目（6个自动化测试）

**关键修复**：
- **问题1**：ClientCodeAnalyzer编译错误 `CS0012: 类型"List<>"在未引用的程序集中定义`
  - 解决：添加 `System.Collections` 程序集引用

### 6.2 第二天：方法标记系统重大改进（2024-10-24 上午）

**核心改进**：从单选变为多选标记系统

**之前设计**：
- 只有一个 `IsMaintenance` 布尔字段
- 方法只能是"维护方法"或"调度方法"之一

**改进后**：
- `IsIncluded`：是否包含（默认true）
- `IsOperations`：是否为调度方法（默认false）
- `IsMaintenance`：是否为维护方法（默认根据方法名判断）
- 方法可以同时拥有多个特性或不拥有任何特性

**其他修复**：
- **问题2**：生成的D3项目未复制reflib文件
  - 解决：添加 `FindReflibDirectory` 方法，优先从reflib目录复制DLL
- **问题3**：CheckBox需要点击两次
  - 解决：使用 `DataGridTemplateColumn` 替代 `DataGridCheckBoxColumn`
- **问题4**：DataGrid有空行
  - 解决：设置 `CanUserAddRows="False"`
- **问题5**：没有特性标记的方法也被生成
  - 解决：过滤条件改为 `m.IsIncluded && (m.IsOperations || m.IsMaintenance)`

### 6.3 第二天下午：项目名称与UI优化（2024-10-24 下午）

**改进内容**：
- ✅ **项目名称与命名空间统一**：使用命名空间作为项目名
  - 之前：`{Brand}{Model}.D3Driver.csproj`
  - 现在：`BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}.csproj`
- ✅ **彻底修复CheckBox点击问题**：改用模板列
- ✅ **简化方法预览界面**：移除冗余的"包含"列
- ✅ **编译找不到项目文件问题**：修正项目文件路径
- ✅ **XML注释文件生成配置**：添加 `<GenerateDocumentationFile>true</GenerateDocumentationFile>`

### 6.4 第二天晚上：在线服务器测试完整修复（2024-10-24 晚上）

**问题**：在线服务器测试失败
- `CS0234`: 命名空间"Tecan.Sila2"中不存在类型或命名空间名"DynamicClient"
- `CS0101/CS0111`: 类型重复定义

**解决方案**：
1. **添加DLL自动复制机制**：
   - 在 `ClientCodeGenerator.cs` 中添加 `CopyRequiredDllsToClientDirectory` 方法
   - 复制所有必需的Tecan和gRPC DLL到GeneratedClient目录
   
2. **实现代码去重功能**：
   - 创建 `GeneratedCodeDeduplicator.cs` 类
   - 使用Roslyn解析生成的C#代码
   - 识别并注释掉重复的顶层类型定义
   - 只处理顶层类型，不影响嵌套类型

3. **补充缺失的DynamicClient.dll**：
   - 在必需DLL列表中添加 `Tecan.Sila2.DynamicClient.dll`

**结果**：
- ✅ 所有7个自动化测试通过
- ✅ 在线服务器测试成功（23个特性、69个文件）

### 6.5 最终优化：代码生成逻辑优化（2024-10-24 下午3点）

**优化目标**：
- `AllSila2Client.cs`：保持原始方法签名，不添加额外JSON参数
- `D3Driver.cs`：对不支持的类型自动转换为JsonString

**实施内容**：

1. **简化 AllSila2ClientGenerator**：
   - 移除额外JSON参数的添加逻辑
   - 移除JSON相关的注释提示
   - 保持方法签名与Tecan Generator生成的代码一致

2. **增强 D3DriverGenerator**：
   - 参数类型转换：`Type param` → `string paramJsonString`
   - 返回类型转换：`ComplexType` → `string`
   - 实现方法体的序列化/反序列化逻辑
   - 添加JSON格式标识的注释

3. **修复类型名称生成问题**：
   - 添加 `GetFriendlyTypeName` 方法
   - 避免生成带程序集信息的完整限定名称
   - 正确处理泛型、嵌套泛型、数组等复杂类型

### 6.6 最后改进：DTO构造函数特性（2024-10-24 下午4点）

**需求**：为生成的DTO类的构造函数添加 `[JsonConstructor]` 特性

**实施**：
- 修改 `Generator/Generators/DtoGenerator.cs`
- 添加 `Newtonsoft.Json` 命名空间导入
- 在默认构造函数上添加 `[JsonConstructor]` 特性

**结果**：
- ✅ 所有DTO类的默认构造函数包含 `[JsonConstructor()]` 特性
- ✅ JSON序列化器可以明确知道使用哪个构造函数

---

## 七、测试覆盖

### 7.1 自动化测试套件

| 测试 | 描述 | 覆盖内容 | 状态 |
|------|------|---------|------|
| 测试1 | 生成D3项目（本地单特性） | 客户端生成、代码分析、D3驱动生成 | ✅ |
| 测试2 | 编译项目 | 项目编译、错误处理 | ✅ |
| 测试3 | 调整方法分类 | 方法特性调整、重新生成 | ✅ |
| 测试4 | 无效文件处理 | 错误处理、友好提示 | ✅ |
| 测试5 | 编译失败处理 | 编译错误解析、错误统计 | ✅ |
| 测试6 | 多特性完整流程（本地） | 多特性整合、命名冲突处理 | ✅ |
| 测试7 | 在线服务器完整流程 | 服务器扫描、在线生成、去重功能 | ✅ |

### 7.2 测试统计

- **总测试数**：7个
- **通过率**：100%
- **测试覆盖**：
  - ✅ 客户端代码生成
  - ✅ 代码分析和反射
  - ✅ D3驱动代码生成
  - ✅ 项目编译
  - ✅ 方法分类调整
  - ✅ 错误处理
  - ✅ 在线服务器集成
  - ✅ 代码去重
  - ✅ DLL自动复制

### 7.3 运行测试

```bash
cd Sila2DriverGen/TestConsole
dotnet run -- --auto  # 运行所有自动化测试
dotnet run            # 交互式测试菜单
```

---

## 八、使用指南

### 8.1 快速开始

#### 方式1：使用本地.sila.xml文件（推荐）

1. 点击"📁 添加本地特性"按钮
2. 选择一个或多个.sila.xml文件
3. 在侧边栏勾选需要的特性
4. 点击"✨ 生成D3项目"按钮
5. 在弹出的对话框中输入设备信息（品牌、型号、类型、开发者）
6. 在方法预览窗口中勾选"调度方法"或"维护方法"
7. 点击"确定"生成项目
8. 点击"🔨 编译D3项目"按钮编译
9. 编译成功后，点击"📦 打开DLL目录"查看输出

#### 方式2：使用在线服务器

1. 点击"🔍 扫描服务器"按钮
2. 等待扫描完成（约3秒）
3. 展开服务器节点，勾选需要的特性（限同一服务器）
4. 后续步骤同"方式1"的第4-9步

### 8.2 高级功能

#### 导出特性文件

1. 勾选需要导出的特性
2. 点击"📤 导出特性"按钮
3. 选择导出目录
4. 系统会将.sila.xml文件复制或下载到目标目录

#### 调整方法特性

1. 在生成项目后，点击"🔧 调整方法特性"按钮
2. 重新配置方法的"调度"或"维护"标记
3. 点击"确定"后，系统会重新生成 `D3Driver.cs` 文件
4. 重新编译项目以应用更改

#### 批量操作

在方法预览窗口中：
- **全部调度**：将所有方法标记为调度方法
- **全部维护**：将所有方法标记为维护方法
- **清除特性**：清除所有方法的特性标记

### 8.3 注意事项

1. **单服务器限制**：只能选择同一服务器的特性，跨服务器选择会自动取消并提示错误

2. **方法命名**：
   - D3调用的方法不能有重载（方法名相同但参数不同）
   - 如有命名冲突，系统会自动添加 `FeatureName_` 前缀

3. **类型支持**：
   - 不支持的复杂类型会自动转换为JSON字符串
   - 调用时需要传递JSON格式的字符串参数

4. **编译要求**：
   - 需要.NET SDK 8.0或更高版本
   - 确保reflib目录包含所有必需的DLL

### 8.4 生成的文件结构

```
BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}/
├── BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}.csproj
├── BR.ECS.DeviceDrivers.{DeviceType}.{Brand}_{Model}.sln
├── GeneratedClient/
│   ├── *Client.cs              # Tecan Generator生成的客户端
│   ├── *Dtos.cs                # 数据传输对象
│   ├── I*.cs                   # 接口定义
│   └── *.dll                   # 依赖库
├── lib/
│   ├── BR.PC.Device.Sila2Discovery.dll
│   ├── BR.ECS.Executor.Device.*.dll
│   └── ...
├── AllSila2Client.cs           # 多特性整合类
├── D3Driver.cs                 # D3驱动主类
├── Sila2Base.cs                # 基类
└── CommunicationPars.cs        # 通信配置
```

### 8.5 常见问题

**Q: 编译失败，提示找不到某个类型？**  
A: 确保reflib目录包含所有必需的DLL，或检查依赖库版本是否匹配。

**Q: 方法预览窗口中没有显示某些方法？**  
A: 检查方法是否是public的，以及返回类型是否被正确识别。

**Q: 在线服务器扫描不到任何设备？**  
A: 确保SiLA2服务器正在运行、在同一网络内，且mDNS服务已启用。

**Q: 生成的代码有重复的类型定义？**  
A: 系统会自动去重，但如果仍有问题，可以手动注释掉重复的定义。

**Q: 如何判断哪些方法应该标记为"调度方法"或"维护方法"？**  
A: 
- **调度方法**：D3调度系统会调用的方法，通常是设备的核心操作
- **维护方法**：设备维护相关的方法，如校准、复位、初始化等
- 系统会根据方法名自动判断（包含 maintenance、calibrate、reset、init、config 的方法默认为维护方法）

---

## 附录：修改的文件清单

### A. 核心服务

| 文件 | 功能 | 关键类/方法 |
|------|------|-----------|
| `Services/ClientCodeGenerator.cs` | 客户端代码生成 | `GenerateClientCode`, `CopyRequiredDllsToClientDirectory` |
| `Services/ClientCodeAnalyzer.cs` | 代码分析 | `Analyze`, `CompileToAssembly`, `IsSupportedType` |
| `Services/D3DriverGeneratorService.cs` | D3驱动生成 | `Generate`, `CompileProjectAsync`, `CopyDependencyLibraries` |
| `Services/GeneratedCodeDeduplicator.cs` | 代码去重 | `DeduplicateGeneratedCode`, `CommentOutDuplicateTypes` |
| `Services/CodeDom/AllSila2ClientGenerator.cs` | AllSila2Client生成 | `GenerateCode` |
| `Services/CodeDom/D3DriverGenerator.cs` | D3Driver生成 | `GenerateCode`, `AddMethodBody`, `GetFriendlyTypeName` |

### B. UI和ViewModel

| 文件 | 功能 |
|------|------|
| `Views/D3DriverView.xaml` | 主界面 |
| `Views/MethodPreviewWindow.xaml` | 方法预览窗口 |
| `ViewModels/D3DriverViewModel.cs` | 主ViewModel |
| `ViewModels/MethodPreviewViewModel.cs` | 方法预览ViewModel |

### C. 数据模型

| 文件 | 功能 |
|------|------|
| `Models/MethodGenerationInfo.cs` | 方法信息模型 |
| `Models/ParameterGenerationInfo.cs` | 参数信息模型 |
| `Models/ClientAnalysisResult.cs` | 分析结果模型 |
| `Models/D3DriverGenerationConfig.cs` | 生成配置模型 |

### D. 测试

| 文件 | 功能 |
|------|------|
| `TestConsole/AutomatedTest.cs` | 自动化测试 |
| `TestConsole/TestRunner.cs` | 交互式测试 |
| `TestConsole/README.md` | 测试说明 |

### E. Tecan Generator

| 文件 | 功能 |
|------|------|
| `Generator/Generators/DtoGenerator.cs` | DTO生成器（添加JsonConstructor特性） |

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/canusayany) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
