## serial-helper

> > 行为准则源自 Andrej Karpathy 对 LLM 编码痛点的观察。四条铁律在最前面，技术规范紧随其后。

# 串口调试助手 — 项目规则

> 行为准则源自 Andrej Karpathy 对 LLM 编码痛点的观察。四条铁律在最前面，技术规范紧随其后。

## 行为准则（高于一切）

### 1. 动手前先想清楚
**禁止假设。禁止隐藏困惑。暴露所有权衡。**

- 不确定时**显式声明假设**，宁可问清楚也不要猜
- 存在歧义时**列出所有可能的解读**，不要默默选一个
- 有更简单的方案时**直接说出来**，哪怕和当前方向不同
- 发现自己困惑时**停下来**，指出哪里不清楚，请求澄清

### 2. 简洁优先
**刚好解决问题的代码量。不写任何推测性代码。**

- 没有被要求的 features 不写
- 只为一次调用写的抽象不建
- 没被要求的"灵活性"或"可配置性"不加
- 不可能发生的场景不加错误处理
- 200 行能解决的问题不写 1000 行
- **自我审查：** 写完问自己——一个 senior 工程师会不会觉得这里复杂过头了？会就改。

### 3. 精准修改
**只改必须改的。只清理自己的垃圾。**

改已有代码时：
- 不改相邻代码的风格、注释、格式
- 不重构没坏的东西
- 匹配已有风格，哪怕不是你喜欢的
- 发现无关的死代码，**提一嘴**——但不动手删

清理时：
- 只删除**你的改动**导致不再使用的 import/变量/函数
- 不动原有代码中的死代码，除非被要求

**测试标准：** 每一行改动都能直接追溯到用户的需求。

### 4. 目标驱动
**定义成功标准。循环直到验证通过。**

| 用户说…… | 转化为…… |
|----------|---------|
| "加个校验" | "先写非法输入的测试，跑通了再实现" |
| "修这个 bug" | "先写一个能复现的测试，跑挂了再修" |
| "重构 X" | "确保重构前后测试全部通过" |

多步任务的计划格式：
```
1. [步骤] → 验证: [检查点]
2. [步骤] → 验证: [检查点]
3. [步骤] → 验证: [检查点]
```

强成功标准让 Agent 能自主循环。弱标准（"把它弄好"）只会让你反复澄清。

---

## 技术栈

| 项 | 选型 |
|---|------|
| 语言 | C++17 |
| 框架 | Qt 6.11.0 |
| 编译器 | MSVC 2022 64-bit (主力) / MinGW 13.1.0 64-bit (备选) |
| 构建 | CMake 3.28+ |
| 串口 | QSerialPort (Qt 模块) |
| 图表 | QChart (实时波形) — QPainter 手动绘制作为高性能备选 |
| 脚本 | QJSEngine (JS 引擎，用于宏/脚本) |
| 配置存储 | JSON (QJsonDocument + QSaveFile，跟随 exe 目录) |
| 主题 | QSS 暗色主题，扁平化现代风格 |
| 测试 | GoogleTest |
| 打包 | windeployqt → NSIS/Inno Setup |

## 目录结构

```
project/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   ├── app/
│   │   ├── Application.h/cpp       # QApplication 子类，全局初始化
│   │   └── ThemeManager.h/cpp      # 暗色主题 QSS 管理
│   ├── core/
│   │   ├── SerialPort.h/cpp        # QSerialPort 封装，串口线程
│   │   ├── ProtocolParser.h/cpp    # 协议解析器基类 + 注册工厂
│   │   ├── MacroEngine.h/cpp       # QJSEngine 封装，脚本执行
│   │   └── DataLogger.h/cpp        # 数据日志读写
│   ├── ui/
│   │   ├── MainWindow.h/cpp        # 主窗口
│   │   ├── TabManager.h/cpp        # 多串口 Tab 管理 (QTabWidget)
│   │   ├── SessionView.h/cpp       # 单个串口会话视图
│   │   ├── ReceiveView.h/cpp       # 接收区 (QPlainTextEdit)
│   │   ├── SendView.h/cpp          # 发送区 + 快捷按钮列表
│   │   ├── WaveformView.h/cpp      # 波形绘制窗口
│   │   ├── ProtocolView.h/cpp      # 协议解析面板
│   │   ├── MacroEditView.h/cpp     # 脚本编辑器
│   │   └── StatusBar.h/cpp         # 状态栏统计
│   └── util/
│       ├── HexUtils.h/cpp          # HEX/ASCII 互转工具
│       └── Constants.h             # 全局常量
├── res/
│   ├── theme/
│   │   ├── dark.qss
│   │   └── light.qss
│   └── icons/
├── scripts/                          # 用户 JS 脚本目录
│   └── example.js                    # 示例脚本
├── tests/
│   ├── CMakeLists.txt
│   ├── test_hex_utils.cpp
│   ├── test_protocol_parser.cpp
│   └── test_serial_port.cpp
└── docs/
    └── architecture.md
```

## 文件约束

- 每文件 ≤ 800 行，超过则拆分
- 每函数 ≤ 50 行，超过则提取
- 嵌套 ≤ 4 层，超过则 early return
- 头文件用 `#pragma once`
- 禁止在头文件中 `using namespace`

## 编码风格

### 花括号与缩进

```cpp
// Allman 风格 — 花括号独立一行
void foo()
{
    if (condition)
    {
        doSomething();
    }
    else
    {
        doOther();
    }
}
```

- 缩进：4 空格，禁止 Tab
- 行尾不留空白
- 文件末尾保留一个空行

### include 顺序

```cpp
// 1. 自身头文件
#include "SerialPort.h"

// 2. Qt 头文件
#include <QSerialPort>
#include <QTimer>

// 3. 项目内部
#include "core/DataLogger.h"
#include "util/HexUtils.h"

// 4. 标准库
#include <memory>
#include <vector>
```

每组之间空一行。用 `""` 还是 `<>` 按 Qt 惯例。

### 类声明模板

```cpp
#pragma once

#include <QObject>
#include <QSerialPort>

class SerialPort : public QObject
{
    Q_OBJECT

public:
    explicit SerialPort(QObject *parent = nullptr);
    ~SerialPort() override;

    // 禁止拷贝
    SerialPort(const SerialPort &) = delete;
    SerialPort &operator=(const SerialPort &) = delete;

    bool openPort(const QString &name, qint32 baudRate, QSerialPort::DataBits dataBits,
                 QSerialPort::StopBits stopBits, QSerialPort::Parity parity,
                 QSerialPort::FlowControl flowControl);
    void closePort();
    bool isOpen() const;

    qint64 sendData(const QByteArray &data);

signals:
    void dataReceived(const ReceivedFrame &frame);
    void errorOccurred(const QString &message);
    void portOpened();
    void portClosed();

private slots:
    void onReadyRead();
    void onErrorOccurred(QSerialPort::SerialPortError error);

private:
    QSerialPort *m_serialPort = nullptr;
    QByteArray m_readBuffer;
};
```

要点：
- 成员变量 `m_` 前缀，指针成员在声明处初始化为 `nullptr`
- `explicit` 标记单参数构造函数
- 禁止拷贝（Qt 对象本来就不可拷贝，显式 delete 更清晰）
- signals/public/private 分区明确
- 析构函数 `override` 而非 `virtual`（C++17）

## 命名约定

| 类型 | 风格 | 示例 |
|------|------|------|
| 类/结构体 | PascalCase | `SerialPort`, `ReceiveView` |
| 函数/方法 | camelCase | `openPort()`, `sendData()` |
| 成员变量 | camelCase + `m_` 前缀 | `m_serialPort`, `m_receiveBuffer` |
| 静态/全局常量 | UPPER_SNAKE_CASE | `MAX_BUFFER_SIZE`, `DEFAULT_BAUD_RATE` |
| 局部变量 | camelCase | `portName`, `baudRate` |
| bool 变量 | `is`/`has`/`can`/`should` 前缀 | `isConnected`, `canSend`, `hasData` |
| 信号 | 动词过去式 | `dataReceived()`, `portOpened()` |
| 槽 | 动词原形 | `onSendClicked()`, `handleData()` |

## 信号槽规则

- 禁止用 `SIGNAL()`/`SLOT()` 宏（字符串匹配，编译期不检查）
- 一律用 `connect(ptr, &Class::signal, ptr, &Class::slot)` 新式语法
- 跨线程对象用 `connect` 时必须传 `Qt::QueuedConnection`
- 高频率信号（接收数据）用 `Qt::QueuedConnection` 避免阻塞串口线程
- lambda 连接中捕获 `this` 确保对象生命周期长于连接

## 串口线程模型

```
┌──────────────┐    信号槽    ┌──────────────┐
│  主线程 (UI)  │◄────────────│  串口工作线程   │
│              │  dataReady() │              │
│  MainWindow  │─────────────►│  SerialWorker │
│  ReceiveView │  sendData()  │              │
│  SendView    │              │  QSerialPort  │
└──────────────┘              └──────────────┘
```

- **串口 I/O 跑在独立 `QThread`**，不阻塞 UI
- 工作线程通过 `moveToThread()` 移入，信号槽自动跨线程排队
- **接收算法（借鉴 llcom）：**
  ```
  readyRead 信号 → sleep(50ms) 等缓冲区累积
    → 循环 readAll() 直到 bytesAvailable() == 0
    → emit dataReceived(batch)
  ```
  不使用固定阈值（"1KB 或 50ms"），而是"每次 readyRead 后等待 50ms，然后一次性读空缓冲区"。这样低速设备不会延迟太久，高速设备也能批量接收
- 主线程只负责 UI 更新，不做任何阻塞 I/O
- 关闭串口时先 `close()` 再 `thread()->quit()` 再 `wait()`

### QSerialPort 生命周期管理

**全量销毁重建模式（借鉴 llcom 的 `refreshSerialDevice`）：**

- **打开串口：** 每次 `openPort()` 都 `new QSerialPort`，设置参数，`open()`
- **关闭串口：** `close()` + `deleteLater()`，不留旧实例
- **重连时：** 销毁旧 `QSerialPort`，创建新实例再打开

**原因：** Windows 串口句柄在异常断开后可能残留，复用旧 `QSerialPort` 会导致 `ObjectDisposedException` 等效问题。全量重建是验证过的安全做法。

### 内存看门狗（借鉴 SuperCom）

- 定时器每 30 秒检查 `PrivateMemorySize64`
- 超过 **1 GB** 阈值时：找到文本最长的 `ReceiveView`，销毁重建其内部的 `QPlainTextEdit`，让 OS 回收累积的内存
- 同时保留"超过 10 万行裁剪前 3 万行"作为常态措施
- 状态栏显示当前内存用量（可选）

### 串口异常断开与自动重连

**触发条件：** 串口工作线程检测到 `QSerialPort::ResourceError`（USB 拔出、设备掉电、驱动崩溃）

**关键标记位 `m_userClosed`（`std::atomic<bool>`）：** 区分"用户手动关闭"（不重连）和"设备异常断开"（自动重连）。用户点击"关闭串口"按钮时 `m_userClosed = true`，异常断开时 `m_userClosed = false`。只有 `!m_userClosed` 时启动重连循环。使用 `std::atomic<bool>` 因为 UI 线程写入、串口工作线程读取，跨线程无锁访问。

**处理流程：**

```
ResourceError
    │
    ├─ 1. 立即 close() 释放底层句柄，避免残留占用
    ├─ 2. 停止发送队列，清空待发数据
    ├─ 3. emit portDisconnected()（携带断开原因字符串）
    ├─ 4. 进入重连循环（独立 timer，不阻塞主线程）
    │       │
    │       ├─ 重连间隔:  首次 1s，每次翻倍，最大 30s
    │       │             1s → 2s → 4s → 8s → 16s → 30s → 30s...
    │       │
    │       ├─ 最大重试:  无限重试，直到用户手动关闭
    │       │
    │       ├─ 重连成功:  emit portRecovered()
    │       │             恢复断连前的串口设置（波特率等不变）
    │       │
    │       └─ 用户取消:  点"关闭串口"按钮 → 停止重连 → 回到就绪状态
    │
    └─ 5. UI 反馈
            ├─ 状态栏指示灯变红，文字显示"已断开 (COM3)"
            ├─ 状态栏提示持续显示，不自动消失
            ├─ 接收区最后一行追加 "——— 串口已断开 13:42:01 ———"（灰色分隔线）
            ├─ 发送按钮灰掉，快捷键发送无效
            └─ 重连成功时指示灯变绿，"已连接 (COM3)"，追加 "——— 串口已恢复 13:42:15 ———"
```

**设计原则：**
- 不弹对话框打断用户（断开是物理事件，不是软件错误，不该用模态弹窗）
- 所有状态变化在状态栏即可感知
- 自动重连是默认行为，用户可手动停止

## 错误处理约定

```cpp
// core 层：通过信号报告错误，不弹对话框
emit errorOccurred(tr("无法打开串口 %1: %2").arg(name, port->errorString()));

// ui 层：接收错误信号，通过 QMessageBox 或状态栏展示
// 状态栏错误信息带红色（持续 5 秒后消失）：

// 返回值为 bool 的方法：成功返回 true，失败返回 false + emit errorOccurred
// 返回 QByteArray 的方法：成功返回数据，失败返回空 + emit errorOccurred
// 返回 optional 的方法：用 std::optional<T>
```

分层处理：
| 层 | 怎么处理 |
|----|---------|
| `util/` | 纯函数，返回错误值，不抛异常 |
| `core/` | 信号报告错误，必要时返回 `std::optional` |
| `ui/` | 接收错误信号，弹 QMessageBox 或状态栏提醒 |

## 配置持久化

- 使用 **JSON 文件**（非 INI），存储在 exe 同目录 `ComAssistant.json`
- **Eager-save 策略（借鉴 llcom）：** 每次设置变更立刻写盘，不等到关闭才保存。好处：突然崩溃不丢配置
- 用 `QJsonDocument` + `QJsonObject` 读写，Configuration 单例管理
- 写盘用 `QSaveFile`（先写临时文件再 rename），保证原子性，写一半崩溃不会损坏配置

保存项：
| 分类 | 内容 |
|------|------|
| 串口设置 | 上次使用的端口、波特率、数据位、停止位、校验、流控 |
| 窗口状态 | 位置、大小、最大化状态、分割器比例 |
| 用户偏好 | 主题、HEX/ASCII 默认、时间戳开关、换行符选择 |
| 快捷发送 | 按钮名称和对应数据列表（嵌套数组） |
| 发送历史 | 最近 50 条发送记录 |

**原子写盘实现：**

```cpp
void Configuration::save()
{
    const QString tmpPath = m_filePath + ".tmp";
    QSaveFile file(tmpPath);
    if (file.open(QIODevice::WriteOnly)) {
        file.write(m_doc.toJson(QJsonDocument::Indented));
        file.commit();  // rename tmp → real
    }
}
```

## 数据流定义

core 层和 ui 层之间通过信号槽传递的数据结构：

```cpp
// 接收的每一帧
struct ReceivedFrame {
    QByteArray rawData;       // 原始字节
    QDateTime timestamp;      // 接收时间
    bool isHex;               // 展示模式
};

// 发送的每一帧
struct SendFrame {
    QByteArray rawData;
    QDateTime timestamp;
    bool isHex;
};
```

- core 层传出 `ReceivedFrame`，ui 层自行格式化展示
- 十六进制显示格式：`"48 65 6C 6C 6F"`（大写、字节间空格）
- ASCII 显示：不可打印字符显示为 `.`
- 时间戳格式：`"HH:mm:ss.zzz"`（ms 精度），可选开启

### 数据日志轮转策略

`DataLogger` 负责将接收到的数据持续写入磁盘。

| 配置项 | 值 | 说明 |
|--------|-----|------|
| 单个日志上限 | 50 MB | 超过自动轮转 |
| 轮转文件数 | 10 个 | `log.0.txt` ~ `log.9.txt`，循环覆盖 |
| 文件名格式 | `ComAssistant_YYYYMMDD.log.N` | 每天一个主文件，超出大小加 `.0` `.1` 后缀 |
| 总磁盘上限 | 500 MB / 天 | 超过后弹出提示，暂停记录，不静默吃盘 |

**长时间运行策略：**
- 日志写入使用 `QFile` + `QTextStream`，每次 `flush()` 确保掉电不丢数据
- 轮转时在新文件头部写入 `# Started: 2026-05-18 13:42:01`
- 可在设置中关闭日志记录（默认开启）
- 状态栏显示当前日志文件大小和写入状态
- 超过磁盘上限时状态栏变橙色警告，不清空已有日志

## 暗色主题规范

### 色板

| 用途 | 色值 | 说明 |
|------|------|------|
| 主背景 | `#1e1e2e` | 窗口背景、编辑区背景 |
| 次背景 | `#181825` | 面板、工具栏、侧边栏背景 |
| 表面 | `#313244` | 按钮、输入框、下拉框背景 |
| 主文字 | `#cdd6f4` | 正文、标签 |
| 次要文字 | `#a6adc8` | 辅助说明、占位符 |
| 强调色 | `#89b4fa` | 选中态、链接、发送按钮 |
| 错误 | `#f38ba8` | 错误提示、断开连接指示 |
| 成功 | `#a6e3a1` | 连接成功指示、校验通过 |
| 警告 | `#fab387` | 超时、缓冲满 |
| 边框 | `#45475a` | 按钮、输入框、下拉框边框 |
| 悬停/按下 | `#585b70` | 按钮悬停态、按下态背景 |
| HEX高亮 | `#cba6f7` | 接收区 HEX 字节 |
| 发送色 | `#89b4fa` | 发送数据高亮 |
| 接收色 | `#a6e3a1` | 接收数据高亮 |

### 字体

| 用途 | 字体 | 大小 |
|------|------|------|
| 界面文字 | Microsoft YaHei | 9pt |
| HEX/数据区 | `Cascadia Code` → `Consolas` → `Courier New` | 10pt |
| 状态栏 | Microsoft YaHei | 8pt |

### 中文界面要求

- 本项目是中文串口调试软件，所有面向用户可见的窗口标题、菜单、工具栏、按钮、标签、Tab 标题、状态栏、提示信息、对话框标题和 tooltip **必须使用中文**。
- 协议名、数据格式名和行业通用缩写可保留英文或大写缩写，例如 `HEX`、`ASCII`、`CRC8`、`CRC16`、`CRC32`、`UINT16`、`FLOAT`、`RX`、`TX`。
- 配置键、脚本 API、日志文件名、命令行参数和测试内部断言不属于界面文案，可按现有英文命名保留。
- 新增 UI 文案时优先使用简洁中文，不引入国际化翻译文件；本项目只维护中文界面。

### QSS 组件要点

```css
/* QPushButton — 圆角、悬停变色、按下下沉 */
QPushButton {
    background: #313244;
    color: #cdd6f4;
    border: 1px solid #45475a;
    border-radius: 4px;
    padding: 4px 12px;
}
QPushButton:hover { background: #45475a; }
QPushButton:pressed { background: #585b70; }

/* QComboBox — 下拉箭头定制 */
QComboBox {
    background: #313244;
    color: #cdd6f4;
    border: 1px solid #45475a;
    border-radius: 4px;
    padding: 4px 8px;
}

/* QLineEdit — 聚焦时边框变色 */
QLineEdit {
    background: #313244;
    color: #cdd6f4;
    border: 1px solid #45475a;
    border-radius: 4px;
}
QLineEdit:focus { border-color: #89b4fa; }

/* QPlainTextEdit — 接收区专用 */
QPlainTextEdit {
    background: #1e1e2e;
    color: #cdd6f4;
    border: 1px solid #313244;
    font-family: "Cascadia Code", "Consolas", "Courier New", monospace;
    font-size: 10pt;
}

/* QGroupBox — 分组框 */
QGroupBox {
    border: 1px solid #45475a;
    border-radius: 4px;
    margin-top: 8px;
    padding-top: 8px;
    color: #cdd6f4;
}
QGroupBox::title {
    subcontrol-origin: margin;
    left: 12px;
    padding: 0 4px;
    color: #a6adc8;
}

/* QCheckBox / QRadioButton */
QCheckBox, QRadioButton {
    color: #cdd6f4;
    spacing: 6px;
}
QCheckBox::indicator, QRadioButton::indicator {
    width: 16px;
    height: 16px;
    border: 1px solid #45475a;
    background: #313244;
}
QCheckBox::indicator:checked, QRadioButton::indicator:checked {
    background: #89b4fa;
    border-color: #89b4fa;
}

/* QSpinBox — 数值输入 */
QSpinBox {
    background: #313244;
    color: #cdd6f4;
    border: 1px solid #45475a;
    border-radius: 4px;
    padding: 4px 8px;
}
QSpinBox:focus { border-color: #89b4fa; }

/* QTabWidget — 标签页 */
QTabWidget::pane {
    border: 1px solid #45475a;
    background: #1e1e2e;
}
QTabBar::tab {
    background: #313244;
    color: #a6adc8;
    border: 1px solid #45475a;
    padding: 6px 16px;
    margin-right: 2px;
}
QTabBar::tab:selected {
    background: #1e1e2e;
    color: #cdd6f4;
    border-bottom-color: #1e1e2e;
}
QTabBar::tab:hover:!selected {
    background: #45475a;
    color: #cdd6f4;
}

/* QScrollBar — 滚动条 */
QScrollBar:vertical {
    background: #181825;
    width: 10px;
    border: none;
}
QScrollBar::handle:vertical {
    background: #45475a;
    border-radius: 5px;
    min-height: 30px;
}
QScrollBar::handle:vertical:hover { background: #585b70; }
QScrollBar::add-line:vertical, QScrollBar::sub-line:vertical { height: 0; }
QScrollBar:horizontal {
    background: #181825;
    height: 10px;
    border: none;
}
QScrollBar::handle:horizontal {
    background: #45475a;
    border-radius: 5px;
    min-width: 30px;
}
QScrollBar::handle:horizontal:hover { background: #585b70; }
QScrollBar::add-line:horizontal, QScrollBar::sub-line:horizontal { width: 0; }

/* QToolBar */
QToolBar {
    background: #181825;
    border-bottom: 1px solid #45475a;
    spacing: 4px;
    padding: 2px;
}

/* QStatusBar */
QStatusBar {
    background: #181825;
    color: #a6adc8;
    border-top: 1px solid #45475a;
    font-size: 8pt;
}

/* QMenu */
QMenu {
    background: #313244;
    color: #cdd6f4;
    border: 1px solid #45475a;
    padding: 4px 0;
}
QMenu::item {
    padding: 6px 24px;
    margin: 0 4px;
    border-radius: 4px;
}
QMenu::item:selected {
    background: #45475a;
}
QMenu::separator {
    height: 1px;
    background: #45475a;
    margin: 4px 8px;
}
```

### 主题切换

- `ThemeManager` 单例负责加载和管理 QSS
- 默认暗色主题，后期可加亮色
- QSS 文件放在 `res/theme/`，编译时通过 Qt Resource System 嵌入

## 键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+Enter` | 发送当前数据 |
| `Ctrl+L` | 清空接收区 |
| `Ctrl+S` | 保存接收区内容到文件 |
| `Ctrl+H` | 切换 HEX/ASCII 显示 |
| `Ctrl+T` | 切换时间戳显示 |
| `Ctrl+P` | 打开/关闭串口 |
| `Ctrl+1~9` | 触发快捷发送按钮 1~9 |
| `Escape` | 关闭当前弹窗/面板 |

## 架构原则

### 核心层 (core/) 不依赖 UI 层 (ui/)
- `SerialPort` 纯逻辑，不持有任何 Widget
- `ProtocolParser` 只做数据解析，不画 UI
- 数据通过信号槽驱动 UI 更新

### 每个 Tab 独立
- `TabManager` 管理多个 `SessionView`
- 每个 `SessionView` 持有独立的 `SerialPort`、`ReceiveView`、`SendView`
- 关闭 Tab 时正确释放串口线程资源

### 接收区性能
- 用 `QPlainTextEdit`，不用 `QTextEdit`（前者不做富文本布局，大量数据时不卡）
- 超过 10 万行自动裁剪前 3 万行
- HEX 转换在独立线程完成，避免 UI 卡顿
- 超高频数据（>1000 包/秒）自动切换到摘要模式：
  - 摘要模式不逐包渲染，改为一秒刷新一次统计面板
  - 统计面板显示：最近 1 秒收包数、字节数、平均包大小、峰值速率
  - ASCII 模式下显示最近 5 行摘要文本（截取前 80 字符），HEX 模式显示最近 5 帧数据（截取前 40 字节）
  - 用户可以手动切回普通模式，但 >2000 包/秒时强制锁定摘要模式
  - 低于 500 包/秒持续 3 秒后自动恢复普通模式

## Phase 规划与验收标准

### Phase 1 — 核心收发骨架
**内容：** CMake 骨架 + MainWindow 框架 + 暗色主题 + 串口基本收发 + HEX/ASCII 切换

**验收标准：**
- [ ] CMake 配置正确，MSVC 和 MinGW 下均 `cmake --build` 成功
- [ ] 窗口可拖拽，最小尺寸 800×600，记忆上次关闭时的大小和位置
- [ ] 暗色主题完整应用，所有控件风格一致
- [ ] 扫描系统可用串口，下拉框可切换
- [ ] 波特率/数据位/停止位/校验/流控六个设置项联动生效
- [ ] 打开一对虚拟串口（例如 com0com 创建的 CNCA0→CNCB0），CNCB0 发 100 字节 → CNCA0 收到 100 字节，内容一致
- [ ] HEX 模式显示 `"48 65 6C 6C 6F"`，ASCII 模式显示 `"Hello"`
- [ ] 数据区超过 1 万行不卡顿
- [ ] 关闭串口时线程正确退出，无残留

### Phase 2 — 发送增强
**内容：** 定时发送 + 快捷发送列表 + 发送历史 + 日志保存

**验收标准：**
- [ ] 定时发送间隔从 50ms 到 60s 可调（最小 50ms，因为 Windows 非实时系统 10ms 精度不可靠）
- [ ] 定时发送期间 UI 不卡顿
- [ ] 快捷发送按钮可自定义名称和数据，支持 HEX/ASCII 模式
- [ ] 快捷键 `Ctrl+1~9` 触发对应快捷按钮
- [ ] 发送历史保留最近 50 条，可双击复用
- [ ] 接收区内容可保存为 `.txt` 和 `.hex` 文件
- [ ] 关闭再打开，快捷按钮和设置不丢失

### Phase 3 — 波形绘制
**内容：** QChart 实时波形，多通道，可配置

**验收标准：**
- [ ] 单通道支持 16 条独立曲线
- [ ] 支持 UINT8/UINT16/UINT32/INT8/INT16/INT32/FLOAT 共 7 种数据类型解析
- [ ] 支持大端/小端切换
- [ ] X 轴显示采样点数，Y 轴自动缩放
- [ ] 鼠标可拖拽平移、滚轮缩放
- [ ] 数据速率 100 包/秒下曲线不撕裂
- [ ] **性能基线：** 1000 数据点/秒持续刷新不掉帧（帧率 ≥ 30 FPS）。若 QChart 不达标，降级到 QPainter 直接绑 `QOpenGLWidget` 绘制，不允许用第三方图表库
- [ ] 场景切换有过渡动画，保留 5 个预设场景

### Phase 4 — 协议解析 + 脚本
**内容：** 协议解析框架 + 脚本/宏引擎

**验收标准：**
- [ ] 支持自定义帧头、帧尾、长度域、CRC 校验（CRC8/CRC16/CRC32）
- [ ] 解析结果分窗显示：有效帧 / 错误帧 / 原始数据
- [ ] 有效帧高亮，错误帧红色标记
- [ ] QJSEngine 可执行 JS 脚本，提供 `serial.send()` / `serial.onReceive()` 等 API
- [ ] 脚本可设置定时器和条件触发
- [ ] 脚本语法错误时显示行号和错误信息

### 脚本引擎安全边界

QJSEngine 直接嵌入在进程内，用户脚本有 full access 风险。必须做以下限制：

**执行保护：**

| 防护项 | 策略 |
|--------|------|
| 超时保护 | 单次 `evaluate()` 执行 3 秒超时，超时后 `abortEvaluation()` 强制终止。定时回调类脚本每次回调独立计时 |
| 无限循环 | `while(true){}` 类死循环由超时保护兜底，但 UI 线程在 3 秒内会冻结。因此脚本永远在 **独立 `QThread`** 中执行，不跑主线程 |
| 崩溃隔离 | JS 异常（`throw`、语法错误）被 `try/catch` 包裹，emit `scriptError()` 信号，不影响主程序。Qt 层 C++ 崩溃无法被 JS try/catch 捕获，因此脚本线程 sandbox 只暴露白名单 API |

**API 白名单（参照 llcom 的 Lua 接口设计）：**

```
全局对象:  serial, logger, timer, sys, plot

┌── 串口操作 ──────────────────────────────────────────
│ serial.send(data)          // 发送数据 (QByteArray)
│ serial.sendHex(hexStr)     // HEX 格式发送
│ serial.isOpen()            // 查询连接状态
│ serial.onReceive(cb)       // 注册接收回调 cb(QByteArray)
│
├── 日志输出 ──────────────────────────────────────────
│ logger.trace(tag, msg)     // 六级日志，tag 为模块名
│ logger.debug(tag, msg)
│ logger.info(tag, msg)
│ logger.warn(tag, msg)
│ logger.error(tag, msg)
│ logger.fatal(tag, msg)
│ console.log(msg)           // 等同于 logger.info('script', msg)
│
├── 定时器 ────────────────────────────────────────────
│ timer.once(ms, cb)         // 单次定时器，返回 timerId
│ timer.loop(ms, cb)         // 循环定时器，返回 timerId
│ timer.stop(id)             // 停止定时器
│ timer.isActive(id)         // 查询定时器是否运行
│
├── 协程调度 (移植自合宙 Luat 架构) ─────────────────
│ sys.wait(ms)               // 协程延时，只能在 task 内使用
│ sys.taskInit(fun, ...)     // 创建协程任务，args 为 fun 参数
│ sys.waitUntil(id, timeout) // 条件等待，超时返回 false
│ sys.publish(id, ...)       // 发布消息到内部消息队列
│ sys.subscribe(id, cb)      // 订阅消息
│ sys.unsubscribe(id, cb)    // 取消订阅
│
└── 波形绘制 ──────────────────────────────────────────
  plot.addPoint(value, lineIndex)  // 添加数据点到曲线 (lineIndex: 0~15)
  plot.clear(lineIndex)            // 清空指定曲线 (-1 清空全部)
```

**协程调度示例（QJSEngine 用 event loop + 递归调用模拟）：**

```js
// 创建任务：收到串口数据后自动回复
sys.taskInit(function() {
    while (true) {
        var ok, data = sys.waitUntil("UART_RX", 30000);
        if (ok) {
            logger.info("task", "收到: " + data);
            serial.send("ok!\r\n");
        }
    }
});

// 循环定时器：每秒打印一次
sys.taskInit(function() {
    while (true) {
        sys.wait(1000);
        logger.info("heartbeat", "1s tick");
    }
});
```

**禁止暴露的 API：**
- 不做 `QFile` 绑定（脚本不应操作文件系统）
- 不做 `QProcess` / `QTcpSocket` 绑定（脚本不应访问网络）
- 不做 `QSettings` 绑定（脚本不应修改配置）
- 不暴露 C++ 对象指针，只暴露 QObject 信号槽包装
- 脚本文件路径锁定在 `scripts/` 目录内，禁止 `..` 路径穿越

### Phase 5 — 多串口 + 打磨
**内容：** 多串口 Tab + UI 细节 + 打包

**验收标准：**
- [ ] 可同时打开 4 个以上串口，各自独立收发
- [ ] Tab 可拖拽重排、可分离为独立窗口
- [ ] 状态栏实时统计收发字节数、速率 (KB/s)
- [ ] 所有按钮有 tooltip
- [ ] `windeployqt` 打包后可在无 Qt 环境的 Windows 上直接运行
- [ ] NSIS/Inno Setup 安装包可安装卸载

## 开发流程

1. **先写测试** — 核心工具类（HexUtils、ProtocolParser）必须 TDD
2. **编译通过再继续** — 不允许 commit broken build
3. **写完立刻审查** — 用 cpp-review agent
4. **提交前检查** — 无硬编码路径、无 `qDebug()` 残留、无注释掉的代码
5. **串口测试** — 用 com0com 虚拟串口对测收发

## Git 分支策略与提交规范

### 分支模型

```
main          ← 稳定可发布
  │
  ├─ develop           ← 日常开发主分支
  │     │
  │     ├─ feat/xxx    ← 新功能分支（从 develop 拉）
  │     ├─ fix/xxx     ← bug 修复分支
  │     └─ refactor/xxx← 重构分支
  │
  └─ release/x.x.x    ← 发布前冻结分支
```

- `main` 只接受来自 `develop` 或 `release/*` 的 PR 合并
- 单人开发时可以简化：直接在 `develop` 上工作，Phase 完成后合并到 `main`
- 多人协作时每人从 `develop` 拉 `feat/xxx`，PR 合并回去

### 提交信息格式

```
<type>: <中文简述>

<可选详细说明>
```

| type | 用途 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: 添加十六进制收发切换` |
| `fix` | bug 修复 | `fix: 修复串口热拔出时崩溃` |
| `refactor` | 重构（不改变行为） | `refactor: 提取串口线程到独立 Worker 类` |
| `docs` | 文档 | `docs: 补充串口异常断开策略到 AGENTS.md` |
| `test` | 测试 | `test: 添加 HexUtils 单元测试` |
| `chore` | 构建/工具/配置 | `chore: 配置 MSVC 编译选项` |
| `style` | 格式/代码风格 | `style: 统一缩进为 4 空格` |

**规则：**
- type 用小写英文，冒号后一个空格，简述用中文
- 简述不超过 50 字
- 不要写 `feat: 修改了一些文件` 这种废话——说明改了**什么**、**为什么**
- 一个 commit 只做一件事

### 提交前检查清单

- [ ] `cmake --build` 通过（MSVC Debug + Release）
- [ ] 无 `qDebug()` / `std::cout` 调试输出残留
- [ ] 无注释掉的旧代码
- [ ] 无硬编码路径（用 `QStandardPaths` 或相对路径）
- [ ] 新增文件已加入 `CMakeLists.txt` 的 `add_executable` / `target_sources`

## CI/CD

### 自动构建（GitHub Actions / 本地脚本）

如后续多人协作，建议配 GitHub Actions：

| 触发条件 | 动作 |
|----------|------|
| push 到 `develop` / `feat/*` | `cmake --build` MSVC Debug |
| PR 合并到 `main` | `cmake --build` MSVC Release + MinGW Release + GoogleTest |
| 打 tag `v*.*.*` | Release build + `windeployqt` + 打包 .exe + 上传 Artifact |

即使没有 GitHub Actions，本地也必须验证：
```bash
cmake -S . -B build/msvc    -G "Visual Studio 17 2025" -A x64
cmake --build build/msvc --config Debug
cmake --build build/msvc --config Release
```

### 版本号

```
主版本.次版本.修订号
  0  .   1  .   0
```

- Phase 1 完成后打 `v0.1.0`
- 每个 Phase 完成打次版本号
- bug fix 打修订号
- `main` 分支上每个 commit 都应该是可发布的

## 不做什么

- 不做网络调试（TCP/UDP/HID），只做串口
- 不做国际化，只做中文
- 不引入第三方串口库（QSerialPort 足够）
- 不用 qmake，只用 CMake
- 不写 Doxygen 风格注释（代码命名本身就是文档）
- 不画 UML 图（架构文档 = 目录结构 + 数据流图）
- 不用 `QThread::run()` 旧模式 — 一律 `moveToThread()` + worker object（QSerialPortHelper 的 `run()` + `exec()` 写法已过时，且收发各开一个串口不合理）

## 参考项目

| 项目 | 语言 | 核心参考价值 |
|------|------|-------------|
| `example/QSerialPortHelper` | Qt5/6 C++ | **CMake 模板**（Qt5/Qt6 双兼容）、`QSerialPortInfo` 填充下拉框套路 |
| `example/llcom` | C# WPF | **串口读取算法**（readyRead→sleep→读空）、**SerialPort 全量销毁重建**、**eager-save JSON 配置**、脚本协程调度 API、双显示模式、延迟启动 |
| `example/SuperCom` | C# WPF | **内存看门狗**、**com0com CLI 集成**、语法高亮规则引擎、关闭超时保护 |
| `ComAssistant_MSVC/` | Qt5 C++ | 功能对照（纸飞机完整版），看 .ini 配置即知所有功能和默认值 |

**关键移植细节：**

| 借鉴 | 来源 | Qt 实现方案 |
|------|------|------------|
| 串口读取 | llcom Uart.ReadTask | `QSerialPort::readyRead` → `QTimer::singleShot(50ms)` → 循环 `readAll()` 直到 `bytesAvailable()==0` |
| 串口生命周期 | llcom refreshSerialDevice | 每次 `openPort()` 都 `new QSerialPort`，`closePort()` 都 `close()+deleteLater()` |
| Eager-save 配置 | llcom Settings | 每个 setter 调 `Configuration::save()`，`QSaveFile` 原子写 JSON |
| 内存看门狗 | SuperCom MemoryDog | `QTimer` 30s 检测，超 1GB 重建 `QPlainTextEdit` |
| com0com 集成 | SuperCom VirtualPort | `QProcess` 调 `setupc.exe list/install/remove/change` |
| 双显示模式 | llcom DataShowPage | `PacketMode`（分帧 ListView）vs `StreamMode`（连续追加） |
| 延迟启动 | llcom MainWindow | 构造只做 `setupUi()`，`QTimer::singleShot(0, this, &MainWindow::init)` |
| 关闭超时 | SuperCom AsyncClosePort | `QFutureWatcher` + `QTimer` 超时强制 `deleteLater()` |

**移植指南：**
- llcom 的 `sys.taskInit` / `sys.wait` / `sys.publish` / `sys.subscribe` 这套协程调度用 QJSEngine + `QMetaObject::invokeMethod` + `QTimer` 实现
- 协程 `sys.wait()` 本质是 JS 函数返回 `Promise` 或 yield，QJSEngine 不支持 yield。替代方案：将协程编译为状态机，每步由 QTimer 驱动
- 如果协程实现复杂度 > 300 行，Phase 4 降级为"定时器 + 回调"模式，不做协程
- QSerialPortHelper 的代码**只读不抄**（有空指针 bug、收发分离设计不合理）

---
> Source: [agentthink/serial-helper](https://github.com/agentthink/serial-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
