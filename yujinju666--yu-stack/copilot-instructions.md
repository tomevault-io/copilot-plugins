## yu-stack

> **来源**: [github.com/MarcusRawlins/agency-agents](https://github.com/MarcusRawlins/agency-agents)

# 🎭 智能体系统参考手册

## 系统一：The Agency — 144个专家角色

**来源**: [github.com/MarcusRawlins/agency-agents](https://github.com/MarcusRawlins/agency-agents)
**位置**: Claude Code（右屏）
**激活方式**: 在Claude Code中说 "激活前端开发者模式"

### 12大部门速查

```
The Agency (总部)
├── 🔧 工程部门 - 24个角色
├── 🎨 设计部门 - 9个角色
├── 📢 营销部门 - 30+个角色
├── 💰 销售部门 - 8个角色
├── 💳 付费广告部门 - 8个角色
├── 📦 产品部门 - 12个角色
├── 📊 项目管理部门 - 10个角色
├── 🧪 测试与质量部门 - 8个角色
├── 🎮 游戏开发部门 - 10个角色
├── 🎓 学术部门 - 8个角色
├── 💼 专业服务部门 - 10个角色
└── 🛠️ 客户支持部门 - 7个角色
```

### 常用激活示例

```
"激活前端开发者模式"
"切换到后端架构师模式"
"激活UI设计师模式，帮我设计一个电商产品页面"
"切换到质量工程师模式，为这个功能编写测试"
"激活安全工程师模式，审查这个合并请求"
"激活增长黑客模式，分析增长策略"
"切换到小红书专家模式，写一篇种草文案"
```

---

## 系统二：gstack — 43个开发工作流

**来源**: [github.com/garrytan/gstack](https://github.com/garrytan/gstack)
**位置**: Hermes（左屏）
**作者**: Garry Tan (YC CEO)

### 规划与策略
| 工作流 | 用途 | 前置条件 |
|--------|------|---------|
| `/office-hours` | 产品创意重构 | 任何新想法 |
| `/plan-ceo-review` | CEO级战略审核 | 功能/产品方向 |
| `/plan-eng-review` | 架构评审 | 方案设计完成后 |
| `/plan-design-review` | 设计评审 | 设计方案后 |
| `/plan-devex-review` | 开发者体验审核 | 应用编程接口/命令行/软件开发工具包设计时 |
| `/autoplan` | 自动跑所有评审 | 完整规划 |

### 设计与实现
| 工作流 | 用途 |
|--------|------|
| `/design-consultation` | 从零搭建设计系统 |
| `/design-review` | 视觉设计审计+修复 |
| `/design-html` | 超文本标记语言/层叠样式表设计实现 |
| `/design-shotgun` | 多方向设计探索 |

### 开发流程
| 工作流 | 用途 |
|--------|------|
| `/review` | 合并前代码审查 |
| `/investigate` | 系统性调试（4个阶段） |
| `/codex` | 第二意见 |
| `/browse` | 无头浏览器控制 |
| `/pair-agent` | 结对编程 |

### 测试与质量
| 工作流 | 用途 |
|--------|------|
| `/qa` | 浏览器端质量测试+修复 |
| `/qa-only` | 质量测试报告 |
| `/benchmark` | 性能测试 |
| `/health` | 代码质量仪表盘 |

### 安全
| 工作流 | 用途 |
|--------|------|
| `/cso` | 安全审计（OWASP十大风险+STRIDE威胁建模） |
| `/setup-browser-cookies` | 浏览器Cookie导入 |

### 部署
| 工作流 | 用途 |
|--------|------|
| `/ship` | 完整发版 |
| `/land-and-deploy` | 合并+部署+验证 |
| `/canary` | 上线后监控 |
| `/setup-deploy` | 部署配置 |
| `/document-release` | 文档发布 |

### 运维协作
| 工作流 | 用途 |
|--------|------|
| `/retro` | 周会回顾 |
| `/learn` | 学习记录管理 |
| `/context-save` | 进度保存 |
| `/context-restore` | 进度恢复 |
| `/plan-tune` | 问题灵敏度调节 |
| `/gstack-upgrade` | 自身升级 |

### 安全控制
| 工作流 | 用途 |
|--------|------|
| `/careful` | 危险操作警告 |
| `/freeze` | 锁定目录编辑 |
| `/guard` | 双重保护 |
| `/unfreeze` | 解除锁定 |

---

## 协作工作流

```
你提出需求
    │
    ▼
┌── Hermes（分析）───────────────────────────────────┐
│  /office-hours → 理解需求背景                      │
│  /plan-ceo-review → 评估价值方向                    │
│  /plan-eng-review → 确定技术架构                    │
│  输出: 方案文档 → 桌面/共享文件/                    │
└────────────────────────────────────────────────────┘
    │
    ▼
┌── Claude Code（实现）────────────────────────────────┐
│  "激活全栈工程师模式"                                │
│  读取方案 → 编码实现 → 测试                         │
│  输出: 代码 → 桌面/共享文件/                        │
└────────────────────────────────────────────────────┘
    │
    ▼
┌── Hermes（整合）────────────────────────────────────┐
│  扫描桌面/共享文件/ 的新输出                         │
│  分类 → 集成到技能/记忆                              │
│  更新工作区状态                                     │
└────────────────────────────────────────────────────┘
```

---

## 工作流模板

### 完整项目启动
```
1. yu --task "构建一个XX系统"
2. yu → 启动协作终端
3. Hermes: /office-hours → 理解需求
4. Hermes: /plan-ceo-review → 战略评估
5. Hermes: 输出方案到共享文件
6. Claude Code: 读取方案 → 编码
7. Hermes: /review 审查代码
8. Claude Code: 修复问题
9. Hermes: /qa 测试
10. Hermes: /ship 发版
```

### 修复Bug
```
1. yu --task "#bug XX功能报错"
2. Hermes: /investigate → 根因分析
3. Hermes: 输出根因报告
4. Claude Code: 修复代码
5. Hermes: 验证修复
6. Hermes: /learn 记录教训
```

### 安全审计
```
1. yu --task "安全审计当前项目"
2. Hermes: /cso → OWASP十大风险+STRIDE威胁建模
3. Hermes: 输出审计报告
4. Claude Code: 修复漏洞
5. Hermes: 验证修复
```

---
> Source: [yujinju666/yu-stack](https://github.com/yujinju666/yu-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
