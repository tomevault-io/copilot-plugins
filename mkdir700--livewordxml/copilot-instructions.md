## livewordxml

> 本项目采用服务导向架构，将不同功能分离到独立的服务类中。

# 服务架构模式

本项目采用服务导向架构，将不同功能分离到独立的服务类中。

## 核心服务

### DocumentService
- [DocumentService.cs](mdc:src/LiveWordXml.Wpf/Services/DocumentService.cs) - 主要文档服务
- [DocumentService_new.cs](mdc:src/LiveWordXml.Wpf/Services/DocumentService_new.cs) - 新版文档服务
- 负责Word文档的加载、解析和XML提取
- 使用 `DocumentFormat.OpenXML` 库处理.docx文件

### TextMatchingService
- [TextMatchingService.cs](mdc:src/LiveWordXml.Wpf/Services/TextMatchingService.cs) - 文本匹配服务
- 负责将用户输入的文本与Word文档中的XML元素进行匹配
- 实现智能文本查找和定位算法

### ClipboardService
- [ClipboardService.cs](mdc:src/LiveWordXml.Wpf/Services/ClipboardService.cs) - 剪贴板服务
- 管理系统剪贴板操作
- 自动复制提取的XML到剪贴板

### NotificationService
- [NotificationService.cs](mdc:src/LiveWordXml.Wpf/Services/NotificationService.cs) - 通知服务
- 提供系统托盘通知功能
- 用户反馈和状态提示

## 服务设计原则

### 单一职责
- 每个服务专注于一个特定功能领域
- 保持服务接口简洁清晰

### 依赖注入
- 服务通过构造函数注入依赖
- 使用接口定义服务契约
- 便于单元测试和模拟

### 异步操作
- 文件I/O和长时间运行的操作使用异步方法
- 使用 `async/await` 模式
- 避免阻塞UI线程

## 服务使用模式

```csharp
// 在ViewModel中使用服务
public class MainViewModel : ObservableObject
{
    private readonly IDocumentService _documentService;
    private readonly IClipboardService _clipboardService;
    
    public MainViewModel(IDocumentService documentService, IClipboardService clipboardService)
    {
        _documentService = documentService;
        _clipboardService = clipboardService;
    }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mkdir700) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
