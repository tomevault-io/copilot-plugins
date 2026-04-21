## awesome-phm-paper-agent

> *APPA系统的Claude Code配置文件*

# 🤖 CLAUDE.md - Claude Code专用配置 [重要,本文件禁止修改]

*APPA系统的Claude Code配置文件*

## 🎯 核心目标 (Core Goals)

**APPA (Awesome PHM Paper Agent)** 是一个智能学术论文管理系统，专注于 **真实学术数据查询**，使用 academic-researcher agent 从真实的学术数据库中发现、分析和组织 PHM 领域的研究论文。

### 🚀 GitHub Awesome 项目 (GitHub Awesome Project)
- **标准格式**: 遵循 Awesome 项目标准，适合开源社区协作
- **自动更新**: 通过 Claude agents 实现定期自动更新论文库
- **社区友好**: 支持贡献者添加新论文和改进分类

### 🔬 真实数据驱动 (Real Data Driven)
- **数据源**: 使用 academic-researcher agent 查询 ArXiv、PubMed、Google Scholar 等真实学术数据库
- **无虚假数据**: 完全杜绝虚假或模拟论文数据，确保所有论文信息来自真实出版物
- **摘要完整性**: 提取并整理论文完整摘要，保持学术严谨性

### 📚 知识库组织 (Knowledge Organization)
- **分类导航**: 按研究主题分类组织（深度学习、故障诊断、RUL预测、数字孪生等）
- **双向链接**: 实现论文间、作者间、主题间的双向引用和导航
- **多维索引**: 支持按年份、期刊、作者、引用数等多维度检索

### 🤖 Agent 驱动更新 (Agent-Driven Updates)
- **academic-researcher**: 🔥 **主要文献搜索引擎** - 集成PHM专业知识的增强版学术研究代理
- **质量过滤系统**: 🚫 **MDPI出版商过滤** + **影响因子≥5** + **期刊分区控制**
- **定期更新**: 自动调用 agents 发现最新研究成果
- **向后兼容**: `phm-paper-discovery` 自动重定向到增强版 `academic-researcher`

## 🚀 快速命令

| 命令 | 功能 | 脚本 |
|------|------|------|
| **每日问候** | 显示系统状态和推荐 | `./scripts/daily_greeting.sh` |
| **搜索论文** | 按条件查找论文 | `./scripts/search_papers.sh --help` |
| **显示所有论文** | 浏览完整论文库 | `./scripts/search_papers.sh --all` |
| **系统状态** | 查看统计信息 | `python main.py --status` |

## 📋 Claude Code工作流

### 1. 每日启动流程
```bash
# Claude可以直接执行
./scripts/daily_greeting.sh
```

### 2. 论文搜索示例
```bash
# 按年份搜索
./scripts/search_papers.sh --year 2024

# 按作者搜索  
./scripts/search_papers.sh --author zhang

# 按主题搜索
./scripts/search_papers.sh --topic deep-learning-phm

# 按关键词搜索
./scripts/search_papers.sh --keyword "bearing fault"
```

### 3. 内容管理
```bash
# 查看目录结构
ls -la papers/ topics/ authors/ venues/

# 统计信息
find papers -name "index.md" | wc -l
```

## 🛠️ Claude Code工具配置

### 📚 Paper Discovery Agent
**工具**: Bash, WebFetch, Grep, Write, TodoWrite
```bash
# API查询示例
curl -s "https://api.openalex.org/works?search=bearing+fault+diagnosis" | jq '.results[0]'
```

### 🌟 Daily Greeting Agent  
**工具**: Read, Bash, Write, WebSearch
```bash
# 每日问候
echo "🌅 $(date '+%A, %B %d') - PHM Daily Update"
```

### 🔍 Quality Curation Agent
**工具**: Read, Edit, WebFetch, Grep
```bash
# 质量检查
grep -r "引用数" papers/ | grep -v "0$"
```

### 📝 Content Analysis Agent
**工具**: Read, Write, Task, WebSearch  
```bash
# 分析摘要
head -20 papers/*/index.md | grep -A 5 "TL;DR"
```

### 🔗 Smart Linking Agent
**工具**: Edit, Grep, MultiEdit, Glob
```bash
# 查找链接
grep -r "\.md)" papers/ | head -5
```

## 📊 用户交互模式

### 🗣️ 自然语言命令映射
| 用户说话 | Claude执行 |
|----------|------------|
| "早上好，有什么新论文吗？" | `./scripts/daily_greeting.sh` |
| "搜索深度学习相关论文" | `./scripts/search_papers.sh --topic deep-learning-phm` |
| "显示2024年的所有论文" | `./scripts/search_papers.sh --year 2024` |
| "查找张伟的论文" | `./scripts/search_papers.sh --author zhang` |
| "最近一周有什么更新？" | `./scripts/search_papers.sh --recent 7` |

### 📈 智能响应模板
```markdown
Claude响应格式：
1. 🎯 直接回答用户问题
2. 📊 提供相关统计信息  
3. 🔗 给出相关链接
4. 💡 建议下一步操作
```

## 🔧 系统配置

### 📂 目录结构
```
APPA/
├── papers/YYYY/YYYY-VENUE-Author-Title/
│   ├── index.md          # 论文详情页
│   └── refs.bib          # BibTeX引用
├── topics/topic-name/
│   └── README.md         # 主题概览
├── authors/author-name/
│   └── README.md         # 作者资料
├── venues/venue-name/
│   └── README.md         # 期刊信息
├── indices/              # 各种索引
├── scripts/              # Claude Code脚本
└── logs/                 # 日志和状态
```

### 🔗 链接格式规范
```markdown
# GitHub友好的相对路径链接
[论文标题](../papers/2024/2024-MSSP-Zhang-DeepLearning/index.md)
[主题页面](../topics/deep-learning-phm/README.md) 
[作者页面](../authors/zhang-wei/README.md)
[期刊页面](../venues/mssp/README.md)
```

### 📅 自动化调度
```bash
# 每日9点执行问候
0 9 * * * cd ~/APPA && ./scripts/daily_greeting.sh

# 每周日执行全量更新
0 1 * * 0 cd ~/APPA && python main.py --incremental
```

## 🎯 Claude Code执行模式

### 模式1: 交互式对话
```
用户: "帮我找一下轴承故障诊断的论文"
Claude: [执行] ./scripts/search_papers.sh --keyword "bearing fault"
        [分析] 找到1篇相关论文
        [展示] 论文链接和摘要
        [建议] 是否需要查看详细分析？
```

### 模式2: 主动服务
```
Claude: [定时执行] ./scripts/daily_greeting.sh
        [检测] 发现新增论文
        [通知] 主动推荐给用户
        [等待] 用户进一步指令
```

### 模式3: 深度分析
```
用户: "分析一下深度学习在PHM中的发展趋势"
Claude: [读取] topics/deep-learning-phm/README.md
        [统计] 论文数量和引用情况
        [分析] 技术发展脉络
        [生成] 趋势分析报告
```

## 🔍 调试和监控

### 📊 状态检查
```bash
# 检查系统状态
ls -la logs/
cat logs/stats.json
tail -20 logs/appa.log
```

### 🐛 常见问题
1. **脚本权限**: `chmod +x scripts/*.sh`
2. **路径问题**: 确保在APPA根目录执行
3. **依赖检查**: `python main.py --status`

## 🤖 Claude专用PHM研究Agents

### 🎯 专业PHM研究助手

APPA系统现已优化为**5+1**个专门的Claude Agents，以`academic-researcher`为核心：

#### 📚 **academic-researcher** - 🔥 **主要文献搜索引擎** (增强版)
```
用法示例:
"使用academic-researcher搜索最新的轴承故障诊断论文"
"让academic-researcher找一些Q1期刊的深度学习PHM研究，排除MDPI"
"用academic-researcher获取影响因子≥5的预测性维护文献"
```
**🆕 核心功能**:
- ✅ **PHM专业知识**: 集成原phm-paper-discovery所有功能并增强
- ✅ **质量过滤**: MDPI黑名单、影响因子≥5、期刊分区筛选
- ✅ **双模式**: 自动检测PHM领域查询vs通用学术查询
- ✅ **向后兼容**: `phm-paper-discovery`调用自动重定向

**输出**: 高质量论文列表、质量分级、相关性评分、统计分析、过滤报告

#### ⭐ **phm-quality-curator** - PHM论文质量评估专家
```
用法示例:
"让phm-quality-curator筛选出Q1期刊的PHM论文"
"用phm-quality-curator评估这批论文的学术价值"
"请phm-quality-curator过滤掉低质量的预印本论文"
```
**专长**: 多维度质量评估、期刊影响力分析、创新性判断、实验验证评估
**输出**: 质量分级、筛选报告、改进建议、质量趋势分析

#### 📝 **phm-content-analyzer** - PHM论文内容深度分析专家
```
用法示例:
"用phm-content-analyzer深度分析这篇Transformer PHM论文"
"让phm-content-analyzer生成论文的中英文TL;DR摘要"
"请phm-content-analyzer评估论文的工程实用价值"
```
**专长**: 5层分析架构、技术创新识别、应用价值评估、可重现性分析
**输出**: TL;DR摘要、核心贡献、技术分析、影响评估、实用建议

#### 📁 **phm-knowledge-organizer** - PHM知识库组织专家
```
用法示例:
"让phm-knowledge-organizer整理这些论文为标准知识库"
"用phm-knowledge-organizer创建多维度的论文索引系统"
"请phm-knowledge-organizer生成GitHub友好的导航结构"
```
**专长**: 标准化目录结构、多维度索引、BibTeX生成、GitHub优化
**输出**: 结构化知识库、交叉索引、导航系统、统计报告

#### 🔗 **phm-relationship-builder** - PHM研究关系网络构建专家
```
用法示例:
"用phm-relationship-builder构建论文间的关系网络"
"让phm-relationship-builder分析PHM技术发展脉络"
"请phm-relationship-builder建立智能交叉引用系统"
```
**专长**: 多维度关系分析、网络中心性计算、技术演进追踪、智能导航
**输出**: 关系网络图、影响力分析、演进路径、导航链接

#### 🎯 **phm-research-coordinator** - PHM研究协调专家 (主控制器)
```
用法示例:
"让phm-research-coordinator为我建立完整的深度学习PHM文献库"
"用phm-research-coordinator分析轴承故障诊断的研究现状"
"请phm-research-coordinator建立自动化的PHM研究跟踪系统"
```
**专长**: 工作流编排、子agent管理、智能决策、质量控制
**输出**: 完整研究报告、趋势分析、实用建议、系统性洞察

### 🚀 使用场景示例

#### **完整研究流程**
```
用户: "帮我建立一个关于Transformer在PHM中应用的完整研究体系"

Claude: 我将使用phm-research-coordinator来协调整个研究流程：
1. 首先用phm-paper-discovery搜索相关论文
2. 然后用phm-quality-curator筛选高质量研究
3. 接着用phm-content-analyzer深度分析内容
4. 再用phm-knowledge-organizer构建知识库
5. 最后用phm-relationship-builder建立关系网络
```

#### **专项任务调用**
```
用户: "这篇论文的主要贡献是什么？"
Claude: 让我使用phm-content-analyzer来分析这篇论文的核心贡献...

用户: "帮我找一些高质量的RUL预测论文"
Claude: 我先用phm-paper-discovery搜索，再用phm-quality-curator筛选...

用户: "整理一下我的PHM论文收藏"
Claude: 我使用phm-knowledge-organizer为您创建标准化的知识库结构...
```

### 📊 Agent能力对比 (v2.0)

| Agent | 主要功能 | 输入类型 | 输出格式 | 使用频率 | 状态 |
|-------|----------|----------|----------|----------|------|
| **academic-researcher** | 🔥 **文献搜索+质量过滤** | 关键词、领域、质量要求 | 高质量论文列表+分析 | 🔴 **最高** | ✅ **增强版** |
| ~~paper-discovery~~ | ~~论文搜索发现~~ | ~~关键词、领域~~ | ~~论文列表+分析~~ | - | ⚠️ **已合并** → `academic-researcher` |
| quality-curator | 质量评估筛选 | 论文列表 | 质量报告+建议 | 高 | ✅ **独立** |
| content-analyzer | 内容深度分析 | 单篇论文 | 多层分析报告 | 中 | ✅ **独立** |
| knowledge-organizer | 知识库组织 | 论文集合 | 结构化知识库 | 中 | ✅ **独立** |
| relationship-builder | 关系网络构建 | 知识库 | 关系图+导航 | 低 | ✅ **独立** |
| research-coordinator | 流程协调管理 | 研究需求 | 完整解决方案 | 高 | ✅ **独立** |

## 🔬 Academic Researcher Agent 集成

### 🎯 核心功能
**academic-researcher** 是系统的核心数据源，负责从真实学术数据库查询PHM相关论文：

```bash
# 调用示例
用户: "查找2024年关于深度学习PHM的论文"
Claude: 我使用 academic-researcher agent 从ArXiv和Google Scholar搜索最新研究...

用户: "找一些轴承故障诊断的经典论文"  
Claude: 让我通过 academic-researcher 查询高引用的轴承诊断研究...
```

### 📊 数据流程 (真实数据管道)
```
1. 关键词输入 → academic-researcher agent
2. 真实数据库查询 (ArXiv, PubMed, Google Scholar) 
3. 论文元数据提取 (标题、作者、摘要、DOI等)
4. 质量验证与分类
5. 知识库组织与双向链接
6. GitHub Awesome 项目格式输出
```

### 🛠️ 技术实现要点
- **Task工具调用**: 使用Task tool调用academic-researcher subagent
- **JSON解析**: 解析返回的标准学术论文JSON格式  
- **摘要完整性**: 确保提取完整的论文摘要和关键信息
- **去重机制**: 基于DOI和论文指纹进行智能去重
- **分类标签**: 自动为论文生成PHM相关的分类标签

## 💡 扩展建议

### 🚀 Agent增强计划
- **多语言支持**: 增强中文文献的处理能力
- **实时更新**: 建立论文自动跟踪和更新机制
- **可视化**: 集成网络图和趋势图的生成能力
- **个性化**: 基于用户研究方向的定制化推荐

### 🔗 系统集成
- **学术数据库**: 直接集成IEEE Xplore、SpringerLink等
- **文献管理**: 与Zotero、EndNote等工具对接
- **团队协作**: 支持多用户协作和权限管理
- **云端同步**: 实现跨设备的知识库同步

## 🔧 系统升级 v2.0 (2025-08-23)

### 🆕 **Agent架构优化**
- ✅ **合并优化**: `phm-paper-discovery` → `academic-researcher` (增强版)
- ✅ **质量先行**: 集成多维度质量过滤系统
- ✅ **向后兼容**: 旧Agent调用自动重定向，无需修改现有代码

### 🚫 **质量过滤系统**
```yaml
# configs/quality_filters.yaml
publishers:
  blacklist: [MDPI, Hindawi, Scientific Research Publishing]
  whitelist: [IEEE, Elsevier, Springer Nature, Wiley]

basic_filters:
  impact_factor_threshold: 5.0    # 用户要求: IF≥5
  min_quartile: "Q2"              # 期刊分区控制
  min_year: 2020                  # 时间窗口
```

**过滤效果**:
- 🚫 自动排除MDPI等低质量出版商
- 📊 影响因子≥5.0的高质量论文优先
- 🏆 Q1/Q2期刊分级筛选
- ⭐ PHM相关性智能评估

### 🧪 **测试验证**
```bash
# 运行质量过滤测试
python tests/run_tests.py --verbose --coverage

# 验证Agent功能
python -c "from src.utils.paper_quality_filter import PaperQualityFilter; print('✅ 质量过滤器加载成功')"
```

### 📈 **性能提升**
- ⚡ **搜索效率**: 统一Agent减少调用开销
- 🎯 **结果精准**: 自动质量筛选提升论文质量
- 🔄 **维护简化**: 单一Agent降低维护复杂度
- 📊 **统计详细**: 提供详细的过滤统计和质量分析

### 🔄 **迁移指南**
```python
# 旧调用方式 (仍然支持)
agent = "phm-paper-discovery"
# ↓ 自动重定向到
agent = "academic-researcher"  # 建议直接使用

# 新的高级功能
request = {
    "agent": "academic-researcher",
    "query": "轴承故障诊断深度学习",
    "filters": {
        "exclude_publishers": ["MDPI"],
        "min_impact_factor": 5.0,
        "max_quartile": "Q1"
    }
}
```

---

*📅 配置更新: 2025-08-23 | 🤖 Agent架构升级v2.0 | 🚫 MDPI过滤 | 📊 质量优先 | ✨ 向后兼容*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PHMbench) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
