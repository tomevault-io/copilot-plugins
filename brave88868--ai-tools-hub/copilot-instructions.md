## ai-tools-hub

> **产品名**: AI Tools Station

# CLAUDE.md — AI Tools Station
# 最后更新: 2026-04-11
# 路径: E:\Projects\ai_tools_hub

---

## 项目概览

**产品名**: AI Tools Station  
**域名**: aitoolsstation.com  
**GitHub**: brave88868/ai_tools_hub  
**技术栈**: Next.js 14 + Supabase + Stripe + Vercel  
**当前状态**: 已上线，持续迭代中

---

## 核心数据

- **Toolkits**: 24个（大写显示，去掉"Toolkit"后缀）
- **AI Tools**: 638个（is_active = true）
- **AI Generators**: 20个（/ai-generators/*）
- **Blog Articles**: 若干（已生成）
- **SEO Comparisons**: 10条（已 seed）
- **SEO Alternatives**: 10条（已 seed）

---

## 目录结构

```
E:\Projects\ai_tools_hub
├── app/
│   ├── page.tsx                    # 首页
│   ├── layout.tsx                  # 全局布局（flex flex-col min-h-screen）
│   ├── api/
│   │   ├── generate/route.ts       # OpenAI API 代理（gpt-4o-mini）
│   │   └── demo/generate/route.ts  # Demo 生成（同上）
│   ├── ai-generators/
│   │   ├── page.tsx                # Server Component，fetch generators
│   │   └── [slug]/page.tsx         # 单个 generator 页面
│   ├── toolkits/
│   │   └── page.tsx                # Toolkits 页面（含价格和订阅）
│   ├── blog/
│   │   ├── page.tsx
│   │   └── [slug]/page.tsx
│   ├── compare/[slug]/page.tsx     # SEO 对比页
│   ├── alternatives/[slug]/page.tsx # SEO 替代品页
│   ├── tools/[slug]/page.tsx       # 单个工具页
│   ├── features/page.tsx           # 功能介绍页
│   ├── pricing/page.tsx            # 重定向到 /toolkits
│   ├── privacy/page.tsx            # 隐私政策（含 APP/GDPR/EU AI Act）
│   ├── terms/page.tsx              # 服务条款
│   └── sitemap-generators.xml/
│       └── route.ts                # Sitemap（.range(0, 49999)）
├── components/
│   ├── home/
│   │   ├── Hero.tsx                # 两列布局：左(标题+HOW IT WORKS) 右(Example+Demo)
│   │   ├── HeroDemoEmbed.tsx       # 嵌入式 Live Demo（调用 /api/generate）
│   │   ├── WhatYouCanCreate.tsx    # 6个用例 pills（1行）
│   │   ├── ToolkitsByGroup.tsx     # Workflow Explorer（首页3卡片）
│   │   ├── TrustSection.tsx        # 品牌 badges：OpenAI/Anthropic/Supabase/Stripe
│   │   ├── FinalCTA.tsx            # 渐变 bar（indigo→violet）
│   │   ├── PopularGenerators.tsx   # 已删除（从首页移除）
│   │   ├── SocialProof.tsx         # 已删除（stats 移入 Hero）
│   │   ├── ValuePillars.tsx        # 已删除（替换为 WhatYouCanCreate）
│   │   └── PopularTools.tsx        # 已删除
│   ├── generators/
│   │   └── GeneratorsGrid.tsx      # Client Component，filter + sort
│   └── Footer.tsx
├── scripts/
│   ├── seed-comparisons.mjs
│   ├── seed-alternatives.mjs
│   ├── generate-comparisons.mjs
│   ├── generate-alternatives.mjs
│   └── generate-use-cases-missing.mjs
└── public/
    └── robots.txt                  # 含 Sitemap URLs
```

---

## 首页组件顺序（app/page.tsx）

```tsx
<Hero />              // 两列：左=标题+HOW IT WORKS，右=Example Output+Live Demo
<WhatYouCanCreate />  // 6个用例图标（1行）
<ToolkitsByGroup />   // Workflow Explorer（首页 limit=3）
<TrustSection />      // OpenAI/Anthropic/Supabase/Stripe badges
<FinalCTA />          // 渐变色 bar
```

---

## API 配置

### OpenAI（生产使用）
```
OPENAI_API_KEY=sk-...（在 .env.local 和 Vercel 环境变量中）
模型: gpt-4o-mini
路由: app/api/generate/route.ts
```

### Anthropic（未启用，key 为空）
```
ANTHROPIC_API_KEY=（空，暂未配置）
```

### Supabase
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
数据存储区域: AWS ap-southeast-2（悉尼）
```

### Stripe
```
STRIPE_SECRET_KEY=（Live 模式，已确认）
STRIPE_WEBHOOK_SECRET=
```

---

## 关键设计决策

### Toolkit 显示规范
- 名字全部**大写**
- 去掉 "Toolkit" 后缀
- 例：`Jobseeker Toolkit` → `JOBSEEKER`
- 例：`Exam Toolkit` → `EXAM & STUDY`

### Generator 路由
- URL: `/ai-generators/[slug]`（非 `/tools/[slug]`）
- Footer 链接全部指向 `/ai-generators/*`
- Breadcrumb: Home → AI Generators → Compare/Alternatives

### Filter 顺序（GeneratorsGrid）
固定 FILTER_ORDER：
```
all / job-seekers / marketing / business / creators / developers / education / legal / productivity
```

### Sitemap
- 所有表查询用 `.range(0, 49999)`（非 `.limit()`）
- robots.txt 包含 sitemap URL

### layout.tsx 结构
```tsx
<Header />
<div className="flex flex-col min-h-screen">
  <div className="pt-14 flex-1">
    {children}
  </div>
  <Footer />
</div>
```

### /pricing 页面
已重定向到 `/toolkits`（含价格和订阅按钮）

---

## SEO 状态（2026-04-11）

### 已收录
- 首页 ✅
- Blog 页面 ✅（1周前）
- Use case 页面 ✅

### 待收录
- /ai-generators/* ⚠️（已提交 Google Search Console，等待抓取）
- /compare/* ⚠️
- /alternatives/* ⚠️

### 已修复
- 旧品牌名 "AI Tools Hub" → "AI Tools Station"（DB 已更新）
- Sitemap 已提交到 Google Search Console
- robots.txt 已加 Sitemap URL

---

## 合规声明（已写入 /privacy 和 /terms）

- **澳大利亚 APP**（Privacy Act 1988）✅
- **GDPR**（EU 用户）✅
- **EU AI Act**（限制风险级别）✅
- **数据存储地**：AWS ap-southeast-2（悉尼）✅
- **数据保留期**：账户数据90天，支付记录7年 ✅
- **管辖法律**：新南威尔士州法律 ✅

---

## 已知问题 / 待处理

| 优先级 | 问题 | 状态 |
|--------|------|------|
| 🔴 高 | OpenAI API key 填入 .env.local | 待操作 |
| 🔴 高 | /ai-generators 页面等待 Google 收录 | 等待中 |
| 🟡 中 | Blog 内容生成脚本重新运行 | 待执行 |
| 🟡 中 | /compare 和 /alternatives 内容生成 | 待执行 |
| 🟢 低 | /features 页面内容丰富 | 已有基础版 |

---

## 常用命令

```bash
# 本地开发
npm run dev

# TypeScript 检查
npx tsc --noEmit --skipLibCheck

# 生成 Blog 内容
node scripts/generate-blogs-bulk.mjs

# 生成 Compare 内容
node scripts/generate-comparisons.mjs

# 生成 Alternatives 内容
node scripts/generate-alternatives.mjs

# 检查 generator 数量
node -e "
const {readFileSync} = require('fs');
const env = readFileSync('.env.local','utf-8');
env.split('\n').forEach(l => { const i=l.indexOf('='); if(i>0) process.env[l.slice(0,i).trim()]=l.slice(i+1).trim(); });
const url = process.env.NEXT_PUBLIC_SUPABASE_URL;
const key = process.env.SUPABASE_SERVICE_ROLE_KEY;
fetch(url+'/rest/v1/generators?select=slug&is_active=eq.true',
  {headers:{apikey:key,Authorization:'Bearer '+key}})
  .then(r=>r.json()).then(d=>console.log('Active generators:', d.length));
"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brave88868) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
