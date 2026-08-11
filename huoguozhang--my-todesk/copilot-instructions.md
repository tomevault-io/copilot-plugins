## my-todesk

> 类 ToDesk 的远程桌面控制系统。画面走 WebRTC（P2P），控制指令走 DataChannel，信令走服务端。支持双向连接：Mac↔Mac、Windows→Mac、iOS→Mac 全控制；Mac→iOS 仅投屏 / 远程摄像头（iOS 系统限制，不可反向控制）。

# my-toDesk

类 ToDesk 的远程桌面控制系统。画面走 WebRTC（P2P），控制指令走 DataChannel，信令走服务端。支持双向连接：Mac↔Mac、Windows→Mac、iOS→Mac 全控制；Mac→iOS 仅投屏 / 远程摄像头（iOS 系统限制，不可反向控制）。

## 客户端形态

客户端统一采用 **Flutter 壳 + WebView 内容**方案，频繁更新的业务界面由 H5（`mWeb` / `deskWeb`）承载，不常用更新的或者追求极致性能的，使用flutter。

| 平台 | Flutter 壳 | 共用内容 |
|------|------|------|
| 手机端 | `mApp` | `mWeb` |
| 桌面端 | `deskApp` | `deskWeb` |

## 目录结构

| 目录 | 类型 | 接口前缀 | 说明 |
|------|------|------|------|
| `mApp` | 手机端 App（Flutter）| `/m-app/` | 手机端 Flutter 壳。三 tab + WebView 加载 `mWeb`；提供 WebRTC 媒体、投屏、摄像头 |
| `mWeb` | 手机端 WebView | `/m-web/` | H5 三页面（设备列表 / 远程连接 / 我的），被 `mApp` 加载 |
| `deskApp` | 桌面端 App（Flutter）| `/d-app/` | 桌面端 Flutter 壳。WebView 加载 `deskWeb`；提供屏幕采集、键鼠注入、WebRTC |
| `deskWeb` | 桌面端 WebView | `/d-web/` | H5 页面，被 `deskApp` 加载 |
| `server` | 服务端 | — | Go + GoFrame v2。信令、设备注册、连接码 / 鉴权、会话管理（媒体不经过服务端），按前缀为各端提供接口 |

## 接口约定

服务端按**端类型**划分接口前缀：

| 端 | 目录 | 前缀 |
|------|------|------|
| 手机端 App | `mApp` | `/m-app/` |
| 桌面端 App | `deskApp` | `/d-app/` |
| 手机端 Web | `mWeb` | `/m-web/` |
| 桌面端 Web | `deskWeb` | `/d-web/` |

## 端口分配

| 端 | 目录 | 端口 |
|------|------|------|
| 服务端 | `server` | `8066` |
| 手机端 Web | `mWeb` | `8077` |
| 桌面端 Web | `deskWeb` | `8088` |

> `mApp` / `deskApp` 为客户端壳，不占用固定端口。

## 规则

- 修复问题时，不要瞎猜问题出在哪里，可以增加调试信息，辅助排查问题。，需要通过验证，涉及到界面时，使用e2e。功能是纯逻辑时，使用单元测试验证通过后。才能确定是完成任务。
- 出现bug时，分析根因（如不清楚这块语法或者用法 可以通过安装skill或者 增加约束进本文件里面），形成规则，避免下次再范类似的错误。
- 涉及持久化写入（SharedPreferences、数据库、文件）时，必须先验证前置条件成立（如 channel 调用成功），再写入，不能在不确定状态下写入。
- 开发新功能/使用新技术，参考依据：1 确认本地是否有相关skill有的话，就使用。2 在https://www.skills.sh/中查找相关skill，引导用户安装。

## Skill 参考规范

开发时必须加载对应 skill，以确保代码符合框架最佳实践：

| 场景 | 必须参考的 Skill |
|------|------|
| 开发 `server`（Go） | `goframe-v2`、`golang-pro` |
| 开发 `deskApp` / `mApp`（Flutter） | `flutter-apply-architecture-best-practices` |
| 开发 macOS 原生层（Swift） | `swiftui-pro` |

Skill 加载方式：在开始编写对应代码前，通过 Skill 工具调用对应 skill，参考其规范后再动手。

## 技术选型规则

-  能用社区能力就用社区的，使用前需要询问用户，并且列出star数据和下载量等数据

## 技术细节
- 需要记录授权状态，如第一次拒绝时，下一次应该还走授 
权的逻辑，如无法弹窗也应该文本提醒 


## Flutter 编码规范

- 传给 Flutter 框架的列表，返回类型必须写框架期望的类型，不能写更具体的子类型。例如 `MultiProvider` 的 `providers` 参数期望 `List<SingleChildWidget>`，返回值就声明 `List<SingleChildWidget>`，不能写 `List<ChangeNotifierProvider>`，否则运行时 Provider 找不到正确类型。


## 技术架构

- 跨端功能统一封装，分为路由代码+端侧代码，如何调用摄像头功能方法为 `callCamera` ,使用策略模式，ios则调用ios目录的camera方法，window则调用window目录得camera方法，方法名都一样，只是内部实现不一样。

---
> Source: [huoguozhang/my-toDesk](https://github.com/huoguozhang/my-toDesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
