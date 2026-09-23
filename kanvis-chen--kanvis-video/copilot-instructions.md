## kanvis-video

> - This is a mandatory gate for every feature, refactor, infrastructure change, and substantial bug fix. Do not begin implementation until the open-source review below is complete.

# Kanvis Studio development guidance

## Open-source-first implementation

- This is a mandatory gate for every feature, refactor, infrastructure change, and substantial bug fix. Do not begin implementation until the open-source review below is complete.
- Before implementing a feature, audit maintained open-source projects and libraries that already solve the same problem or expose a suitable extension point.
- Prefer adopting a dependency, wrapping an upstream API, or composing proven components over reimplementing equivalent behavior locally.
- Evaluate candidates for license compatibility, maintenance activity, security history, runtime and bundle cost, API fit, and long-term replaceability.
- Preserve all required attribution, copyright notices, and license files when third-party work is adopted.
- Do not copy code with an unclear license, or incorporate GPL/AGPL code into this project, without explicit approval and a license-compliance review.
- Record important adopted and rejected alternatives, with reasons, in an ADR or the feature's implementation notes.
- Write custom code only when no suitable maintained implementation meets the requirements. Document the specific capability, integration, or quality gap that requires custom work.
- Keep integrations adapter-based. HyperFrames remains the primary native engine, and adapters should preserve each engine's native project artifacts instead of performing lossy rewrites whenever possible.

### Required workflow for every development task

1. Define the capability and constraints before searching for implementations.
2. Search for maintained open-source projects, standards, libraries, and extension points that address the capability.
3. Shortlist the strongest candidates and compare license, maintenance, security, API fit, performance, bundle/runtime cost, platform support, and replaceability.
4. Choose one of these outcomes explicitly:
   - adopt an existing dependency;
   - wrap an upstream API behind a Kanvis adapter;
   - compose existing primitives;
   - implement custom code because documented gaps make the alternatives unsuitable.
5. Record the adopted and rejected options, links, versions, licenses, and decision reasons in an ADR or the feature implementation notes before merging.
6. Preserve attribution and license notices, pin intentional versions, and add tests around the integration boundary.

### Enforcement

- Plans and implementation summaries must include an `Open-source review` section, even when the conclusion is that no suitable project exists.
- Pull requests must identify any new dependency, its license, its purpose, and the adapter or replacement boundary.
- Do not copy source code from examples, repositories, blog posts, or generated snippets unless its license and attribution requirements are known and satisfied.
- Do not introduce GPL, AGPL, SSPL, Commons Clause, or other source-available/restrictive dependencies without explicit approval and a documented license review.
- Prefer small, replaceable integrations over framework-wide coupling. Business logic and project data contracts must not depend directly on one vendor's proprietary schema when an adapter boundary is practical.
- If the open-source review is missing, stop implementation and complete it first.

## 开源安全与隐私红线

> 适用于本仓库的所有提交、Release、Issue、PR、示例文件、截图、日志和构建产物。未完成检查,不得推送或公开发布。

### 1. 密钥与凭据禁止入库
- API Key、Access Token、Cookie、Session、密码、私钥、签名 URL、OAuth 凭据、云服务配置、真实供应商账号信息不得进入仓库。
- 凭据必须通过运行时环境变量(`process.env.*`)读取,不得硬编码。
- `.env` 文件禁止入库(已由 `.gitignore` 排除);仅 `.env.example` 可提交,且不得含真实值。

### 2. 本机与身份信息禁止泄露
- 电脑用户名、设备名、本机绝对路径(`C:\Users\...`、`E:\...`)、浏览器资料、手机号、身份证、私人邮箱、微信号不得进入源码、注释、文档或配置。
- 确需对外展示的商务二维码或公开联系方式,必须由用户明确批准。
- 文档中引用路径时使用相对路径或占位符(如 `<project-root>`、`~/.codex/`)。

### 3. 客户与生物特征资料禁止公开
- 客户素材、聊天记录、后台截图、订单与店铺信息、语音、肖像、数字人源文件、项目日志和数据库不得作为示例、fixture 或文档提交。
- 测试 fixture 必须是合成数据,不得包含真实客户数据。

### 4. 第三方权益保护
- 引用第三方代码必须保留原作者版权声明和许可证文本,并在 `THIRD_PARTY_NOTICES.md` 中登记。
- 不得复制许可证不明确、或 GPL/AGPL/SSPL/Commons Clause 等限制性许可证的代码(与上方 open-source-first 规则一致)。
- 不得在产品名、派生物、营销材料中冒用他人注册商标;`Kanvis` 名称和 logo 是商标,他人未经许可不得在产品名或派生物中使用。
- 商标、品牌名引用仅用于如实陈述兼容性(如"兼容 Codex"),不得暗示官方背书。

### 5. Kanvis 版权与商标
- 所有新增的关键源码文件(`.mjs`、`.ts`、`.tsx`)必须在文件头部保留版权声明:`Kanvis Video Copyright (c) 2026 Kanvis Chen MIT License - https://opensource.org/licenses/MIT`
- `package.json` 在开源分发版本中不得包含 `"private": true`。
- 根目录必须维护 `THIRD_PARTY_NOTICES.md`,列出所有依赖及其许可证和版权归属。

### 6. 生成物与缓存
- `dist/`、`release/`、`.visualhyper/`、`*.tsbuildinfo`、Source Map、日志、缓存、崩溃报告不得入库(已由 `.gitignore` 排除)。
- 副本目录 `visualhyper-codex-panel/` 含本机路径和商业文档,禁止入库(已由 `.gitignore` 排除)。
- ts-rs 等工具生成的代码(`packages/codex-client/src/generated/`)不入库,保留生成器配置和源定义即可。

### 7. AI 推送门禁
- AI 在执行 `git push`、创建公开仓库或发布 Release 前,必须报告扫描范围、结果和残余风险。
- 发现未解决的高风险问题(凭据泄露、本机路径、客户资料、限制性许可证代码)时,禁止推送。
- 私有仓库备份同样适用本红线;后续若转为公开仓库,必须重新执行完整扫描。

### 8. 发布前双重扫描
- 先扫描当前待提交文件和二进制元数据,再扫描完整 Git 历史。
- 至少执行仓库自带发布审计、GitHub Secret Scanning / Push Protection 和独立密钥扫描。
- 发现泄露立即停止发布:先撤销或轮换凭据,再从当前文件和 Git 历史中清除;只删除最新版本不算完成整改。

---
> Source: [Kanvis-chen/kanvis-video](https://github.com/Kanvis-chen/kanvis-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
