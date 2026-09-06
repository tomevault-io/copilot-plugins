## blivechat-niconico-rating

> 本文档面向接手本仓库的开发者和编码代理。修改前先阅读相关模块，保持现有行为兼容；README 描述与代码不一致时，以当前代码和实际验证结果为准，并同步修正文档。

# AGENTS.md

本文档面向接手本仓库的开发者和编码代理。修改前先阅读相关模块，保持现有行为兼容；README 描述与代码不一致时，以当前代码和实际验证结果为准，并同步修正文档。

## 项目定位

这是 blivechat 的 Windows 插件，为直播弹幕提供 1～5 级投票、niconico 风格统计、wxPython 管理界面和 OBS 浏览器源页面。

- 插件入口：`main.py`
- 桌面界面与投票状态：`gui.py`
- blivechat 消息接入：`listener.py`
- HTTP 服务与前端数据接口：`web_server.py`
- OBS 页面：`frontend/index.html`
- 内置 blivechat SDK：`blcsdk/`
- 插件元数据：`plugin.json`
- 默认配置：`config.json`
- PyInstaller 配置：`blivechat-niconico-rating.spec`

`result_exporter.py` 是旧的静态 HTML 导出实现，当前主流程没有引用。不要假设它仍是产品功能，也不要在无明确需求时扩展它。

## 运行模型

应用有两个长期线程：

1. wxPython 主线程运行 `VoteApp.MainLoop()`，创建并操作 `VoteFrame`。
2. `NetworkWorker` 在后台线程中运行独立的 `asyncio` 事件循环，初始化 `blcsdk`，接收弹幕，并承载 `aiohttp` Web 服务。

主要数据流如下：

```text
blivechat WebSocket
  -> blcsdk.BaseHandler
  -> listener.VoteHandler
  -> wx.CallAfter(VoteFrame.process_vote_by_level)
  -> VoteFrame 中的投票状态
  -> GUI 主线程发布 web_server 投票快照
  -> GET /api/votes
  -> frontend/index.html 每 500 ms 轮询并渲染
```

线程边界是修改时最重要的约束：

- 后台线程不得直接修改 wx 控件；通过 `wx.CallAfter(...)` 回到 GUI 线程。
- GUI 发起服务启停时，通过 `asyncio.run_coroutine_threadsafe(...)` 投递到网络事件循环。
- `web_server._vote_data` 由锁保护，GUI 主线程通过 `web_server.publish_vote_snapshot()` 主动发布普通数据；HTTP 请求不得读取 GUI 对象。
- 关闭顺序保持为：停止 Web 服务、解绑 listener、关闭 blcsdk。

## 核心业务约束

- 只处理非插件来源的文字弹幕；礼物、舰队和醒目留言不参与投票。
- 投票内容先 `strip()`，再按等级 1 到 5 的正则插入顺序匹配。重叠规则中较低等级优先。
- 正则使用 `pattern.match()`，不是 `fullmatch()`；默认表达式 `^1$`～`^5$` 才是精确匹配。
- 只在统计进行中编译和匹配规则。开始或停止统计后必须调用 `listener.update_vote_patterns()` 刷新缓存。
- 同一 `uid` 只记录第一张有效票。不要把后续投票改成覆盖，除非产品需求明确改变。
- 每次开始统计都会清空 `vote_counts`、`vote_records` 和 `total_votes`。
- 传统模式直接展示实际票数。
- niconico 模式将默认等级人数计算为：

  ```text
  max(初始人数, 实际总票数) - 其他四个等级的实际票数
  ```

  结果不得小于 0。当前 GUI 表格仍展示原始投票，补票计算发生在 `web_server.publish_vote_snapshot()`；若增加其他结果视图或导出路径，必须复用同一口径。
- 非实时模式在统计期间隐藏百分比，停止统计后再显示；实时模式在统计期间显示。

## HTTP 与前端契约

Web 服务不会随应用自动启动，需在 GUI 的“服务启动”区域操作。端口范围为 `1024..65535`，监听 `0.0.0.0`。启动时优先绑定填写的端口；若端口被占用或不可访问，则按距离自动尝试上下各 20 个端口，并将实际端口回写 GUI。页面地址显示为 `http://localhost:<实际端口>`。

路由只有：

- `GET /`：返回 `frontend/index.html`
- `GET /api/votes`：返回投票快照
- `/static/`：提供 `frontend/` 下的静态文件

`/api/votes` 的字段为：`title`、`labels`、`patterns`、`percentages`、`total_votes`、`raw_votes`、`is_counting`、`countdown_seconds`、`include_repo`、`show_total`、`result_visible`。修改字段名、类型或可见性语义时，必须同步修改 Python 端和 Vue 页面。

前端是单文件 Vue 3 页面，优先加载仓库内的 `frontend/vendor/vue.global.prod.js`，CDN 只是兜底。不要引入构建链，除非需求确实需要；当前页面应能由 aiohttp 直接提供。OBS 推荐画布为 900×600，页面内部按 1200×800 等比缩放。

## 配置与宿主环境

`blcsdk.api.init()` 依赖 blivechat 启动插件时注入：

- `BLC_PORT`：blivechat 本地 API 端口
- `BLC_TOKEN`：插件认证令牌

因此直接运行 `python main.py` 通常会因缺少宿主环境或无法连接 blivechat 而失败。这不等同于应用逻辑回归。

`config.json` 位于插件目录，GUI 只在用户点击“保存当前配置”或“加载配置文件”时读写它，不会在启动时自动加载。配置键与 `VoteFrame.save_config()` / `load_config()` 必须同步。配置中可能包含用户偏好，测试保存功能前先备份或使用临时副本，避免覆盖工作区默认配置。

日志写入工作目录下的 `log/niconico-rating.log`，启动时会清空当天主日志，历史日志最多保留 7 份。

## 开发命令

建议使用仓库本地虚拟环境：

```sh
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

最小自动检查：

```sh
python -m compileall -q main.py gui.py listener.py web_server.py result_exporter.py blcsdk
python -m json.tool plugin.json >/dev/null
python -m json.tool config.json >/dev/null
```

GUI 人工检查：

```sh
python test_gui.py
```

`test_gui.py` 只模拟 listener 并打开窗口，不是自动化测试，也不会覆盖真实 blivechat 连接、投票消息、HTTP 服务或 OBS 渲染。涉及这些模块时应按改动范围补充以下验证：

- GUI：控件初始状态、开始/停止倒计时、配置保存/加载、窗口关闭锁。
- listener：统计状态切换、正则优先级、空消息、重复 `uid`。
- Web：服务启动/停止/重启、端口占用错误、`/api/votes` 字段与百分比。
- 前端：结果隐藏/揭晓/实时更新、连接错误、900×600 OBS 视口。
- 集成：在 blivechat 中开启“通过服务器转发消息”，验证断开时插件退出。

当前仓库没有 pytest/unittest 测试套件、lint 配置或 CI。不要声称测试通过，除非明确说明实际执行了哪些检查。

## 打包与发布

Windows 发布包由 PyInstaller spec 生成：

```sh
pyinstaller -y ./blivechat-niconico-rating.spec
```

spec 会打包 `plugin.json`、`LICENSE`、`icon.ico`、`config.json`、前端 HTML 和本地 Vue runtime，并在 `dist/` 中生成 `niconico-rating.zip`。新增运行时资源时必须同步更新 `DATAS`，并检查压缩包内目录层级与 README 的安装说明一致。

发布前同时核对：

- `plugin.json` 的版本、入口 `niconico-rating.exe` 和功能描述。
- `requirements.txt` 与 PyInstaller 兼容性；不要无目的升级固定版本或核心依赖。
- `dist/`、`build/`、日志和缓存均为生成物，不提交到 Git。
- 前端不可只依赖公网资源，否则 OBS 离线环境会失效。

## 修改约定

- 遵循 KISS、YAGNI 和最小改动原则，优先沿用现有模块边界。
- 用户可见文本和现有注释以简体中文为主；新增内容保持一致。标识符沿用英文。
- 不要为了格式统一对 `blcsdk/` 做无关重写；它承担与 blivechat SDK 1.x 的协议兼容。
- 避免在 `gui.py`、`listener.py`、`web_server.py` 复制统计公式或状态判断。若业务规则需要变化，先提取一个无 GUI 依赖的纯函数，并为它补自动化测试。
- 正则由用户提供，不要静默改变其语义。无效表达式应在进入统计前给出清晰反馈。
- 生成 HTML 时不得直接插入未经转义的用户输入；当前旧版 `result_exporter.py` 存在该风险，若重新启用应先修复。
- 不提交密钥、令牌、本地 IDE/代理目录或用户生成结果。尤其不要提交 `BLC_TOKEN`、`.serena/`、`log/`、`build/`、`dist/`。
- 提交前查看 `git status --short` 和 `git diff --check`，只暂存本次任务相关文件；不要覆盖或清理用户已有改动。

## 常见改动定位

- 改投票规则或去重：`listener.py` 与 `VoteFrame.process_vote_by_level()`。
- 改统计口径：`VoteFrame.stop_counting()`、`web_server._sync_from_gui()`，以及前端展示。
- 改 GUI 或配置项：`gui.py`，同时更新 `config.json` 和保存/加载逻辑。
- 改 API：`web_server.py` 与 `frontend/index.html` 必须成对验证。
- 改 blivechat 协议：先确认宿主 SDK 版本，再修改 `blcsdk/`；消息数组字段依赖固定下标，尤其要谨慎。
- 增加静态资源：更新 PyInstaller spec，并验证源码运行和打包运行两种路径。

---
> Source: [KingRayCao/blivechat-niconico-rating](https://github.com/KingRayCao/blivechat-niconico-rating) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
