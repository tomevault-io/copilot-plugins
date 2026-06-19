## telos

> 逆向设计 / 从结果倒推的学习引擎。说出目标 → 倒推出带前置依赖的能力图谱 → 诊断起点 → 只教缺的、边教边验证 → 间隔复习。

# Telos — 项目说明（自描述）

逆向设计 / 从结果倒推的学习引擎。说出目标 → 倒推出带前置依赖的能力图谱 → 诊断起点 → 只教缺的、边教边验证 → 间隔复习。

## 交付标准（最重要）

- **极致完成度**：交付的版本就是高完成度成品，不交半成品 / 过程产物。
- **不闭门造车**：动手前做**广度 + 深度兼具的 web search**，参考头部开源项目、头部产品设计、对应学术论文。
- **细节一次做全**：导航 / 空状态 / 回退入口 / 错误态等一次补齐，不要等用户反复提醒。

## 架构

- `core/` — 学习引擎，Python 零依赖（KST · BKT+CBM 诊断 · FIRe · FSRS · LLM 倒推）。`serve.py` = 本地 CORS 代理（读 `core/.env` 的 key，跑 127.0.0.1:8787，路由 `/derive` `/lesson` `/probe`）。
- `web/` — 产品，Next.js 16 + React 19 + Tailwind v4 + TS，**静态导出**（`output:export`，prod `basePath=/app`）。部署在自定义域 `telos.ungetsu.net`：landing 在 `/`，app 在 `/app/`（GitHub Pages 项目站 `…github.io/telos/*` 由 GitHub 自动 301 过来）。
- `workers/` — 生产用 Cloudflare Worker 代理（镜像 serve.py）+ **付费 webhook `/billing/webhook`**（验签后用 service_role 写 Supabase `app_metadata.telos_pro`；service_role/webhook secret 只存 wrangler secret，绝不进前端/仓库）。
- **付费（Telos Pro）**：配置单点 `web/lib/telos/billing-config.ts`（服务商/价格/checkout/限额/托管配额/加油包）；权益 `web/lib/telos/billing.ts`（`isPro()`/`refreshEntitlement()`，真源=app_metadata）。免费=BYOK 无限+托管试用 3 次+3 项目+水印导出；Pro=托管 AI 月度配额（30 倒推+600 微课）+无限项目+无水印+Anki 导出+全部官方模板+书斋治学轨（治学通行证治学轨额外墨 + 赛季限定印章/雅号/文房 + 全部画风与纸色）。定价页 `/pro`。
- **书斋 / 形象经济**（一级页 `/studio`，4 大板块）：**形象**（神态·造型·画风）· **印记**（雅号·印章）· **陈设**（案头装点·纸色）· **治学通行证**。顶部统一 anchor 贯穿全页（她+雅号+印+墨+进阶）。概念分域勿混：「画风」=她被怎么画的笔法墨韵（泼墨/木刻，`portraits.ts` theme 系列，Pro）；「纸色」=整个 App 的纸张色温（`theme.ts`，免费 2 + Pro 2）。引擎 `lib/telos/{pass,seals,studyroom,wardrobe,theme}.ts`，解锁语义复用 `portraits.matchUnlock`，跨设备同步走 `sync-state.ts`。红线：纯外观/荣誉、绝不影响学习与 XP；墨只赚不卖、无开箱抽卡、通行证进度只能靠学不可买跳级。Pro=治学轨（每阶额外墨 + 赛季限定印章/雅号/文房）+ 全画风 + 全纸色。
- **托管 AI**：无 BYOK 的请求带 Supabase token → Worker `hostedGate` 验身份 + KV(`TELOS_USAGE`) 计量（月度/试用/加油包 `pack_d10|pack_l200`）；BYOK 请求旁路计量零成本。错误码 NEED_LOGIN/HOSTED_TRIAL_USED/HOSTED_QUOTA/NO_HOSTED 前端本地化。买断(lifetime)**不含**托管用量。
- **模板店内容安全**：付费图谱内容（desc/drill/benchmark）**绝不进前端/仓库**（防白嫖）——前端 `templates.ts` 只留 meta+脱敏大纲，完整 points 存 KV `tpl:<id>`（私有母版 `workers/templates-private.json` **git-ignored**，`workers/seed-templates.sh` 灌入），Worker `POST /template` 鉴权（`verifyUser`→已购 `app_metadata.telos_templates` 或 Pro）后下发，前端 `derive.ts fetchTemplatePoints` 拉取。免费模板（科二）内容内嵌前端。Pro=全解锁，非 Pro 可单买。
- **设计基准 = 实际 App（`/app/`，即 `web/`）**，所有产品视觉/品牌/图标以它为准。`landing/index.html`（营销落地页，部署在 `/`）已对齐当前 App + 产品：单文件内联 CSS、响应式（预览图用随容器缩放的 viewBox SVG，禁止固定 px 绝对定位防溢出）、webp 立绘；内容含 工作方式/学习地图/坚持/书斋形象经济/定价(BYOK·Pro·模板店)/学习科学。产品再演进时同步更新此页，勿让「完全免费」等措辞回潮（现为 BYOK 免费 + Pro 托管）。

## web 关键

- 路由：`/`(引导或地图主页) · `/diagnose` · `/review` · `/streak`(坚持/激励) · `/studio`(书斋) · `/me` · `/settings` · `/account`(登录) · `/privacy` · `/terms`。学习/诊断是全屏接管。
- `lib/telos/use-project.tsx` — 单一真相源 Context（localStorage `telos:project`），Provider 挂 `app/layout.tsx`。
- 端点配置：env `NEXT_PUBLIC_TELOS_DERIVE_URL` 或 localStorage 覆盖；key 永远在服务端，绝不进前端 / 仓库。
- React Flow 画布（`components/canvas.tsx`），手机竖屏走 `components/path-view.tsx`。

## 约定

- **设计语言**：纯黑白 + 暖灰纸感；Fraunces 衬线 + Inter + JetBrains Mono；**手绘线性图标**（`components/icon.tsx`，带手抖滤镜）。**禁用 emoji**。看板娘=年轻女老师，黑白墨线。**完整设计参考见 `docs/DESIGN.md`**（色彩/字体/图标/状态/组件/游戏化/动效/IA 全量规范）。
- commit：conventional（`feat:`/`fix:`/`chore:`），结尾 `Co-Authored-By: Claude Opus 4.8 (1M context) <noreply@anthropic.com>`。
- 代码改动**默认直接提交 + push + 部署**，不用问（用户已授权）。
- 回复用**中文**；不写"总结"段落。
- **绝不**提交 `.env` / 任何 key；提交前过一遍 secret 检查。

---
> Source: [YunyueLi/telos](https://github.com/YunyueLi/telos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
