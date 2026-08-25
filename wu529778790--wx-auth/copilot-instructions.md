## wx-auth

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

这是一个**微信订阅号认证系统**，核心功能是通过微信扫码关注公众号获取验证码，输入6位验证码完成认证。认证成功后保存状态，下次访问自动认证（无需重复操作）。

**技术栈：**
- **前端**: Nuxt 4 + Vue 3 + TypeScript + Tailwind CSS
- **后端**: Nitro Server API (Node.js)
- **SDK**: 原生 JavaScript + Vite 构建（< 12KB，零依赖）
- **存储**: Turso（libSQL 托管）单一后端，对外统一 async 接口

## 快速导航

- **后端 API**: `server/api/` - Nitro Server API endpoints
- **工具函数**: `server/utils/` - 微信加解密、存储、Session 管理
- **SDK 源码**: `wx-auth-sdk/src/` - 前端 SDK 核心逻辑
- **前端页面**: `pages/` - Nuxt 页面路由
- **数据目录**: `data/` - 本地 Turso 文件库（TURSO_URL=file: 时）及历史备份

## 开发命令

### 主项目
```bash
npm install          # 安装依赖
npm run dev          # 开发模式（http://localhost:3000）
npm run build        # 构建生产版本
npm run preview      # 预览生产构建
npm run generate     # 生成静态站点
```

### SDK 开发
```bash
cd wx-auth-sdk
npm install          # 安装依赖
npm run dev          # SDK 开发模式
npm run build        # 构建 SDK（输出到 dist/）
npm run type-check   # TypeScript 类型检查
```

### Docker 部署
```bash
# 本地构建测试
docker build -t wx-auth .
docker run -d --name wx-auth --env-file .env -p 3000:3000 -v ./data:/app/data wx-auth

# 查看日志
docker logs -f wx-auth

# 停止容器
docker stop wx-auth && docker rm wx-auth
```

## 核心架构

### 后端 API 端点
1. **`/api/wechat/message`** - 微信消息处理（GET/POST）- 接收微信服务器推送
2. **`/api/auth/check`** - 认证检查（参数：`authToken` 或 `openid`，可选 `siteId`）
3. **`/api/auth/userinfo`** - 用户详情（业务授权用：`role` / `vipExpiresAt` / `isAdmin`，参数 `token` 或 Cookie；不提供明文 openid 通道）
4. **`/api/auth/session`** - Session 管理（POST/GET/DELETE）
5. **`/api/sdk/config`** - SDK 配置下发（返回 wechatName、qrcodeUrl）

### 后台管理 API（需管理员登录，身份 = 用户表 `is_admin` 标记）
1. **`POST /api/admin/login`** - 管理员登录（接收微信签名 Token，校验 `is_admin`，写 `wxauth-admin` Cookie；免密）
2. **`POST /api/admin/logout`** - 管理员登出
3. **`GET /api/admin/stats`** - 统计看板（累计/有效/今日/流失率，按站点分组）
4. **`GET /api/admin/users`** - 用户列表（分页 + siteId/status/role 筛选 + keyword 搜索）
5. **`PATCH /api/admin/users/[openid]`** - 编辑用户（role / vipExpiresAt / remark / status / isAdmin）

> 鉴权：`server/middleware/admin-guard.ts` 守卫所有 `/api/admin/*`（login 除外），未登录返回 401。登录态基于加密 Cookie（AES-256-GCM），与用户认证 Session 隔离。

### 前端页面
- **`pages/index.vue`** - 认证演示页面 + SDK 接入文档（合并展示）
- **`pages/admin.vue`** - 后台管理（数据看板 + 用户列表，Tabs 切换）
- **`pages/admin/login.vue`** - 后台登录页（微信认证免密登录）

### SDK 工作流程
1. 检查 Cookie `wxauth-openid`
2. 已认证 → 静默通过
3. 未认证 → 显示弹窗（二维码 + 6位输入框）
4. 用户扫码 + 输入验证码
5. 验证成功 → 保存 Cookie + 回调
6. 用户关闭弹窗 → 触发 `onClose` 回调

### 工具层（server/utils/）
- **`wechat.ts`** - 微信 API 交互（签名验证、消息解析、加密解密）
- **`storage.ts`** - 存储层抽象（Turso 单一后端，按 siteId 隔离）
- **`turso.ts`** - Turso 访问层（@libsql/client，惰性 prepare + 事务 batch + 幂等建表）
- **`session.ts`** - Session 生成与验证（AES-256-GCM 加密）
- **`token.ts`** - Token 生成与管理
- **`rate-limit.ts`** - 速率限制（防止暴力破解）
- **`admin-auth.ts`** - 后台管理员登录态（独立 Cookie，AES-256-GCM 加密）

### 用户数据模型（storage.ts）
```ts
interface AuthenticatedUserData {
  authenticatedAt: string;
  nickname?: string;
  headimgurl?: string;
  unionid?: string;
  siteId?: string;
  status?: 'active' | 'unsubscribed' | 'disabled';  // active=有效 / unsubscribed=已取关 / disabled=被封禁
  unsubscribedAt?: string;   // 取关时间（软删除留痕）
  role?: 'normal' | 'vip';   // 角色
  vipExpiresAt?: number;     // VIP 到期时间戳（毫秒），null 表示永久
  remark?: string;           // 管理员备注
}
```

> **软删除约定**：取关事件 `clearUserAuthentication` 只打 `status='unsubscribed'` 标记并记录时间，**不物理删除**。认证检查（`isUserAuthenticated` / `getAuthenticatedUser`）仅认 `active`（老数据无 status 视为 active），重新认证自动复活为 `active` 且保留 role/remark。后台统计区分「累计（totalAuth）」与「有效（totalActive）」。

## 关键配置

### 环境变量（.env）
```bash
# 必须（Docker 部署必须加 NUXT_ 前缀，否则构建时被内联为空值）
NUXT_PUBLIC_SITE_URL=https://wx-auth.shenzjd.com
NUXT_WECHAT_TOKEN=your-wechat-token
NUXT_SESSION_SECRET=dev-secret-change-in-production
# 后台已改为免密登录（is_admin 角色判定），无需 ADMIN_PASSWORD；
# 首次设置管理员用：npx tsx scripts/set-admin.ts <openid>
NUXT_WECHAT_NAME=神族九帝
NUXT_WECHAT_QRCODE_URL=https://your-site.com/qrcode.jpg

# 可选（个人订阅号留空）
NUXT_WECHAT_AES_KEY=
NUXT_CODE_EXPIRY=300
NUXT_KEYWORDS=["验证码"]      # 触发关键词，支持简繁体（发"驗證碼"也可触发）
NUXT_WECHAT_MENU_KEY=GET_CODE  # "验证码"菜单的事件 key（需与公众号后台配置一致）
# 存储后端：turso（唯一，默认；设置其他值会启动报错退出，防数据分叉）
# 注意：STORAGE_TYPE 无 NUXT_ 前缀，直接读环境变量，Docker 部署时通过 -e STORAGE_TYPE=turso 传入
# Turso 模式需配置 TURSO_URL / TURSO_TOKEN（Turso 控制台 Database → Generate Token）
STORAGE_TYPE=turso
```

### SDK 配置
```typescript
// ✅ 零配置接入（推荐）
WxAuth.init({
  onVerified: (user) => { ... },  // ← 唯一必填：验证成功回调
  onClose: () => { ... }          // ← 可选：关闭弹窗回调
});

// 或手动指定配置（可选）
WxAuth.init({
  apiBase: 'https://wx-auth.shenzjd.com',  // 后端 API 地址（可选，有默认值）
  required: false,                     // 是否必须认证（默认 true）
  silent: false,                       // ← 可选：true 时 init 不调弹窗，弹窗由 requireAuth() 手动触发
  // wechatName 和 qrcodeUrl 无需配置，自动从后端获取
  // 接入方配置自己的公众号名称和二维码无效（统一使用"神族九帝"）
  onVerified: (user) => { ... },       // 验证成功回调
  onError: (error) => { ... },        // 错误回调
  onClose: () => { ... }               // 关闭弹窗回调（仅 required=false 时触发）
});

// ✅ silent 模式：延迟弹窗（例：免费搜索 3 次后再要求认证）
WxAuth.init({
  silent: true,
  required: false,
  onVerified: (user) => { /* 标注已认证 */ },
});
// 业务代码里：freeSearches 用完后 await WxAuth.requireAuth() 手动触发弹窗
```

**核心概念：**
- ✅ **所有参数都无需手动配置**
  - `siteId`：自动从 `document.referrer` 或当前域名获取（SDK 内部处理，无需也**不支持**配置）
  - `apiBase`：使用默认值 `https://wx-auth.shenzjd.com`
  - `wechatName` / `qrcodeUrl`：自动从后端获取
- ✅ 所有接入方共享"神族九帝"公众号
- ✅ 认证流程统一走后端配置的公众号

### required 模式
- **true（默认）** - 强制认证，不显示关闭按钮，点击遮罩无效
- **false** - 可选认证，显示关闭按钮，点击遮罩可关闭，支持 `onClose` 回调

## CI/CD 工作流

### 自动部署（push 到 main）
- **触发条件**: 推送代码到 `main` 分支
- **工作流**: `.github/workflows/docker.yml`
- **流程**:
  1. 构建 Docker 镜像
  2. 推送到 GitHub Container Registry (ghcr.io)
  3. SSH 部署到生产服务器（端口 `6702`）
  4. 数据持久化: `/opt/1panel/apps/openresty/openresty/www/sites/wx-auth.shenzjd.com/index/data`

### SDK 自动发布（修改 wx-auth-sdk 后 push 到 main）
- **触发条件**: 修改 `wx-auth-sdk/` 目录后推送
- **工作流**: `.github/workflows/publish-sdk.yml`
- **流程**:
  1. 自动获取 NPM 最新版本并 +1
  2. 更新 `wx-auth-sdk/package.json` 版本号
  3. 构建 SDK 并验证产物
  4. 发布到 NPM（`wx-auth-sdk`）
  5. 提交版本号更新到 main
  6. 创建 GitHub Release（`sdk-v{x.y.z}`）

## 核心特性

### 安全性
- AES-256-GCM 加密 Session
- 验证码 5 分钟过期，一次性使用
- 微信消息签名验证
- 支持安全模式（加密消息）
- Cookie: HttpOnly + SameSite=none + Secure

### 用户体验
- 自动聚焦第一个输入框
- 支持粘贴 6 位数字
- 键盘导航（退格、方向键）
- 输入完成自动验证
- 微信原生风格 UI（#07C160）
- 自动认证（无需重复操作）

### 存储层
- **Turso**（唯一后端，2026-08-18 起生产）：libSQL 托管（SQLite 兼容分支），托管多副本 + 自动备份，根治 SQLite 文件损坏事故（8/17 btree 损坏 80+ 处 → 崩溃循环）
  - 切换：`STORAGE_TYPE=turso`（默认）+ `TURSO_URL`（libsql://xxx.turso.io）+ `TURSO_TOKEN`
  - 访问层 `server/utils/turso.ts`：基于官方 @libsql/client（惰性单例），对外接口统一（惰性 prepare + 事务性 batch + 幂等建表）
  - 本地开发：`TURSO_URL=file:./data/turso.db` 走本地文件，无需联网（自动探测：file: 前缀 = local，其余 = http）
  - 注意：@libsql/client `batch()` 默认事务（deferred）模式；execute args 需为 `InArgs`（数组或对象）
- 存储层对外接口全部 async，调用方需 `await`
- 已移除：Cloudflare D1、SQLite（node:sqlite）、JSON 文件（file）后端、旧部署脚本（Cloudflare/EdgeOne）与一次性迁移脚本

## SDK 构建产物

`wx-auth-sdk/dist/`:
- `wx-auth.js` - ES Module (7.1 kB)
- `wx-auth.umd.js` - UMD (11 kB)
- `wx-auth.css` - 样式 (3.4 kB)
- `index.d.ts` - TypeScript 类型声明

## 认证流程

### 强制认证模式（required=true，默认）

```
用户访问 → 检查 Cookie
    ↓
已认证？ → 静默通过
    ↓
未认证？ → 显示弹窗（无关闭按钮）
    ↓
用户扫码关注公众号 → 公众号自动回复验证码
    ↓
用户输入 6 位验证码 → 验证
    ↓
成功 → 保存 Cookie + 回调
```

### 可选认证模式（required=false）

```
用户访问 → 检查 Cookie
    ↓
已认证？ → 静默通过
    ↓
未认证？ → 显示弹窗（有关闭按钮）
    ↓
用户选择：
    - 扫码认证 → 完成验证 → 保存 Cookie
    - 关闭弹窗 → 触发 onClose 回调 → 继续浏览
```

## 注意事项

1. **微信配置**：个人订阅号只需 `WECHAT_TOKEN`，服务号才需要 `WECHAT_APPID`/`WECHAT_APPSECRET`
2. **Session 密钥**：生产环境必须使用随机字符串（`openssl rand -hex 32`）
3. **网站地址**：`SITE_URL` 必须与微信后台配置的回调地址一致
4. **SDK 导入**：开发时从 `../wx-auth-sdk/src/index` 导入，生产时从 NPM 包导入
5. **公众号配置**：接入方无需配置 `wechatName` 和 `qrcodeUrl`，配置也无效（统一使用后端配置的"神族九帝"公众号）
6. **siteId 自动获取**：SDK 会自动从 `document.referrer` 或当前域名获取 `siteId`，无需手动配置
7. **Docker 环境变量前缀**：私有配置需加 `NUXT_` 前缀（如 `NUXT_WECHAT_TOKEN`），公开配置加 `NUXT_PUBLIC_` 前缀（如 `NUXT_PUBLIC_SITE_URL`）

## 调试技巧

### 查看后端日志
```bash
# Docker 环境
docker logs -f wx-auth

# 开发环境
# 日志直接输出到终端
```

### 测试微信消息接收
```bash
# 微信后台配置服务器 URL
# https://wx-auth.shenzjd.com/api/wechat/message
```

### 验证认证状态
```bash
# 检查 Cookie
curl -I https://wx-auth.shenzjd.com

# 手动调用认证检查接口
curl "https://wx-auth.shenzjd.com/api/auth/check?openid=test-openid"
```

### SDK 开发调试
```bash
# 在首页测试（演示 + 接入文档已合并到一页）
# http://localhost:3000

# 清空 Cookie 重新测试
document.cookie = 'wxauth-openid=; path=/; max-age=0'
location.reload()
```

## Git 工作流

- **主分支**: `main` - 生产环境代码
- **发布流程**: push 到 main 自动触发 Docker 构建 + 部署
- **SDK 发布**: 修改 `wx-auth-sdk/` 后 push 自动发布 NPM 包
- **Commit 规范**: 建议使用 `feat:` / `fix:` / `chore:` / `refactor:` 等 conventional commits

## 常见问题

### 1. Cookie 不生效？
- 检查 `siteUrl` 配置是否与当前域名一致
- 确认 Cookie 域名设置（生产环境需配置为父级域名）

### 2. 验证码不生效？
- 检查验证码是否过期（默认 5 分钟）
- 确认验证码已从消息队列删除（一次性使用）
- 检查 `code` 存储中是否包含正确的 `siteId`

### 3. 微信消息收不到？
- 确认 `WECHAT_TOKEN` 配置正确
- 检查微信后台服务器配置 URL 是否正确
- 查看后端日志是否有签名验证失败
- 确认防火墙允许微信服务器 IP 访问

### 4. Docker 环境变量不生效？
- Docker 部署必须加 `NUXT_` 前缀
- 本地开发直接用原始名称（`WECHAT_TOKEN` 而非 `NUXT_WECHAT_TOKEN`）
- 检查 `.env` 文件是否在正确的目录

### 5. SDK 配置不生效？
- `wechatName` 和 `qrcodeUrl` 由后端 `/api/sdk/config` 自动下发
- 接入方在 SDK 中配置这两个参数无效
- 确认后端 `NUXT_WECHAT_NAME` 和 `NUXT_WECHAT_QRCODE_URL` 已配置

## 公众号菜单配置

⚠️ **订阅号限制**：个人订阅号无法通过 API 接口配置菜单，只能通过后台手动配置。

### 手动配置步骤

1. 登录 [微信公众平台](https://mp.weixin.qq.com/)
2. 进入「内容与互动」→「自定义菜单」
3. 点击「+ 创建菜单」
4. 配置菜单（最多3个主菜单，每个主菜单最多5个子菜单）
5. 点击「保存并发布」（24小时内全量生效）

### 推荐菜单配置

#### 认证订阅号（有网页跳转权限）
```
📱 菜单一：前往认证
   └─ 跳转网页 → https://wx-auth.shenzjd.com

🔧 菜单二：我的工具
   ├─ 在线网盘 → https://alist.shenzjd.com
   ├─ 快链工具 → https://duanlian.shenzjd.com
   └─ 视频解析 → https://parse.shenzjd.com

🌐 菜单三：更多
   ├─ 首页 → https://shenzjd.com
   └─ GitHub → https://github.com/wu529778790
```

#### 未认证订阅号（菜单能力受限，免费折中方案）
> ⚠️ **官方限制**：未认证订阅号的菜单不支持「点击推事件」（不会推送事件到服务器），
> 「发送信息」动作也不支持文本类型（只能选图文/图片/语音/视频素材）。
> 因此**无法**通过菜单实现"点一下→服务器动态回复验证码"。
>
> **免费折中方案**：菜单配「发送信息」→ 图文素材，图文里写清操作指引；
> 服务器端已内置**模糊匹配**（`containsAuthKeyword`），用户只需回复一个字：
> - 单字：`码` / `碼`
> - 简体：`验证码`；繁体：`驗證碼`
> - 拼音：`yanzhengma` / `yzm`；错别字：`验证马` / `验证吗` / `验证`
>
> 想实现真正"点菜单直接拿验证码"，唯一路径是微信认证（个人认证订阅号 300 元/年）。

#### 认证订阅号（推荐，支持点击推事件）
```
📱 菜单一：验证码
   └─ 点击推事件 → 事件 key 填 GET_CODE（与 NUXT_WECHAT_MENU_KEY 一致）
      → 服务器收到 CLICK 事件后动态生成验证码回复
```

### 注意事项
- **菜单名称**：最多4个汉字
- **发布延迟**：保存后需要24小时才能全量生效
- **修改限制**：每月只能修改5次菜单（认证服务号无限制）
- **未认证订阅号**：仅支持「发送信息」（图文/图片/语音/视频素材，不支持文本）与跳转图文素材，不支持「点击推事件」「跳转网页」
- **认证订阅号**：支持「点击推事件」→ 服务器 `CLICK` 事件处理（`server/api/wechat/message.ts`），无需用户打字

## 相关资源

- **NPM 包**: https://www.npmjs.com/package/wx-auth-sdk
- **GitHub 仓库**: https://github.com/wu529778790/wx-auth
- **生产地址**: https://wx-auth.shenzjd.com
- **SDK 文档**: `wx-auth-sdk/README.md`

---

**版本**: v1.2.6
**状态**: ✅ 生产就绪

---
> Source: [wu529778790/wx-auth](https://github.com/wu529778790/wx-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
