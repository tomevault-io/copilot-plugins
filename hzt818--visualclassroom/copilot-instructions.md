## visualclassroom

> | 首席架构师 | 统筹全局架构、决策技术选型、协调模块间依赖 | 全局 |

# AGENTS.md —— VisualClassroom AI代理协作手册

## 一、角色分工

| 角色 | 职责 | 负责模块 |
| --- | --- | --- |
| 首席架构师 | 统筹全局架构、决策技术选型、协调模块间依赖 | 全局 |
| Engine-CPP开发官 | C++核心引擎与各学科插件实现 | Engine/、plugins/*Module/ |
| WinUI-Desktop开发官 | Windows端界面、DirectX渲染、WebSocket服务 | VisualClassroom/ |
| Flutter-Mobile开发官 | 移动端遥控器UI、网络通信、引导系统 | MobileRemote/ |
| 白板专员 | WindBoard集成与GoodNotes功能补齐 | WhiteboardModule/ |
| Office专员 | LibreOffice SDK集成与三种文档渲染 | OfficeModule/ |
| UX/设计官 | MD3设计一致性、交互体验、引导系统 | 所有UI层 |
| 集成测试官 | 端到端联调、性能测试、交付验收 | 全栈 |

## 二、开发铁律（违反即打回）

### 2.1 架构铁律

1. **禁止**在移动端（Flutter）导入 dart:ffi、加载 .so/.dll、执行任何科学计算
2. **禁止**从零实现已被成熟方案解决的问题（查表"成熟方案复用清单"）
3. **禁止**硬编码任何学科特定逻辑到 Engine/core/ —— 必须放在 plugins/ 中
4. **禁止**任何操作步骤超过5步

### 2.2 编码铁律

1. 所有文件头必须包含：`// Copyright (c) VisualClassroom. All rights reserved.`
2. C++变量命名：`snake_case`
3. Dart变量命名：`camelCase`
4. XAML命名：`PascalCase`
5. 提交信息格式：`[模块] 操作: 简述`（例：`[PhysicsModule] feat: 添加RK4积分器`）

### 2.3 UI铁律

1. **禁止**使用纯图标按钮（必须配文字标签）
2. **禁止**在没有任何引导的情况下让用户面对空白界面
3. **必须**在首次启动时展示欢迎/引导流程
4. **必须**提供"跳过引导"选项

## 三、第三方依赖管理

### 3.1 依赖引入规范

- C++项目：通过CMake FetchContent引入（禁止手动下载）
- Flutter项目：通过 pubspec.yaml 引入（禁止手动下载）
- WinUI项目：通过NuGet Package Manager引入

### 3.2 当前依赖清单（固定版本）

**C++/CMake依赖：**

```cmake
# 科学计算
FetchContent_Declare(eigen GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git)
# WebSocket
FetchContent_Declare(beast GIT_REPOSITORY https://github.com/boostorg/beast.git)
# 序列化
FetchContent_Declare(msgpack GIT_REPOSITORY https://github.com/msgpack/msgpack-c.git)
# PPT支持
FetchContent_Declare(aspose_slides GIT_REPOSITORY https://github.com/aspose-slides/Aspose.Slides-FOSS-Cpp.git)
```

**Flutter/pub依赖：**

```yaml
dependencies:
  flutter_riverpod: ^2.6.0
  web_socket_channel: ^3.0.0
  msgpack_dart: ^1.0.0
  interactive_3d: ^1.3.0
  sqflite: ^2.3.0
  camera: ^0.10.0
  mDNS: ^1.0.0
```

**WinUI/NuGet依赖：**

```xml
<!-- C# 版主程序（VisualClassroom/） -->
<PackageReference Include="Microsoft.WindowsAppSDK" Version="1.7.250606001" />
<PackageReference Include="Microsoft.Graphics.Win2D" Version="1.3.0" />
<PackageReference Include="MessagePack" Version="2.5.187" />
<!-- legacy C++ 版（legacy/VisualClassroomCpp/） -->
<PackageReference Include="HelixToolkit.WinUI.SharpDX" Version="2.26.0" />
<PackageReference Include="Vortice.WinUI" Version="3.5.0" />
```

## 四、关键审查清单

每次提交PR前，Agent必须自检：

### 4.1 通用检查

- [ ] 是否引入了不必要的第三方依赖？
- [ ] 是否实现了无需实现的"轮子"？
- [ ] 操作步骤是否超过5步？

### 4.2 Windows端检查

- [ ] SwapChainPanel/Viewport3DX 是否在副屏正确全屏？
- [ ] WebSocket服务是否正确监听端口 8899？
- [ ] 插件DLL是否正确加载（路径为 ./plugins/*.dll）？

### 4.3 移动端检查

- [ ] 是否错误导入了 dart:ffi？（禁止）
- [ ] mDNS发现是否正常工作？
- [ ] MessagePack序列化/反序列化是否正确？
- [ ] 滑块调节是否伴随触感反馈（Android需调用 HapticFeedback）？

### 4.4 UI检查

- [ ] 是否严格符合MD3设计令牌？（颜色、圆角、字体）
- [ ] 暗色模式下，卡片背景是否为 #1E1E1E？
- [ ] 所有按钮是否都有Ripple涟漪效果？
- [ ] 所有图标是否都有文字标签？

### 4.5 性能检查

- [ ] 模拟帧率是否达标？（3D≥120FPS，2D≥60FPS）
- [ ] 端到端延迟是否<100ms？
- [ ] 移动端是否实现缩略图降频（1Hz）？

## 五、调试与联调速查

### 5.1 Windows端调试

```bash
# 附加到进程
devenv.exe /DebugExe VisualClassroom.exe

# 关键断点位置
- WebSocketHost.cpp::OnMessage     # 接收指令
- PhysicsModule.cpp::update         # 参数更新
- BoardCanvasControl.cs::OnPointerMoved  # 白板笔触
```

### 5.2 移动端调试

```bash
# 查看WebSocket连接日志
flutter run --verbose

# 查看MessagePack原始数据（在msgpack_helper.dart中）
print(bytesToHex(data));  # 辅助调试
```

### 5.3 联调验证命令

```bash
# 1. 确认PC端WebSocket服务正常
curl -i http://localhost:8899/health

# 2. 模拟移动端发送指令（使用wscat）
wscat -c ws://localhost:8899
> {"cmd":0x02,"payload":{"gravity":1.6}}
```

### 5.4 常见问题排查

| 现象 | 可能原因 | 解决方案 |
| --- | --- | --- |
| 手机扫描不到PC | mDNS未启用/防火墙拦截 | 检查防火墙入站规则，开放8899端口 |
| 滑块延迟>200ms | 计算负载过高 | 检查Eigen是否开启AVX2优化 |
| 白板笔触无压感 | 驱动未安装 | 检查触控笔驱动，WinUI InkCanvas配置 |
| PPT显示空白 | Aspose.Slides库加载失败 | 检查CMake FetchContent是否完整下载 |
| Excel打开卡顿 | 文件过大 | 检查是否开启视口虚拟化渲染 |

## 六、沟通规范

### 6.1 每日同步

- 每个Agent在每天开始时，在对话中声明："今日计划：[模块] + [任务]"
- 遇到阻塞时，立即标注："⚠️ BLOCKED: [原因]"

### 6.2 阶段性汇报模板

```
【Stage X 完成报告】
- 任务：[任务名称]
- 产出：[文件列表]
- 验证：[测试命令/截图说明]
- 风险：[遇到的问题及解决方案]
- 下一步：[后续计划]
```

### 6.3 代码审查要点

- 审查前先检查是否违反"开发铁律"
- 重点审查：性能关键路径、第三方依赖使用、跨线程安全

## 七、最终验收标准（Definition of Done）

### 7.1 功能验收

- [ ] Windows端启动后，3秒内显示欢迎界面
- [ ] 首次使用引导流程完整可走通（教师5步/学生3步）
- [ ] 物理模拟（单摆/粒子轨迹）在大屏正常显示且流畅
- [ ] 白板书写流畅，GoodNotes功能对照表全部打勾
- [ ] PPT/Word/Excel正常打开、翻页、缩放、批注叠加
- [ ] 移动端连接PC并完整遥控所有功能
- [ ] 暗黑模式/亮色模式正常切换

### 7.2 性能验收

- [ ] 3D模拟帧率 ≥ 120 FPS
- [ ] 端到端延迟 < 100ms
- [ ] 移动端连续运行3小时耗电 < 15%
- [ ] 应用冷启动 < 2秒

### 7.3 交付验收

- [ ] Windows: VisualClassroom.exe + plugins/ 文件夹完整打包
- [ ] Android: app-release.apk 可安装运行
- [ ] 用户手册（帮助中心）内容完整
- [ ] 代码仓库结构完整，README.md含构建指南

## 八、核心理念回顾（写在最后）

> "让每一堂课都活起来。"

本项目的成功，取决于以下五个核心支柱的协同：

1. **Windows原生 + C++引擎** —— 把性能压榨到极致
2. **移动端轻量遥控** —— 让教师解放双手
3. **成熟方案复用** —— 站在巨人的肩膀上
4. **全学科全文档** —— 一网打尽，无所不包
5. **以人为本，极简至上** —— 3分钟上手，3秒理解

请每一位参与开发的Agent铭记：我们不是在写代码，我们是在帮助千万教师更好地教书育人。

---

## 附录：快速参考卡

### 端口分配

| 服务 | 端口 |
| --- | --- |
| WebSocket服务 | 8899 |
| Redis缓存 | 6379 |
| Nginx Web | 8080 |

### 关键路径

| 项目 | 路径 |
| --- | --- |
| 引擎插件 | Engine/plugins/*/ |
| 渲染视图 | VisualClassroom/Views/ |
| 移动端功能 | MobileRemote/lib/features/ |
| 通信协议 | Protocol/commands.msgpack |
| 样式定义 | VisualClassroom/MD3Styles.xaml |

### 常用构建命令

| 平台 | 命令 |
| --- | --- |
| Windows | msbuild VisualClassroom.sln /p:Configuration=Release |
| Android | flutter build apk --split-per-abi |
| iOS | flutter build ios --release |
| Web | flutter build web --web-renderer canvaskit |

---
> Source: [hzt818/VisualClassroom](https://github.com/hzt818/VisualClassroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
