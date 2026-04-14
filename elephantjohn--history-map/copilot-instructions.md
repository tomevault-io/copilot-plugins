## history-map

> "say": "项目：华夏脉络 (HuaxiaNexus) - 中国历史学习平台。当前阶段：{{CURRENT_PHASE}}。角色：{{CURRENT_ROLE}}。遵循单人全栈开发模拟团队协作原则。"

{
  "rules": [
    // --- 项目标识与状态 ---
    {
      "when": "always",
      "say": "项目：华夏脉络 (HuaxiaNexus) - 中国历史学习平台。当前阶段：{{CURRENT_PHASE}}。角色：{{CURRENT_ROLE}}。遵循单人全栈开发模拟团队协作原则。"
    },
    {
      "when": "session_start",
      "say": "📋 请确认当前开发状态：\n- 阶段：{{CURRENT_PHASE}}\n- 角色：{{CURRENT_ROLE}}\n- Sprint：{{CURRENT_SPRINT}}\n- 任务：{{CURRENT_TASK}}\n\n今日开发目标是什么？需要切换角色吗？",
      "ask_for_confirmation": true
    },
    
    // --- 角色切换提醒 ---
    {
      "when": "editing_file_type:py AND editing_directory:api",
      "say": "🔧 后端工程师角色：专注于清晰的API设计（GraphQL）、高效的数据访问（PostgreSQL & Neo4j）、健壮的错误处理。数据模型是否与schema一致？是否编写了测试？"
    },
    {
      "when": "editing_file_type:dart AND editing_directory:lib",
      "say": "📱 Flutter工程师角色：遵循Clean Architecture，分离UI（Widgets）、状态（Riverpod）和逻辑（Services）。追求高性能和'朝代主题'美学。组件是否可复用？"
    },
    {
      "when": "editing_file_contains:test_ OR editing_directory:test",
      "say": "🧪 测试工程师角色：编写彻底的单元/集成测试，覆盖边缘情况。历史数据的特殊性（日期不确定性、多重命名）是否已测试？"
    },
    {
      "when": "designing_ui OR reviewing_ui_changes",
      "say": "🎨 UI/UX设计师角色：确保符合目标用户群体（25-40岁，高学历）期望。界面是否直观美观？是否体现中国历史文化元素？与竞品'全知识'相比如何？"
    },
    {
      "when": "editing_file_contains:Historical OR editing_file_contains:Dynasty",
      "say": "📚 数据工程师角色：专注于中国历史数据准确性。朝代划分、人名系统（姓名字号）、历史日期处理是否符合史学标准？数据来源可靠性如何？"
    },
    {
      "when": "planning_features OR creating_backlog",
      "say": "🎯 产品经理角色：此功能如何与免费增值模式匹配？考虑66.8%用户付费意愿，这是核心免费功能还是高级付费功能？用户价值如何？"
    },

    // --- Git工作流与分支管理 ---
    {
      "when": "creating_new_branch",
      "say": "🌿 分支策略：基于Git Flow创建分支。命名规范：\n- feature/{{PHASE}}-{{TASK_ID}}-简短描述\n- fix/{{TASK_ID}}-简短描述\n- docs/{{TASK_ID}}-简短描述\n确保从develop分支创建（hotfix除外）。",
      "ask_for_confirmation": true
    },
    {
      "when": "generating_commit_message",
      "say": "📝 提交规范检查：\n- 格式：<type>(<scope>): <subject>\n- 类型：feat, fix, docs, style, refactor, test, chore\n- 范围：api, ui, data, core, test\n- 任务ID：{{TASK_ID}}\n示例：feat(api): 实现人物关系查询接口 (#{{TASK_ID}})",
      "ask_for_confirmation": true
    },
    {
      "when": "before_merging_to_develop",
      "say": "✅ 合并前检查清单：\n1. 代码是否通过所有测试？\n2. 是否满足任务验收标准？\n3. 是否更新了相关文档？\n4. 是否进行了代码自审？\n5. 历史数据处理是否准确？",
      "require_confirmation": true
    },

    // --- 阶段里程碑跟踪 ---
    {
      "when": "phase_milestone_check",
      "say": "🏁 里程碑检查 - {{CURRENT_PHASE}}：\n- 核心交付物是否完成？\n- 质量标准是否达标？\n- 下阶段依赖是否就绪？\n- 风险是否可控？\n记录进度并更新项目状态文档。"
    },
    {
      "when": "starting_new_sprint",
      "say": "🚀 Sprint开始：\n1. 回顾上Sprint完成情况\n2. 确认本Sprint目标和任务\n3. 识别风险和依赖\n4. 安排每日任务优先级\n当前Sprint：{{CURRENT_SPRINT}}，主要目标：{{SPRINT_GOAL}}"
    },

    // --- 中国历史数据专业化 ---
    {
      "when": "editing_file_contains:Dynasty OR editing_file_contains:朝代",
      "say": "🏛️ 历史数据规范：确保朝代划分准确（夏商周-秦汉-魏晋南北朝-隋唐-宋元明清-民国）。时间范围和朝代特征是否正确？"
    },
    {
      "when": "editing_file_contains:Character OR editing_file_contains:人物",
      "say": "👤 人物数据规范：姓名系统包含姓、名、字、号、谥号、庙号。社会地位、籍贯、生卒年（支持不确定性）是否完整？"
    },
    {
      "when": "editing_file_contains:Relationship OR editing_file_contains:关系",
      "say": "🔗 关系数据规范：中国传统社会关系复杂（血缘、政治、师承、友谊等）。关系类型定义是否准确？时间有效性如何处理？"
    },
    {
      "when": "editing_file_contains:Neo4j OR editing_file_contains:Cypher",
      "say": "🕸️ 图数据库优化：Neo4j查询性能考虑。复杂关系查询是否建立了合适的索引？查询深度是否有限制？目标超越'全知识'基准。"
    },

    // --- 技术栈特定规则 ---
    {
      "when": "editing_file_type:dart AND editing_file_contains:Widget",
      "say": "🎨 Flutter UI规范：遵循Material Design 3，结合中国文化元素。组件是否响应式？是否支持深色模式？性能优化考虑？"
    },
    {
      "when": "editing_file_type:py AND editing_file_contains:FastAPI",
      "say": "⚡ FastAPI规范：异步处理、依赖注入、错误处理。API文档是否自动生成？是否实现了请求验证和响应模型？"
    },
    {
      "when": "editing_file_contains:GraphQL",
      "say": "🔍 GraphQL优化：查询复杂度限制、N+1问题解决、缓存策略。Schema设计是否支持历史数据的复杂查询需求？"
    },
    {
      "when": "editing_file_contains:Riverpod",
      "say": "📊 状态管理：Riverpod最佳实践，避免状态污染。Provider作用域是否合理？是否支持离线状态同步？"
    },

    // --- 质量保证 ---
    {
      "when": "creating_new_file",
      "say": "📄 文档同步检查：新文件是否需要更新API文档、架构文档或用户文档？代码注释是否充分？"
    },
    {
      "when": "performance_optimization",
      "say": "⚡ 性能优化重点：\n1. 图数据库查询优化\n2. Flutter列表虚拟化\n3. 图片懒加载\n4. API响应缓存\n5. 离线数据同步\n目标：3秒内加载，60fps流畅度"
    },
    {
      "when": "debugging",
      "say": "🐛 调试检查清单：\n1. 历史数据边界情况\n2. 网络异常处理\n3. 设备兼容性\n4. 内存泄漏\n5. 日志记录完整性\n错误是否影响核心功能？"
    },

    // --- 商业化考虑 ---
    {
      "when": "feature_planning",
      "say": "💰 商业模式对齐：考虑免费/付费功能划分。核心学习功能免费，高级分析和个性化功能付费。用户留存和付费转化如何？"
    },
    {
      "when": "user_experience_review",
      "say": "👥 用户体验评估：目标用户（历史爱好者、学生、研究者）的使用场景。界面是否降低了历史学习门槛？知识获取是否高效？"
    }
  ],
  
  "variables": {
    "CURRENT_PHASE": "Phase 1: 后端MVP开发期",
    "CURRENT_ROLE": "后端工程师",
    "CURRENT_SPRINT": "Sprint 2 (Week 3-4)",
    "CURRENT_TASK": "FastAPI后端架构设计",
    "SPRINT_GOAL": "完成GraphQL API设计和数据库集成",
    "TASK_ID": "TASK-002"
  },
  
  "project_context": {
    "name": "华夏脉络 (HuaxiaNexus)",
    "type": "中国历史学习平台",
    "tech_stack": "Flutter + FastAPI + PostgreSQL + Neo4j",
    "development_mode": "单人全栈开发",
    "methodology": "敏捷开发 + 角色轮换"
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/elephantjohn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
