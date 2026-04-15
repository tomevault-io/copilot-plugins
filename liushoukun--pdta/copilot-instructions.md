## pdta

> 业务系统文档架构规范 - 定义文档目录结构和组织方式


# 业务系统文档架构规范

## 📁 文档根目录结构

```
docs/
├── product/          # 产品文档（产品经理维护）
├── user/             # 用户手册（面向最终用户）
├── requirement/      # 需求管理（产品+研发+测试）
├── technical/        # 技术文档（技术团队维护）
├── operations/       # 运维文档（运维团队维护）
├── training/         # 培训资料（各角色培训）
├── project/          # 项目管理（项目经理维护）
├── knowledge/        # 知识库（团队沉淀）
├── index.md          # VitePress文档首页
└── CHANGELOG.md      # 系统更新日志
```

## 📱 product/ - 产品文档

**维护者**：产品经理  
**目的**：说明产品定位、业务流程、功能规划

```
product/
├── index.md                  # 产品文档首页
├── overview.md               # 产品概述（系统定位/目标用户/核心价值）
├── business-process/         # 业务流程（含流程图）
│   ├── index.md              # 业务流程概述
│   ├── user-registration.md
│   ├── order-process.md
│   └── payment-process.md
├── features/                 # 功能列表
│   ├── index.md              # 功能列表首页
│   ├── feature-list.md       # 功能清单
│   └── [module]-feature.md   # 各模块功能说明
├── roadmap/                  # 产品规划
│   ├── index.md              # 产品规划概述
│   ├── current-version.md
│   ├── next-version.md
│   └── backlog.md
├── release-notes/            # 版本发布说明
│   ├── index.md              # 版本发布说明首页
│   └── vX.X.X.md
└── wireframes/               # 原型设计
    └── index.md              # 原型设计说明
```

**命名规范**：
- 业务流程：使用动词短语，如 `user-registration.md`
- 功能说明：使用模块名，如 `user-management.md`
- 版本说明：使用版本号，如 `v1.0.0.md`

## 👤 user/ - 用户手册

**维护者**：产品经理 + 技术文档工程师  
**目的**：帮助最终用户快速上手和使用系统

```
user/
├── index.md                  # 用户手册首页
├── quick-start.md            # 快速入门（5分钟上手）
├── login-register.md         # 登录注册
├── guides/                   # 按角色分类的操作指南
│   ├── index.md              # 操作指南概述
│   ├── admin/                # 管理员操作
│   │   ├── index.md          # 管理员操作概述
│   │   ├── user-management.md
│   │   ├── role-management.md
│   │   └── system-settings.md
│   ├── operator/             # 运营人员操作
│   │   ├── index.md          # 运营操作概述
│   │   ├── content-management.md
│   │   └── order-handling.md
│   └── customer-service/     # 客服操作
│       ├── index.md          # 客服操作概述
│       └── ticket-handling.md
├── faq.md                    # 常见问题
├── troubleshooting.md        # 问题排查
└── videos/                   # 视频教程链接
    └── index.md              # 视频教程目录
```

**编写要求**：
- ✅ 大量使用截图和步骤说明
- ✅ 使用序号列表描述操作步骤
- ✅ 突出关键按钮和字段
- ✅ 提供实际业务场景示例

## 📋 requirement/ - 需求管理（核心）

**维护者**：产品经理（PRD）、技术负责人（TDD）、测试工程师（TEST）  
**目的**：管理需求全生命周期，保证需求可追溯

```
requirement/
├── index.md                  # 需求管理说明
├── backlog/                  # 需求池
│   ├── index.md              # 需求池概述
│   ├── pending.md            # 待评估需求
│   ├── approved.md           # 已批准待排期
│   └── rejected.md           # 已拒绝需求
├── template/                 # 文档模板
│   ├── index.md              # 模板使用说明
│   ├── PRD-template.md
│   ├── TDD-template.md
│   └── TEST-template.md
├── YYYYMM-需求名称/          # 需求文件夹
│   ├── index.md              # 需求概述
│   ├── product/              # 产品需求
│   │   ├── PRD.md
│   │   └── attachments/      # 原型图、流程图
│   ├── technical/            # 技术设计
│   │   ├── TDD.md
│   │   ├── database.md       # 数据库设计
│   │   ├── api-design.md     # 接口设计
│   │   └── ui-design.md      # UI设计
│   ├── testing/              # 测试文档
│   │   ├── test-cases.md     # 测试用例
│   │   └── test-report.md    # 测试报告
│   └── review/               # 评审记录
│       ├── prd-review.md     # PRD评审记录
│       ├── tdd-review.md     # TDD评审记录
│       ├── code-review.md    # 代码审查报告
│       └── acceptance.md     # 验收记录
└── archives/                 # 历史需求归档
    ├── index.md              # 归档说明
    └── YYYY/
        └── index.md          # 年度归档概述
```

**命名规范**：
- 需求文件夹：`YYYYMM-需求简短名称`，如 `202602-用户登录注册`
- 需求简短名称：中文，不超过10个字，使用连字符分隔
- 评审记录：包含评审日期和结论

**文档版本管理**：
```markdown
---
version: 1.2.0
created: 2026-02-09
updated: 2026-02-15
author: 张三
status: draft | review | approved | development | testing | published
---
```

## 🔧 technical/ - 技术文档

**维护者**：技术团队  
**目的**：说明系统架构、接口设计、开发规范

```
technical/
├── index.md                  # 技术文档首页
├── architecture/             # 系统架构
│   ├── index.md              # 架构概览（含架构图）
│   ├── overview.md           # 架构详细说明
│   ├── system-design.md      # 系统设计
│   ├── tech-stack.md         # 技术栈
│   ├── frontend-architecture.md
│   ├── backend-architecture.md
│   └── security.md           # 安全设计
├── database/                 # 数据库文档
│   ├── index.md              # 数据库概览
│   ├── overview.md           # 数据库详细说明
│   ├── er-diagram.md         # ER图（使用Mermaid）
│   ├── tables/               # 表结构文档
│   │   ├── index.md          # 表结构概述
│   │   ├── user-tables.md    # 用户相关表
│   │   └── [module]-tables.md
│   ├── migrations.md         # 数据迁移记录
│   └── performance.md        # 性能优化
├── api/                      # 接口文档
│   ├── index.md              # 接口文档首页
│   ├── overview.md           # 接口概览
│   ├── authentication.md     # 认证鉴权
│   ├── modules/              # 按模块分类
│   │   ├── index.md          # 接口模块概述
│   │   ├── user-api.md
│   │   └── [module]-api.md
│   ├── error-codes.md        # 错误码说明
│   └── changelog.md          # 接口变更记录
├── development/              # 开发文档
│   ├── index.md              # 开发文档首页
│   ├── setup.md              # 开发环境搭建
│   ├── code-style.md         # 代码规范
│   ├── git-workflow.md       # Git工作流
│   ├── testing.md            # 测试规范
│   └── build-deploy.md       # 构建部署
├── third-party/              # 第三方集成
│   ├── index.md              # 第三方集成概述
│   ├── payment.md
│   ├── sms.md
│   └── oss.md
└── adr/                      # 架构决策记录
    ├── index.md              # ADR索引
    └── NNNN-decision-title.md
```

**架构决策记录（ADR）格式**：
```markdown
# ADR-0001: 选择React作为前端框架

## 状态
已批准 (2026-02-09)

## 背景
需要选择前端框架...

## 决策
选择React...

## 后果
### 优点
- ...

### 缺点
- ...
```

## 🚀 operations/ - 运维文档

**维护者**：运维团队  
**目的**：指导系统部署、监控、维护和应急处理

```
operations/
├── index.md                  # 运维文档首页
├── deployment/               # 部署文档
│   ├── index.md              # 部署概览
│   ├── overview.md           # 部署详细说明
│   ├── environment.md        # 环境说明（开发/测试/生产）
│   ├── prerequisites.md      # 前置条件
│   ├── docker-deploy.md      # Docker部署
│   ├── k8s-deploy.md         # Kubernetes部署
│   └── upgrade.md            # 升级指南
├── monitoring/               # 监控告警
│   ├── index.md              # 监控概览
│   ├── overview.md           # 监控详细说明
│   ├── metrics.md            # 监控指标
│   ├── alerts.md             # 告警规则
│   ├── dashboard.md          # 监控大盘
│   └── logging.md            # 日志管理
├── maintenance/              # 运维手册
│   ├── index.md              # 运维手册概述
│   ├── daily-tasks.md        # 日常巡检
│   ├── database-backup.md    # 数据库备份
│   ├── data-recovery.md      # 数据恢复
│   └── performance-tuning.md # 性能调优
├── runbook/                  # 应急手册（SOP）
│   ├── index.md              # 应急手册概述
│   ├── incident-response.md  # 故障响应流程
│   ├── service-down.md       # 服务宕机处理
│   ├── database-issues.md    # 数据库问题
│   └── security-incidents.md # 安全事件
└── sop/                      # 标准操作流程
    ├── index.md              # SOP概述
    ├── release-sop.md        # 发布流程
    ├── rollback-sop.md       # 回滚流程
    └── data-migration-sop.md # 数据迁移流程
```

**SOP文档结构**：
```markdown
# [操作名称] SOP

## 适用场景
...

## 前置条件
- [ ] 条件1
- [ ] 条件2

## 操作步骤
1. 步骤1
2. 步骤2

## 验证检查
- [ ] 检查项1
- [ ] 检查项2

## 回滚方案
...

## 注意事项
⚠️ 警告信息
```

## 📚 training/ - 培训资料

**维护者**：各角色负责人  
**目的**：新员工培训和团队能力提升

```
training/
├── index.md                  # 培训资料说明
├── onboarding/               # 新员工培训
│   ├── index.md              # 新员工培训概述
│   ├── system-overview.md
│   ├── business-process.md
│   └── tool-usage.md
├── product/                  # 产品培训
│   └── index.md              # 产品培训概述
├── development/              # 开发培训
│   └── index.md              # 开发培训概述
├── testing/                  # 测试培训
│   └── index.md              # 测试培训概述
└── operations/               # 运维培训
    └── index.md              # 运维培训概述
```

## 📊 project/ - 项目管理

**维护者**：项目经理  
**目的**：项目协作、流程规范、进度跟踪

```
project/
├── index.md                  # 项目管理说明
├── team.md                   # 团队成员和职责
├── process/                  # 流程规范
│   ├── index.md              # 流程规范概述
│   ├── requirement-process.md
│   ├── development-process.md
│   ├── testing-process.md
│   ├── release-process.md
│   └── review-checklist.md
├── milestones/               # 里程碑
│   ├── index.md              # 里程碑概述
│   └── YYYY-QN.md
├── meetings/                 # 会议记录
│   ├── index.md              # 会议记录索引
│   └── YYYY-MM-DD-meeting-topic.md
└── reports/                  # 项目报告
    ├── index.md              # 项目报告索引
    ├── weekly-report.md
    └── monthly-report.md
```

## 💡 knowledge/ - 知识库

**维护者**：全体团队成员  
**目的**：沉淀团队经验和最佳实践

```
knowledge/
├── index.md                  # 知识库说明
├── best-practices/           # 最佳实践
│   ├── index.md              # 最佳实践概述
│   ├── frontend-practices.md
│   ├── backend-practices.md
│   └── database-practices.md
├── case-study/               # 案例分析
│   ├── index.md              # 案例分析索引
│   ├── performance-optimization.md
│   ├── bug-fix-case.md
│   └── refactoring-case.md
├── troubleshooting/          # 问题排查
│   ├── index.md              # 问题排查指南
│   ├── common-errors.md
│   └── debugging-guide.md
└── glossary.md               # 术语表
```

## 📝 通用规范

### 1. index.md规范

每个目录必须包含 `index.md` 作为该目录的首页：

```markdown
---
title: [目录名称]
---

# [目录名称]

## 📖 目录说明
简要说明本目录的作用和内容

## 📁 目录结构
- `file1.md` - 文件说明
- `file2.md` - 文件说明

## 🔗 快速链接
- [链接1](./path/to/file.md)
- [链接2](./path/to/file.md)

## 👥 维护者
- 负责人：张三
- 更新频率：每周/每月
```

**说明**：
- `index.md` 作为目录首页，VitePress 会自动将其作为该目录的默认页面
- 使用 frontmatter 中的 `title` 字段作为侧边栏菜单的显示名称
- `index.md` 在侧边栏中不会单独显示，而是作为目录项的默认链接

### 2. 文件命名规范

- ✅ 使用小写字母
- ✅ 使用连字符分隔单词：`user-management.md`
- ✅ 使用有意义的名称
- ❌ 避免使用：空格、下划线、特殊字符
- ❌ 避免使用：`doc1.md`、`新建文件.md`

**示例**：
```
✅ user-registration-process.md
✅ api-design.md
✅ 202602-用户登录.md

❌ UserRegistration.md
❌ API_Design.md
❌ 新建文档.md
```

### 3. 目录命名规范

- ✅ 使用小写字母
- ✅ 使用连字符分隔：`business-process/`
- ✅ 使用复数形式（如果包含多个同类文件）：`features/`、`guides/`
- ❌ 避免深层嵌套（不超过4层）

### 4. 文档内链规范

- ✅ 使用相对路径：`[链接](../api/overview.md)`
- ✅ 使用有意义的链接文本：`[API文档](../api/overview.md)`
- ❌ 避免使用绝对路径
- ❌ 避免使用"点击这里"

### 5. 图片和附件

```
docs/
├── .vitepress/
│   └── public/               # 公共资源
│       ├── images/           # 通用图片
│       │   ├── architecture/ # 架构图
│       │   ├── screenshots/  # 截图
│       │   └── diagrams/     # 流程图
│       └── files/            # 其他文件
└── requirement/
    └── 202602-需求/
        └── product/
            └── attachments/  # 需求相关附件
```

**引用方式**：
```markdown
![架构图](/images/architecture/system-overview.png)
```

### 6. 版本标识

在文档frontmatter中添加版本信息：

```yaml
---
title: 文档标题
version: 1.0.0
created: 2026-02-09
updated: 2026-02-15
author: 张三
reviewers: [李四, 王五]
status: draft | review | approved | published
---
```

## ❌ 反面示例

### 错误的目录结构
```
docs/
├── 文档/              # ❌ 使用中文目录名
├── Doc_1/            # ❌ 使用下划线和数字
├── temp/             # ❌ 临时文件夹存在文档目录
└── backup/           # ❌ 备份文件夹存在文档目录
```

### 错误的文件命名
```
❌ 新建文档.md
❌ Document_V2_Final_最终版.md
❌ doc1.md
❌ 需求文档 - 副本.md
```

### 错误的文档组织
```
requirement/
└── 所有需求文档都放在根目录.md  # ❌ 没有分类组织
```

## ✅ 正确示例

### 需求文档完整结构
```
requirement/202602-用户登录注册/
├── product/
│   ├── PRD.md                      # 产品需求文档
│   └── attachments/
│       ├── login-flow.png          # 登录流程图
│       └── register-wireframe.png  # 注册原型
├── technical/
│   ├── TDD.md                      # 技术设计文档
│   ├── database.md                 # 数据库设计
│   │   └── user-table-schema
│   ├── api-design.md               # 接口设计
│   │   ├── POST /api/login
│   │   ├── POST /api/register
│   │   └── POST /api/logout
│   └── ui-design.md                # UI设计说明
├── testing/
│   ├── test-cases.md               # 测试用例
│   └── test-report.md              # 测试报告
└── review/
    ├── prd-review.md               # PRD评审记录
    ├── tdd-review.md               # TDD评审记录
    ├── code-review.md              # 代码审查报告
    └── acceptance.md               # 验收记录
```

## 🔍 检查清单

创建或修改文档时，请确认：

- [ ] 文件名使用小写字母和连字符
- [ ] 每个目录包含 `index.md` 作为首页
- [ ] `index.md` 的 frontmatter 包含 `title` 字段（用于侧边栏显示）
- [ ] 文档包含 frontmatter（版本、作者、状态）
- [ ] 使用相对路径引用其他文档
- [ ] 图片放在正确的位置
- [ ] 文档结构符合规范
- [ ] 需求文件夹使用 `YYYYMM-需求名称` 格式
- [ ] 重要文档包含变更记录

## 📌 VitePress Sidebar 配置说明

本文档架构适配 `vitepress-sidebar` 插件的以下特性：

### useFolderTitleFromIndexFile

```javascript
{
  useFolderTitleFromIndexFile: true,
  useTitleFromFrontmatter: true
}
```

**工作原理**：
1. VitePress 会读取每个目录下的 `index.md` 文件
2. 从 `index.md` 的 frontmatter 中的 `title` 字段获取菜单名称
3. `index.md` 不会在侧边栏中显示为单独的菜单项
4. 点击目录项时，会自动跳转到该目录的 `index.md` 页面

**示例**：

```markdown
---
title: 产品文档
---

# 产品文档

...
```

上述 `index.md` 会使侧边栏显示"产品文档"作为目录名称。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/liushoukun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
