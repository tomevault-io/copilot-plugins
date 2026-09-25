## cplayer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

CPlayer 5 是**纯前端、免构建**的单页音乐播放器。没有 npm/构建流程/打包步骤，所有依赖都是仓库内的静态文件（`js/tailwindcss.js`、`js/color-thief.umd.js` 等）。修改后直接刷新浏览器即可看到效果。

## 运行与调试

项目无构建、无测试框架、无 lint 配置。开发时启动任意静态 HTTP 服务器：

```bash
python -m http.server 8080
# 或
npx serve .
```

访问 `http://localhost:8080`。

**Service Worker 缓存陷阱**：`sw.js` 会缓存核心静态资源（含 `index.html`、`css/*`、`js/*`）。改动这些文件后，浏览器可能命中旧缓存看不到更新。调试时需在 DevTools → Application → Service Workers 勾选「Update on reload」或手动 Unregister，并且每次改动缓存资源清单后应递增 `sw.js` 顶部的 `CACHE_NAME`（如 `cplayer5-v4` → `v5`）以触发旧缓存清理。

## 代码结构

绝大部分代码集中在 **`index.html`（~287KB，约 6000+ 行）**，是理解项目的核心：

- `<head>`：外部资源引入 + 大段内联 `<style>`（设计系统，约 41–997 行）。
- `<script type="module">`（约 1708 行至文件末尾）：**全部业务逻辑**。包括播放引擎、搜索、歌词、歌单、云同步 UI、可视化、移动端交互等。所有函数都在这一个模块作用域内，通过顶层 `function` 声明组织，无模块拆分。
- `js/sync.js`：**唯一独立出来的逻辑文件**，导出全局 `window.CPSync` 类，负责云同步全部数据层。
- `playlist.js`：本地离线歌单（`window.LOCAL_PLAYLIST`），**已被 gitignore**，属于用户侧可选文件，启动时若存在则优先加载。

## 核心架构

### 数据/API 分层

- **`MusicService` 类**（index.html 内）：音乐内容 API 层，请求 `https://api.chksz.com/api`（Cloudflare Worker，可被 `localStorage.cp_api_base` 覆盖）。API 侧对 `cp.chksz.top` 等来源做了 Origin 白名单免 Key 放行；本地开发可设 `localStorage.cp_api_key`（个人密钥，`apiUrl()` 会自动附加）。默认音质写死为 `jymaster`（超清母带）。返回的 URL 统一 `http:`→`https:` 升级。
- **多音源**：歌曲 ID 用前缀区分来源——网易云为纯数字（向后兼容），QQ 音乐为 `qq:<mid>`，酷狗为 `kg:<hash>`。`songSourceOf(id)` 推导来源，`getSong`/`getLyric`/搜索按前缀分发；QQ/酷狗的歌词随解析返回，缓存在模块级 `sourceLyricCache`。酷狗音频 CDN 仅 http，播放 URL 经 Worker `/kg_stream?url=` 代理转流。QQ/酷狗不支持音质切换与歌单链接解析导入。搜索源状态存 `localStorage.cp_search_source`，来源徽章用 `sourceBadgeHtml(id)` 渲染。
- **`window.CPSync` 类**（js/sync.js）：云同步与歌单持久化层，请求 `https://sync.chksz.top/api/v1`（可被 `localStorage.cp_sync_api` 覆盖）。是 `EventTarget`，通过 `emit('auth'|'status'|'mirror', …)` 向 UI 广播状态变化。track 的 `id` 允许字符串（服务端 `sanitizeTracks` 同样接受），QQ/酷狗歌曲可直接进云歌单。

这些 API 域名在 `sw.js` 中被显式排除缓存（始终网络优先）。

### 歌曲上下文菜单

`attachSongContextMenu(el, getData)` 给条目绑定桌面右键 + 移动长按（480ms，位移>10px 取消）菜单；`openSongContextMenu({song, context, index, x, y})` 渲染。`context: 'search'` 提供立即播放/下一首播放，`'playlist'` 提供立即播放/从播放列表删除（`removeSongUnified`：云歌单走 `sync.removeTrack`，普通列表本地 splice）。桌面为浮动玻璃拟态菜单（`.ctx-menu`），移动端同一组件以底部动作面板形态呈现（CSS 媒体查询切换）。

### 云歌单同步模型（CPSync 关键设计）

CPSync 维护一份 localStorage 镜像（`cp_sync_mirror`），每个歌单带 `syncState`：

- `local`：仅本地、未登录时创建的歌单，所有改动只写镜像。
- `dirty`：本地有未上传的改动。
- `cloud`：已与服务端同步。
- `public`：他人分享的只读歌单。

**乐观并发控制**：更新走 `version` + `baseVersion`，服务端返回 409 时进入冲突流程（`updatePlaylist` 的 `onConflict` 回调 / UI 侧 `showConflictModal`）。歌单 ID 是 6 位 `[a-z0-9]` 短码（`shortId`/`validShortId`）。

修改同步逻辑时注意：本地态（`local`）与云端态走完全不同的代码路径——本地态只改镜像，云端态才发请求。`getPlaylist`、`updatePlaylist`、`addTracks`、`removeTrack` 都有这个分叉判断。

### 播放引擎

- 全局状态：`audio`（主 Audio 元素）、`preloadAudio`（无缝预加载下一首）、`playlist` 数组、`currentIndex`、`playMode`（`sequence`/`random`/`single`）。
- `loadAndPlaySong(id)` 是播放主入口：**先取音频 URL 立即播放**，再异步加载歌词与封面（音频最高优先级，避免歌词/封面阻塞播放）。
- 歌词：`parseLrc`/`parseLyrics`（LRC + 翻译 TLRC），`renderLyrics`/`updateLyrics` 按时间高亮滚动。
- 长歌单用虚拟滚动渲染（`setupVirtualScroll`/`vsRenderVisible`），桌面与移动端各有一套（移动端 `mCreateItem`/`mRender`）。

### 缓存层（IndexedDB）

`initDatabase` 建立 `CPlayer5DB`，含 `playlists`/`lyrics`/`images` 三个 object store。歌单采用「先读缓存、后台静默刷新」策略（`loadPlaylistById` → `refreshPlaylistInBackground`）。封面图片可经 `getCachedImage` 转 base64 缓存。

### 状态持久化约定

大量配置存于 `localStorage`，键统一以 `cp_` 前缀命名：`cp_quality`、`cp_api_base`、`cp_sync_api`、`cp_sync_token`、`cp_sync_mirror` 等。新增持久化配置应沿用此前缀约定。

## 开发注意事项

- 因为逻辑全在 `index.html` 单文件内，定位函数优先用 grep 搜索 `function 函数名`；文件顶部约 2196–3210 行集中了 UI 初始化与云同步 UI 函数，3216 行起为播放/歌词/歌单核心。
- 新增依赖应放入 `js/` 作为静态文件本地引入，不要引入需要构建的包管理方案。
- 涉及外部内容（API 响应、导入的 JSON/playlist.js）一律当作不可信数据处理，渲染到 DOM 前用 `escapeHtml`。

---
> Source: [ChKSz/CPlayer](https://github.com/ChKSz/CPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
