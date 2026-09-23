## star033

> 一个纯前端单文件 HTML 应用（虚拟恋人聊天模拟器），功能：聊天、字卡回复、朋友圈、信箱、占卜、番茄钟、礼物盒、通话模拟等。无后端、无框架、无构建工具依赖，所有代码在一个 HTML 里，浏览器直接打开就能用。

# 星言项目 — 给 AI 的工作说明（AGENTS.md）

## 这是什么
一个纯前端单文件 HTML 应用（虚拟恋人聊天模拟器），功能：聊天、字卡回复、朋友圈、信箱、占卜、番茄钟、礼物盒、通话模拟等。无后端、无框架、无构建工具依赖，所有代码在一个 HTML 里，浏览器直接打开就能用。

## 文件结构（重要）
```
app/index.html          ← 最终产物，用户实际部署/发送的就是这个文件（构建生成，勿直接改）
app/src/                ← ★ 源码（30 个片段，按功能拆分），改代码只改这里
  ├── 01_head_tools.html     head + 性能工具函数
  ├── 02_style.html         全部 CSS
  ├── 03_head_meta.html     manifest link 等 meta
  ├── 04_card_images.js     塔罗/雷诺曼牌 Base64 图片（2.2MB 纯数据，几乎不需要读）
  ├── 05_keyboard_fix.js    键盘弹出/收起修复
  ├── 06_body_skeleton.html 全部 HTML 骨架 + 开屏 + vConsole 调试注入
  ├── 07_main_js_start.html 主 JS 开头（onerror + Card Data + State）
  ├── 08_default_cards_data.js  默认通用字卡数据（275 组）
  ├── 09_storage.js         存储层（localStorage + IndexedDB，含历史 bug 修复）
  ├── 10_nav_chatlist.js    Nav + Time + Chat List
  ├── 11_chat.js            聊天核心（消息发送/渲染/保存）
  ├── 12_batch_send.js      批量发送
  ├── 13_search_switcher.js 日期搜索 + 联系人切换 + 非即时聊天
  ├── 14_emoji_contacts.js  Emoji + 添加/编辑联系人 + 拍一拍
  ├── 15_redpacket_overlays.js  红包 + 弹窗 + 通话
  ├── 16_default_common_cards.js 默认通用字卡逻辑 + milk 自动回复
  ├── 17_upload_speed_toast.js   上传 + 语速 + Toast
  ├── 18_divination_moments.js   占卜 + 朋友圈
  ├── 19_board_letters.js   留言板 + 信箱
  ├── 20_my_heart_cards.js  我的页 + 心意字卡 v2 + 交流意图
  ├── 21_autosend_settings.js    自动发送 + 设置 + 决策 + 音效
  ├── 22_calendar_diary.js  星言日历 + 日记 + 收藏
  ├── 23_pomodoro_icons.js  番茄钟 + 自定义图标 + 导航
  ├── 24_usage_giftbox.js   使用说明 + 礼物盒（自定义礼物）
  ├── 25_contact_custom.js  联系人输入栏 + 底部导航 + 复制/收藏
  ├── 26_avatar_lib_rest.js 随机头像库 + 其余主 JS
  ├── 27_pwa.js             PWA 注册
  ├── 28_misc.js            杂项
  ├── 29_usage_notice.js    使用须知
  └── 30_tail.html          结尾
scripts/
  ├── build.py              ★ 合并 src/ → app/index.html（改完必跑）
  └── split_index.py        拆分脚本（已执行过，一般不用再跑）
backup/index_拆前备份.html  ← 基准备份（build.py 用它做一致性校验，勿删）
```

## 改代码的标准流程（必须遵守）
1. **不要直接改 `app/index.html`**——它是构建产物，改了会被覆盖
2. 改对应功能的 `app/src/` 片段文件
3. 运行 `python scripts\build.py` 重新生成 `app/index.html`
4. 构建后执行 `Copy-Item app\index.html backup\index_拆前备份.html -Force` 更新基准备份
5. 用户部署/发送 `app/index.html`

## 数据存储架构（关键背景知识）
- 数据存浏览器本地，**双写**：先写 `localStorage`（key 带 `ml2_lf_` 前缀，5MB 上限），再异步写 `IndexedDB`（localforage，库名 `Star`，store `star_data`）
- 聊天记录 key：`ml2_m_<联系人id>`；联系人 `ml2_c`；自己 `ml2_p`；信件 `ml2_letters`；朋友圈 `ml2_moments`
- `?debug` URL 参数可启用 vConsole 调试面板（已内置在 `06_body_skeleton.html`）
- **已修复的历史 bug**（在 `09_storage.js`）：
  - Bug1：`restoreFromDB` 对消息 key 强制从 IndexedDB 重载并合并，防止初始化竞态覆盖历史
  - Bug2：`savemsgs` 的 IndexedDB 读取失败 catch 分支改为从 memoryCache/localStorage 兜底选最完整数据，不再直接覆盖
- **注意**：所有 JS 是全局作用域，`src/` 片段按文件名顺序拼接；改名/增删片段、调整顺序会破坏构建（build.py 的备份校验会报警）

## 常见任务定位
- 改样式 → `02_style.html`
- 改聊天逻辑 → `11_chat.js`
- 改礼物盒 → `24_usage_giftbox.js` + `06_body_skeleton.html` 里 `ov-gift-*` 弹窗
- 改通话 → `15_redpacket_overlays.js` + `06_body_skeleton.html` 里 `call-*` 按钮
- 找功能 → 先 `grep` 关键词定位片段，再读对应文件；**不要整读 index.html（4.6MB）**

## 调试
- 电脑：`python -m http.server 8800 -d app`，Chrome 开 `http://localhost:8800/index.html`
- 手机：部署后访问 `https://你的域名/index.html?debug` 看 vConsole（Console / Storage / 网络）

## 构建环境
- Windows / PowerShell；`python` 可用（Python 3.11）
- `scripts/build.py` 用 UTF-8 读写；PowerShell 下跑构建前建议 `$env:PYTHONIOENCODING='utf-8'` 避免控制台乱码

---
> Source: [ling233330-star/star033](https://github.com/ling233330-star/star033) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
