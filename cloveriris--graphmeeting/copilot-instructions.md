## graphmeeting

> > **架构更新（2026-02-06）**：从 Flutter + Rust FFI 改为**纯 Flutter 实现**，简化架构，加速开发。

# GraphMeeting | 图会议 - Agent 开发指南

> **架构更新（2026-02-06）**：从 Flutter + Rust FFI 改为**纯 Flutter 实现**，简化架构，加速开发。

> **核心使命**：解决多人会议复杂度爆炸问题，通过"异步非线性 + 三维可视化 + 游戏化协作"重新定义会议体验。

---

## 1. 项目背景与愿景

### 1.1 问题定义
传统会议面临**复杂度爆炸**：
- 线性发言 → 打断、等待、效率低下
- 同时在线 → 时间协调困难
- 信息过载 → 会议纪要无法捕捉思维脉络
- 共识困难 → 谁同意谁反对难以可视化

### 1.2 核心洞察
> "参会者像给播客录音一样各自独白，AI 在同一时刻把所有人独白剪成节点，挂到一张会说话的协作星图上。"

### 1.3 产品愿景
**思维建筑工地 (Mind Construction Site)**：
- 每个参与者是**工蜂般的 Avatar**，在虚空中飞行、放置想法方块
- 回放时看到一座**认知宫殿**拔地而起（类似 Minecraft 延时摄影）
- 最终产出是**电影级视觉叙事**，而非枯燥的会议纪要

---

## 2. 核心设计原则

### 2.1 会议范式革新

| 传统会议 | GraphMeeting |
|---------|-------------|
| "同时在线" | **零同时在线** - 完全异步，系统 7×24 收集语音 |
| "麦克风" | **单向录制通道** - 只能说话，听不到别人，避免打断 |
| "发言顺序" | **无顺序** - 每句话立刻变成节点，并发写入 |
| "回合" | **图版本号** - 每 5 秒生成新图，所有人看最新合并版 |
| "举手" | **节点颜色** - 绿色=共识，红色=争议，一目了然 |

### 2.2 三维数据结构：Chrono-Vine（时序藤蔓）

**坐标系定义**：
- **Y轴（时间↑）**：螺旋柱或直线，正方向向上
- **X轴（参与者环绕）**：每个参与者占据一个轨道，环绕时间柱分布
- **Z轴（语义深度）**：AI 叶子层叠在关键节点上方

```
                    Y (时间 ↑)
                    │
    ┌───────────────┼───────────────┐
    │   用户A: ●──●─┼─●──●          │
    │         │     │  \            │
    │   用户B: ●──●─┼─●   ●──●       │
    │               │       \       │
    │   用户C:      ●──●──●──●──●    │
    │               │               │
    └───────────────┼───────────────┘
                    └────────────────→ X (参与者维度)
                   /
                  Z (语义深度)
                  🍃 AI 叶子层叠
                  📋 Todo 清单
                  🔗 关联推荐
```

**布局策略**：
1. **螺旋柱时间轴**：时间沿中心柱向上延伸
2. **环绕轨道**：参与者像藤蔓一样缠绕在柱子上
3. **分叉检测**：话题转变或回复非直接上级时产生新分支
4. **合并点**：显式引用/同意前人观点时汇聚
5. **AI 叶子**：在高密度、决策点自动生成语义层叠

### 2.3 Avatar 实体化系统

每个参与者是一个**可飞行的 3D Avatar**：
- **Idle**：悬浮观望
- **Flying**：飞行冲刺（像钢铁侠）
- **Working**：在节点上作业（种植/连接/解决争议）
- **Teleporting**：飞走/传送

### 2.4 节点作为建筑方块

| 节点类型 | 视觉表现 | 含义 |
|---------|---------|------|
| VoiceBlock | 发光方块 | 语音内容，大小与时长成正比 |
| ConsensusCrystal | 多面水晶 | 达成共识，多面体折射光线 |
| ContentionRubble | 碎石/荆棘 | 争议待解决，红色脉冲 |
| AIFlower | 花朵/果实 | AI总结与可执行项 |

### 2.5 争议解决游戏化

多人同时靠近争议节点 → 发射"治愈光束" → 进度环填充 → 转化为共识水晶。

**Boss Fight 感**：争议不是争吵，而是协作挑战。

---

## 3. 技术架构（纯 Flutter）

### 3.1 分层架构

```
┌─────────────────────────────────────────────────────────┐
│                    Flutter App                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│ │   UI 层     │  │  3D 引擎    │  │  状态管理        │ │
│ │ (播放控制)   │  │ (Custom     │  │ (Riverpod)       │ │
│ │ (房间列表)   │  │  Painter)   │  │ (本地 + 同步)    │ │
│ └─────────────┘  └─────────────┘  └─────────────────┘ │
│                        │                                │
│              ┌─────────┴─────────┐                      │
│              ▼                   ▼                      │
│        ┌─────────────┐    ┌─────────────┐              │
│        │ Avatar 系统  │    │  网络层     │              │
│        │ (动画 + 物理)│    │ (WebSocket) │              │
│        │             │    │             │              │
│        │ • 飞行动画   │    │ • 状态同步   │              │
│        │ • 轨迹拖尾   │    │ • 冲突解决   │              │
│        │ • 协作光束   │    │ • 离线队列   │              │
│        └─────────────┘    └─────────────┘              │
│                                                        │
│  ┌─────────────────────────────────────────────────┐  │
│ │                  AI 服务层                        │ │
│ │  • 内容分类（关键词规则 / 云端 LLM）               │ │
│ │  • 摘要生成                                      │ │
│ │  • 共识检测                                      │ │
│ └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 3.2 关键技术选型

| 模块 | 技术 | 理由 |
|-----|------|------|
| 跨平台 UI | Flutter | 统一移动端/桌面端/Web |
| 3D 渲染 | CustomPainter | 纯 Dart 实现，无需 FFI，热重载友好 |
| 动画 | Flutter Animation | 内置 Tween + Physics |
| 状态管理 | Riverpod | 响应式，依赖注入 |
| 网络 | WebSocket + 本地存储 | 实时同步 + 离线优先 |
| AI 分析 | 云端 API (OpenAI/Claude) | 无需本地模型，响应快 |
| 音频录制 | 原生平台通道 | macOS AVAudioRecorder |

### 3.3 为什么放弃 Rust FFI？

| 问题 | 说明 |
|-----|------|
| **编译复杂** | flutter_rust_bridge 生成代码复杂，类型冲突频繁 |
| **调试困难** | 跨语言调试链路长，堆栈信息断裂 |
| **热重载失效** | Rust 代码修改需要重新编译，开发效率低 |
| **团队成本** | 需要同时掌握 Flutter + Rust，招人困难 |
| **过度设计** | 项目初期不需要 Rust 的性能优势 |

**替代方案**：Flutter CustomPainter 做 3D 投影 + Dart 实现核心逻辑，性能完全够用。

---

## 4. 核心数据模型（Dart）

### 4.1 节点 (VineNode)

```dart
class VineNode {
  final String id;
  final String messageId;           // 关联原始消息
  final SpaceTimePoint position;    // 时空坐标
  
  // 内容
  final String content;             // 语音转文字
  final String contentPreview;      // 前 100 字摘要
  
  // 节点类型与状态
  final NodeType nodeType;          // message/branch/merge/milestone/aiSummary
  final NodeStatus status;          // draft/committed/confirmed/archived
  
  // 拓扑连接
  final String? parentId;           // 上级节点
  final List<String> branchIds;     // 分叉子节点
  final String? mergeTargetId;      // 合并目标
  
  // AI 生成
  final List<LeafAttachment> leaves;
  
  // 元数据
  final String authorId;
  final DateTime createdAt;
  final Map<String, int> vectorClock;  // CRDT 版本向量
}
```

### 4.2 Avatar

```dart
class Avatar {
  final String id;
  final String name;
  final Color color;
  
  // 3D 状态
  Offset3D position;
  Offset3D velocity;
  double energy;        // 0.0 - 1.0
  
  // 状态机
  AvatarState state;    // idle/flying/working/teleporting
  List<Offset3D> trail; // 飞行轨迹（用于拖尾效果）
  
  // 动画
  AnimationController? flyAnimation;
  Offset3D? flyTarget;
}
```

### 4.3 AI 叶子

```dart
class LeafAttachment {
  final String id;
  final LeafType type;      // summary/actionItems/decision/riskAlert/insight
  final String title;
  final String content;
  final List<TodoItem> todos;
  final double relevanceScore;  // 0.0 - 1.0
  final DateTime generatedAt;
}
```

---

## 5. 核心系统实现

### 5.1 3D 藤蔓引擎（Flutter CustomPainter）

**坐标变换流程**：
```
世界坐标 (x, y, z) 
  → 缩放 (zoom)
  → Y轴旋转 (环绕) 
  → X轴旋转 (俯仰) 
  → 透视投影 
  → 屏幕坐标 (x, y) + 深度 (z) + 缩放因子 (scale)
```

**关键算法**：

| 算法 | 用途 | 位置 |
|-----|------|------|
| **螺旋柱布局** | 时间轴 + 参与者环绕 | `VineLayoutEngine.computeLayout()` |
| **3D 投影** | 透视/等角切换 | `Viewport3D.project()` |
| **Z 深度排序** | 正确遮挡 | 渲染前对节点排序 |
| **贝塞尔曲线** | 藤蔓自然弯曲 | `ChronoVinePainter._drawVine()` |

### 5.2 Avatar 飞行动画

**状态机**：
```
Idle ──[flyTo()]──► Flying ──[到达]──► Idle
                   │
                   └──[靠近节点]──► Working ──[完成]──► Idle
```

**动画曲线**：
- 起飞： easeOutQuad（快速离开）
- 巡航： linear（匀速）
- 降落： easeInOutCubic（平滑减速）

**物理效果**：
- 轨迹拖尾：保留最近 50 帧位置
- 朝向：自动转向飞行方向
- 发光：能量值影响光晕强度

### 5.3 网络同步

**架构**：中心化服务器 + WebSocket

**同步流程**：
1. 客户端 A 创建节点 → 本地立即渲染
2. 发送到服务器 → 进入离线队列（如果断网）
3. 服务器广播 → 其他客户端接收
4. 冲突解决 → 基于 Vector Clock 的最后写入胜出

**消息类型**：
```dart
enum MessageType {
  nodeCreated,
  nodeUpdated,
  nodeDeleted,
  avatarMoved,
  leafGenerated,
  userJoined,
  userLeft,
}
```

### 5.4 AI 分析

**触发时机**：
- 新节点创建后 2 秒（防抖）
- 手动点击"分析"按钮
- 会议结束批量生成

**分析流程**：
```
文本内容 
  → 关键词规则分类（快速）
  → 云端 LLM 摘要（异步）
  → 提取行动项
  → 生成 LeafAttachment
  → 本地渲染
```

---

## 6. 开发规范

### 6.1 代码组织

```
lib/
├── main.dart
├── app.dart
├── core/                      # 核心工具
│   ├── constants.dart
│   ├── theme.dart
│   └── utils.dart
├── models/                    # 数据模型
│   ├── chrono_vine/
│   │   ├── vine_node.dart
│   │   ├── leaf_attachment.dart
│   │   └── space_time_axis.dart
│   └── avatar.dart
├── services/                  # 业务服务
│   ├── audio/
│   │   └── voice_recording.dart
│   ├── chrono_vine/
│   │   ├── vine_layout_engine.dart
│   │   └── topology_analyzer.dart
│   ├── avatar/
│   │   └── avatar_service.dart
│   ├── network/
│   │   └── connection_manager.dart
│   └── ai/
│       └── ai_service.dart
├── state/                     # 状态管理（Riverpod）
│   ├── room_provider.dart
│   ├── viewport_provider.dart
│   └── avatar_provider.dart
├── ui/                        # UI 层
│   ├── screens/
│   │   ├── home_screen.dart
│   │   └── room_screen.dart
│   ├── widgets/
│   │   ├── vine_canvas.dart
│   │   ├── avatar_overlay.dart
│   │   └── chat/
│   └── painters/
│       └── chrono_vine_painter.dart
└── bridge/                    # 原生平台通道
    └── audio_recorder.dart
```

### 6.2 命名规范

- **Dart**: `lowerCamelCase` 变量/函数，`UpperCamelCase` 类，`lowercase_with_underscores` 文件

---

## 7. 里程碑规划

### Phase 1: 核心循环验证 (MVP)
- [x] 单 Avatar 飞行控制
- [x] 语音录制（原生平台通道）
- [ ] 放置 VoiceBlock 节点
- [ ] 简单时间轴回放

### Phase 2: 多人协作
- [ ] WebSocket 连接
- [ ] 多 Avatar 同步显示
- [ ] 节点连接与分叉
- [ ] 争议解决可视化

### Phase 3: AI 增强
- [ ] 云端 AI 摘要
- [ ] 自动生成 Leaf
- [ ] 延时摄影回放

### Phase 4: 商业化
- [ ] 房间管理
- [ ] 导出 Markdown/PDF
- [ ] API 接入（飞书/钉钉）

---

## 8. 给 Agent 的指令模板

```
项目：GraphMeeting - 异步非线性会议工具
架构：纯 Flutter，无 FFI
当前阶段：{Phase 1/2/3/4}

核心概念：
- Avatar：参与者的 3D 化身，Flutter 动画实现飞行
- VineNode：想法的物理表现，CustomPainter 渲染
- Space-Time Vine：时间轴 + 参与者轨道 + 语义层级
- WebSocket：实时状态同步

请生成 {具体模块}，注意：
1. 纯 Dart 实现，不使用 Rust FFI
2. 使用 Riverpod 管理状态
3. CustomPainter 做 3D 投影渲染
4. 考虑离线场景（网络不可用时本地缓存）
```

---

*最后更新：2026-02-06*  
*架构变更：Flutter + Rust FFI → 纯 Flutter*  
*原因：简化架构，加速开发，降低团队成本*

---
> Source: [CloverIris/GraphMeeting](https://github.com/CloverIris/GraphMeeting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
