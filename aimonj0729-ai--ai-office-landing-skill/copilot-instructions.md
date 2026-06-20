## ai-office-landing-skill

> Name: ai-office-landing

Name: ai-office-landing
Description: AI Office — Living Brief 协作循环(v2 对话式交互)
Entry Point: true

---

Skill Contents
- [Overview](#overview)
- [Usage](#usage)
- [Workflow](#workflow)
- [State Management](#state-management)
- [Phase 0: Design Reference Collection + Web Search](#phase-0-design-reference-collection--web-search)
- [Phase 1: Initial Interview + Iterative Refinement](#phase-1-initial-interview--iterative-refinement)
- [Phase 2: Style Tokens & Task Generation](#phase-2-style-tokens--task-generation)
- [Phase 3: Sequential Execution with Q&A](#phase-3-sequential-execution-with-qa)
- [Phase 4: Critic Review + Clarification Loop](#phase-4-critic-review--clarification-loop)
- [Phase 5: Final Delivery](#phase-5-final-delivery)
- [Resume & Recovery](#resume--recovery)
- [Cost Expectations](#cost-expectations)

---

## Overview

AI Office v2 是一个**对话驱动**的 Living Brief 协作循环：

**核心升级：**
- **对话式交互**: Executor 可在执行中提问，不必一次性想清楚
- **增量交付**: 每个阶段完成后展示预览，用户确认后再继续
- **问题缓冲**: Agent 的问题统一收集，批量问用户
- **双向通信**: 从"批处理"升级为"请求-响应"模式

**保留下来的优点:**
- Living Brief 作为单一事实来源
- Critic 独立审查，不继承主控上下文
- 风格令牌自动转译(hex/px)
- 断点续跑

New Constraints:
- `[QUESTION: ...]` 标记 Agent 需要用户澄清的地方
- `[USER_INPUT: ...]` 标记用户追加的信息
- `state.json` 持久化当前进度，跨 session 可恢复

---

## Usage

在任意项目目录下触发：

```bash
ai-office-landing [options]
```

**Options:**
- `--resume`: 从 state.json 恢复上次的对话式工作流
- `--auto-continue`: 自动继续，不暂停等待用户确认（适合批处理场景）
- `--human-critic`: 启用人工 Critic 模式
- `--cost-saving`: 启用成本节省模式，自动将 MEDIUM/LOW 角色路由到 Kimi/DeepSeek
- `--adapter <name>`: 强制所有 Executor 使用指定 adapter (claude-agent|kimi-api|deepseek-api)

**Adapter 路由 (v2.4 新增):**

Executor 任务通过 `adapters/route.sh` 自动路由到最优 adapter：

| 角色 | 质量等级 | 默认 Adapter | 成本节省模式 |
|------|---------|-------------|------------|
| Interviewer, Critic, Integrator | HIGH | claude-agent | claude-agent (不可降级) |
| Frontend | MEDIUM-HIGH | claude-agent | deepseek-coder |
| Copywriter, Designer | MEDIUM | claude-agent | kimi-api |
| SEO | LOW | claude-agent | kimi-api / deepseek-api |

环境变量配置：
```bash
export KIMI_API_KEY="your-key"        # 启用 Kimi adapter
export DEEPSEEK_API_KEY="your-key"    # 启用 DeepSeek adapter
export COST_SAVING_MODE=true          # 自动路由到便宜的 adapter
export AI_OFFICE_ADAPTER=kimi-api     # 强制指定 adapter (HIGH 角色除外)
```

**对话式示例：**
```bash
# 启动对话式工作流
ai-office-landing

# 过程中的交互
[Phase 1 完成] 
✓ 保存了 5 个维度的信息
✓ 但 Designer 想问：是否有品牌 logo 图片？

[Phase 3 - Copywriter 完成]
✓ 生成了 Hero/Features/FAQ/CTA 文案
✓ 但 Frontend 想问：是否接受 AI 生成的主视觉图？

[Phase 4 - Critic 审查]
⚠️ HIGH: Copy.md 使用了 "爆款"（在禁用词列表）
→ 需要你的修正: 删除或替换
```

---

## Workflow

```
Phase 0: Initialize
  ↓ 创建 ai-office/ 目录, state.json

Phase 1: Initial Interview
  ↓ AskUserQuestion 批量问 5 维度
  ↓ 生成 brief.md v1
  ↓ *可选* 收集阶段: 其他 Agent 可追加问题
      ↓ 如果有 [QUESTION], 问你 → 更新 brief.md → 回到 Checkpoint
      ↓ 如果没有 → 进入 Phase 2

Phase 2: Style Tokens & Tasks
  ↓ 生成 style-tokens.md, tasks.md
  ↓ *可选* 收集阶段
      ↓ 如果有 [QUESTION] → 问你 → 更新 → 回到 Checkpoint
      ↓ 如果没有 → 进入 Phase 3

Phase 3: Sequential Execution (逐个执行)
  ↓ Task 1: Copywriter → copy.md
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 1
      ↓ 无 → 继续 Task 2
  ↓ Task 2: Designer → design-spec.md
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 2
      ↓ 无 → 继续 Task 3
  ↓ Task 3: Frontend → index.html
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 3
      ↓ 无 → 继续 Task 4
  ↓ Task 4: SEO → meta.md
      ↓ 检查是否有 [QUESTION]
      ↓ 有 → 问你 → 修正 → 回到 Task 4
      ↓ 无 → 进入 Phase 4

Phase 4: Critic Review
  ↓ 独立 Critic 审查所有 outputs
  ↓ 生成 critique.md
  ↓ *可选* 如果有 HIGH/CRITICAL
      ↓ 收集所有问题 → 统一问你 → 修正 → 回到 Phase 3
      ↓ 如果都通过 → 进入 Phase 5

Phase 5: Final Delivery
  ↓ 整合所有交付物
  ↓ 清除 state.json
```

---

## State Management

### State File Location

`ai-office/state.json` - 存储当前会话状态，结构如下：

```json
{
  "version": "v2",
  "current_phase": 3,
  "current_task": 2,
  "checkpoint": {
    "phase": 3,
    "task": 2,
    "text": "Designer 生成 design-spec.md"
  },
  "pending_questions": [
    {
      "source": "copywriter",
      "file": "ai-office/questions/copywriter-questions.md",
      "questions": ["是否有品牌 slogan?", "Hero 图用真实照片还是插画?"]
    }
  ],
  "user_inputs": {
    "brand_slogan": "让世界更美好",
    "hero_image_type": "真实照片"
  },
  "outputs_status": {
    "copy.md": "completed_with_user_feedback",
    "design-spec.md": "in_progress",
    "index.html": "pending",
    "meta.md": "pending"
  }
}
```

### State Operations

```bash
# 先加载仓库内置 helper，避免手写 jq 表达式
source "$SKILL_ROOT/state-management.sh"

# 缺少 state.json 时才初始化，保留 --resume 现场
ensure_state_initialized

# 读取当前进度
CURRENT_PHASE="$(get_current_phase)"
CURRENT_TASK="$(get_current_task)"

# 写入简单字段时显式传 value_type
write_state "current_phase" "3" "number"
write_state "current_task" "1" "number"

# 对带点号/连字符的真实输出键，使用字面键 helper
mark_task_waiting_for_user "design-spec.md"
mark_task_completed "copy.md"

# 保存用户补充输入
save_user_input "hero_image_type" "真实照片"

# 添加待处理问题与 checkpoint，不要直接手拼 pending_questions JSON
add_pending_question "designer" "需要确认配色方案"
create_checkpoint 3 2 "Designer 等待用户确认配色方案"
```

### Recovery Logic

每个阶段开始时检查 state.json：
```bash
if [[ -f "ai-office/state.json" ]]; then
  source "$SKILL_ROOT/state-management.sh"
  CURRENT_PHASE=$(get_current_phase)
  log "从 Phase $CURRENT_PHASE 恢复..."
else
  CURRENT_PHASE=0
fi
```

---

---

## Phase 0: Design Reference Collection + Material Search

**目标:** 收集设计参考，搜索视觉素材，为后续 Agent 提供 concrete 参考

**触发条件:**
- 用户提供了设计思路、参考网站或视觉参考
- 用户希望 Agent 上网搜索素材而非凭空创作

### Step 0.1: Collect Design Intent

```bash
log "Phase 0: 设计参考收集"

# 问用户是否有设计参考
AskUserQuestion {
  question: "请描述你的设计思路（可多选）",
  options: [
    "我有参考网站或落地页",
    "我有品牌视觉指南/设计系统",
    "我只有文字描述的风格",
    "我希望 Agent 帮我找灵感",
    "跳过，直接进入需求访谈"
  ]
}
```

**如果用户选择 "我有参考网站":**
```bash
AskUserQuestion {
  question: "请提供参考网站 URL (多个用逗号分隔)",
  multiSelect: false
}
# 保存到 ai-office/design-intent.md
echo "$USER_INPUT" > ai-office/design-intent.md
echo "source_type: reference_urls" >> ai-office/design-intent.md
```

**如果用户选择 "我有品牌视觉指南":**
```bash
AskUserQuestion {
  question: "请上传或提供路径: 品牌手册/logo文件/设计系统文件",
  options: ["上传文件", "提供本地路径"]
}
# 复制文件到 ai-office/references/
mkdir -p ai-office/references
if [[ "$USER_CHOICE" == "上传文件" ]]; then
  # 提示用户上传文件
  log "请上传文件到 ai-office/references/"
else
  # 复制本地文件
  cp "$USER_PATH" ai-office/references/
fi
echo "source_type: brand_guidelines" > ai-office/design-intent.md
echo "files_location: ai-office/references/" >> ai-office/design-intent.md
```

**如果用户选择 "只有文字描述":**
```bash
AskUserQuestion {
  question: "请描述风格：现代/极简/温暖/科技感/活泼... 具体元素",
  multiSelect: false
}
# 保存描述到 ai-office/design-intent.md
echo "source_type: text_description" > ai-office/design-intent.md
echo "description: $USER_INPUT" >> ai-office/design-intent.md
```

**如果用户选择 "帮我找灵感":**
```bash
log "进入自动搜索模式..."
echo "source_type: auto_inspiration" > ai-office/design-intent.md
echo "note: Will generate search keywords after Phase 1 interview" >> ai-office/design-intent.md
```

**如果用户选择 "跳过":**
```bash
log "跳过 Phase 0，直接开始 Phase 1"
write_state "current_phase" "1"
write_state "outputs_status.design-references" "skipped"
# Continue to Phase 1
```

### Step 0.2: Design Researcher Agent Search

**This step executes only if design-intent.md exists and is not "skipped"**

```bash
# 创建输出目录
mkdir -p ai-office/references

# 检查是否有 design-intent.md
if [[ -f "ai-office/design-intent.md" ]]; then
  SOURCE_TYPE=$(grep "source_type:" ai-office/design-intent.md | cut -d' ' -f2)
  
  if [[ "$SOURCE_TYPE" == "skipped" ]]; then
    log "Phase 0 skipped by user"
  else
    log "Design Researcher 开始搜索素材..."
    
    # 调用 Design Researcher Agent
    # This agent will:
    # 1. Read design-intent.md
    # 2. Execute web searches for materials
    # 3. Generate design-references.md
    
    Agent({
      description: "Design Researcher - Search materials and references",
      prompt: $(cat "${SKILL_ROOT}/prompts/design-researcher.md"),
      context_files: ["ai-office/design-intent.md"],
      output_path: "ai-office/references/design-references.md",
      tools: ["WebSearch", "WebFetch"]
    })
    
    if [[ -f "ai-office/references/design-references.md" ]]; then
      log "✓ design-references.md 已生成"
      write_state "outputs_status.design-references" "completed"
      
      # 提取搜索到的素材摘要
      REF_COUNT=$(grep -c "### " ai-office/references/design-references.md || echo 0)
      ASSET_COUNT=$(grep -c "\[Asset\|http" ai-office/references/design-references.md || echo 0)
      
      log "发现 $REF_COUNT 个参考类别，$ASSET_COUNT 个直接素材"
    else
      log_warning "Design Researcher 未生成输出"
      write_state "outputs_status.design-references" "failed"
    fi
  fi
else
  log "无 design intent，跳过搜索"
  write_state "outputs_status.design-references" "skipped"
fi

# 创建 Checkpoint
write_state "current_phase" "0"
write_state "checkpoint" '{"phase": 0, "text": "设计参考收集完成"}'
log "Phase 0 完成"
```

### Step 0.3: [Optional] User Review Search Results

```bash
# 如果有搜索结果，问用户是否满意
if [[ -f "ai-office/references/design-references.md" ]]; then
  SIZE=$(wc -c < ai-office/references/design-references.md)
  
  if [[ $SIZE -lt 100 ]]; then
    log "搜索结果较少，可能需要补充..."
  fi
  
  log "搜索完成! 已生成 ai-office/references/design-references.md"
  log "请查看搜索结果，是否需要补充？"
  
  AskUserQuestion {
    question: "搜索到的素材是否合适？",
    options: [
      "很满意，继续下一步",
      "需要补充搜索",
      "跳过，直接进入访谈"
    ]
  }
  
  case "$USER_CHOICE" in
    "补充搜索")
      log "请提供补充搜索关键词..."
      AskUserQuestion {
        question: "补充搜索：请描述想要找什么样的素材"
      }
      # 追加到 design-intent.md，重新搜索
      echo "" >> ai-office/design-intent.md
      echo "additional_search: $USER_INPUT" >> ai-office/design-intent.md
      log "重新执行搜索..."
      # 重新调用 Design Researcher
      goto_phase 0
      ;;
    "跳过")
      log "用户选择跳过，进入 Phase 1"
      ;;
    *)
      log "用户确认素材，继续 Phase 1"
      ;;
  esac
fi

# 更新状态，准备进入 Phase 1
write_state "current_phase" "1"
```

---
## Phase 1: Initial Interview + Iterative Refinement
### Step 1.0: 成本预估与限额检查nn```bashn# 成本预估 - Phase 1nestimate_phase_cost 1n# 如果接近限额，提示用户选择nif should_warn_about_cost 1; thenn  log_warn "⚠️  注意：执行 Phase 1 后可能接近限额"n  prompt_for_cost_action 1nfin# 显示当前状态ndisplay_cost_headern```nn---nn

**目标:** 生成 `brief.md` v1，作为 Living Brief 的初始版本

### Step 1.1: Initial Batch Interview

```bash
# 读取访谈题
INTERVIEW_FILE="${SKILL_ROOT}/interview.md"

# 批量问第一批问题（5维度核心问题）
log "开始 Phase 1: 初始访谈（5维度）"
ask_user_batch "${INTERVIEW_FILE}" "## 第一批" "## 第二批"
```

**AskUserQuestion 格式:**
```bash
# 批量提问伪代码
for question in $(get_questions "${INTERVIEW_FILE}" "## 第一批"); do
  AskUserQuestion {
    question: "${question}",
    options: ["选项1", "选项2", "其他"]
  }
done
```

### Step 1.2: Generate Initial Brief

```bash
# 生成初始 brief.md
cp "${SKILL_ROOT}/templates/brief.md" "ai-office/brief.md"
fill_brief_dimensions "ai-office/brief.md" "${ANSWERS}"

# 记录状态
write_state "current_phase" "1"
write_state "current_task" "complete"
write_state "checkpoint" '{"phase": 1, "text": "初始访谈完成"}'

log "✓ brief.md v1 已生成"
```

### Step 1.3: [NEW] Question Collection Phase

这是 v2 的核心升级：允许其他 Agent 在看到 brief 后追加问题。

```bash
collect_questions() {
  local phase="$1"
  
  log "问题收集阶段: Phase $phase"
  log "其他 Agent 可查看 brief.md 并提出澄清问题..."
  
  # 创建一个虚拟 context 让 Agent 思考
  local context=$(cat << EOF
# Context for Question Generation

## Brief (v1)
$(cat ai-office/brief.md)

## Task
作为将要参与此项目的 Agent，请查看上述 brief。
当你有任何需要用户澄清的地方，生成一个 questions.md 文件。

**提问原则:**
- 只问会影响你产出的关键信息
- 不要问已在 brief 里的信息
- 用具体、可回答的方式提问
- 标记: [QUESTION: ...]

**示例:**
[QUESTION: 是否有品牌 logo 文件? 如有，请提供路径]
[QUESTION: Hero 区主视觉图倾向用照片还是插画?]
[QUESTION: 是否接受 AI 生成的配图?]
EOF
  )
  
  # Agent 分析并提问
  # (实际由主 Claude 执行，这里只是逻辑说明)
  
  # 检查是否生成了问题文件
  if [[ -f "ai-office/questions/phase${phase}.md" ]]; then
    log "发现 ${count} 个问题需要澄清..."
    
    # 读取所有问题
    local questions=$(grep "\[QUESTION:" "ai-office/questions/phase${phase}.md" | sed 's/\[QUESTION: //' | sed 's/\]//')
    
    # 问用户
    ask_user_clarification "$questions"
    
    # 保存用户输入到状态
    write_state "user_input" "$(get_user_answers)"
    
    # 更新 brief.md（追加用户输入）
    append_to_brief "${USER_ANSWERS}"
    
    # 回到 Checkpoint（重新生成 brief）
    log "更新 brief.md，回到 Checkpoint..."
    return 1  # 表示需要重新执行
  else
    log "✓ 无追加问题，进入下一阶段"
    return 0
  fi
}

# 调用问题收集
collect_questions "1"
if [[ $? -eq 1 ]]; then
  # 重新生成 brief
  goto checkpoint
fi
```

---

## Phase 2: Style Tokens & Task Generation
### Step 2.0: 成本预估与限额检查nn```bashn# 成本预估 - Phase 2nestimate_phase_cost 2n# 如果接近限额，提示用户选择nif should_warn_about_cost 2; thenn  log_warn "⚠️  注意：执行 Phase 2 后可能接近限额"n  prompt_for_cost_action 2nfin# 显示当前状态ndisplay_cost_headern```nn---nn

### Step 2.1: Generate Style Tokens

```bash
log "生成 style tokens..."

# 提取情绪词
EMOTION_WORDS=$(grep -A5 "情绪关键词" ai-office/brief.md | grep "^" | tr -d '- ')

# 转译为具体 token
generate_style_tokens() {
  for word in $EMOTION_WORDS; do
    # 调用转译函数（同 v1）
    translate_emotion_word "$word"
  done > ai-office/style-tokens.md
}

# 记录决策
write_state "current_phase" "2"
write_state "checkpoint" '{"phase": 2, "text": "style-tokens.md 生成"}'
```

### Step 2.2: Generate Tasks

```bash
log "生成 tasks.md..."

cp "${SKILL_ROOT}/templates/tasks.md" "ai-office/tasks.md"

# 根据 brief 定制验收标准
customize_task_criteria "ai-office/tasks.md" "ai-office/brief.md"

write_state "checkpoint" '{"phase": 2, "text": "tasks.md 生成"}'
```

### Step 2.3: [NEW] Question Collection Phase

```bash
collect_questions "2"

# 如果有问题，识别是哪个 task 需要的
if [[ $? -eq 1 ]]; then
  # 例如 Designer 可能问："brief 里的 '克制' 具体指多大字体?"
  # 用户回答后，更新 tasks.md 的验收标准
  update_task_criteria "designer" "${USER_ANSWER}"
  goto checkpoint
fi
```

---

## Phase 3: Sequential Execution with Q&A
### Step 3.0: 成本预估与限额检查nn```bashn# 成本预估 - Phase 3nestimate_phase_cost 3n# 如果接近限额，提示用户选择nif should_warn_about_cost 3; thenn  log_warn "⚠️  注意：执行 Phase 3 后可能接近限额"n  prompt_for_cost_action 3nfin# 显示当前状态ndisplay_cost_headern```nn---nn

**核心改变：** v1 是并行执行，v2 改为**逐个执行，每个完成后可交互**。

### Step 3.1: Execute Copywriter Task

```bash
execute_with_qa() {
  local task_id="$1"
  local role="$(read_task_field $task_id "task")"
  local output_path="$(read_task_field $task_id "output_path")"
  
  log "执行 Task $task_id: $role"
  
  # 如果 output 已存在，检查是否需要重新生成
  if [[ -f "$output_path" ]]; then
    local status=$(read_state "outputs_status" | jq -r ".$role")
    if [[ "$status" == "completed_confirmed" ]]; then
      log "$role 已确认完成，跳过"
      return 0
    fi
  fi
  
  # 通过 adapter 路由执行 task (v2.4: 自动选择最优 adapter)
  # route.sh 根据角色质量等级 + 环境变量自动选择 claude-agent / kimi-api / deepseek-api
  local adapter_script="${SKILL_ROOT}/adapters/route.sh"
  local prompt_file=$("$adapter_script" "$role" "$task_id" "$output_path")
  
  # 如果 adapter 返回的是 prompt 文件 (claude-agent 模式), 则调用 Agent 工具
  # 如果 adapter 直接写了 output (kimi-api / deepseek-api 模式), 则跳过 Agent 调用
  if [[ -f "$output_path" ]]; then
    log "✓ Adapter 已直接生成输出: $output_path"
  else
    execute_task "$task_id"
  fi
  
  # [NEW] 检查是否有问题
  check_task_questions "$role" "$output_path"
}

check_task_questions() {
  local role="$1"
  local output_path="$2"
  local output_key
  output_key="$(basename "$output_path")"
  
  # 检查 output 文件中是否有 [GAP] 或 [QUESTION] 标记
  if grep -q "\[GAP:" "$output_path" || grep -q "\[QUESTION:" "$output_path"; then
    log "$role 在产出中标记了问题..."
    
    # 提取所有问题
    local questions_file="ai-office/questions/${role}-questions.md"
    extract_questions_from_output "$output_path" > "$questions_file"
    
    # 用 helper 更新状态，避免把自由文本直接拼进 jq
    add_pending_question "$role" "" "$questions_file"
    mark_task_waiting_for_user "$output_key"
    create_checkpoint 3 "${CURRENT_TASK:-0}" "$role 等待用户回答"
    
    # 问用户
    ask_user_and_update "$role"
    
    # 更新 output 文件（移除 [GAP] 并填入用户答案）
    apply_user_answers "$output_path" "${USER_ANSWERS}"
    
    # 标记为已完成
    mark_task_completed "$output_key" "completed_with_user_feedback"
  else
    log "$role 无问题，继续下一步"
    mark_task_completed "$output_key"
  fi
}

# 执行 Copywriter
write_state "current_phase" "3"
write_state "current_task" "1"
execute_with_qa "1"
```

### Step 3.2: Execute Designer Task

```bash
write_state "current_task" "2"
execute_with_qa "2"
```

**Designer 可能提出的问题示例:**
```markdown
# design-spec.md 内部

## Component Specs

### Hero 区主视觉
[QUESTION: 品牌是否有官方主视觉图? 如有请提供路径，如 /images/hero.jpg]
[QUESTION: 如无，倾向用: (1) 找 stock photo (2) AI 生成 (3) 纯色背景]
```

当检测到这个标记时，流程会：
1. 暂停执行
2. 把问题显示给你
3. 等你回答
4. 更新 design-spec.md
5. 继续执行

### Step 3.3: Execute Frontend Task

```bash
write_state "current_task" "3"
execute_with_qa "3"
```

**Frontend 可能提出的问题:**
```markdown
# index.html 内部

<!-- Hero 区 -->
<div class="hero">
  <!-- [GAP: 需要 Hero 图路径] -->
  <!-- [QUESTION: 是否接受 AI 生成的主视觉? 尺寸要求?] -->
</div>
```

### Step 3.4: Execute SEO Task

```bash
write_state "current_task" "4"
execute_with_qa "4"
```

**SEO 可能提出的问题:**
```markdown
# meta.md 内部

## Schema.org

- [QUESTION: 是否已有官网域名? 如有请提供，用于 canonical URL]
- [QUESTION: 是否有品牌 logo 文件? 用于 Schema.org Organization]
```

---

## Phase 3.5: Orchestrator - Work Integration & Summary

**新增 Phase 3.5** - 在所有 Executor 完成后，Orchestrator Agent 自动整合所有工作并生成综合报告。

### Step 3.5.0: 执行 Orchestrator

```bash
# 启动 Orchestrator，自动收集和汇总所有输出
~/.claude/skills/ai-office-landing/orchestrator.sh
```

Orchestrator 会自动：
1. **检查所有 Executor 输出** - 验证 copy.md, design-spec.md, index.html, meta.md 是否存在
2. **生成进度仪表板** - 显示每个 Agent 的完成状态和关键指标
3. **检查跨 Agent 一致性** - 确认 Hero headline、FAQ、CTA、SEO keywords 是否对齐
4. **检查设计令牌合规性** - 统计所有令牌的使用情况
5. **识别信息缺口和冲突** - 找出需要 Critic 或用户关注的问题
6. **生成决策记录** - 汇总所有用户决策
7. **输出项目指标** - 内容、代码、性能指标

**Orchestrator 示例输出:**
```
╔══════════════════════════════════════════════════════════════════╗
║              AI Office Orchestrator - Phase 3.5                  ║
╠══════════════════════════════════════════════════════════════════╣
║  执行状态:                                                       ║
║    ✓ Copywriter: copy.md (45 lines, 2156 bytes)                 ║
║    ✓ Designer: design-spec.md (89 lines, 4123 bytes)            ║
║    ✓ Frontend: index.html (234 lines, 8456 bytes)               ║
║    ✓ SEO: meta.md (23 lines, 1123 bytes)                        ║
╠══════════════════════════════════════════════════════════════════╣
║  整体完成度: 100% (4/4 Agents)                                  ║
╚══════════════════════════════════════════════════════════════════╝

** 主要交付物 **
- ✓ copy.md: Hero + 5 features + 5 FAQs + CTA (2156 bytes)
- ✓ design-spec.md: Responsive layout, 12 components, token-compliant (4123 bytes)
- ✓ index.html: Single file with embedded CSS/JS (8456 bytes)
- ✓ meta.md: Complete SEO meta + OG + Schema.org (1123 bytes)
```

### Step 3.5.1: 查看 Orchestrator 汇总报告

生成的 `ai-office/outputs/orchestrator-summary.md` 包含：

```markdown
# Orchestrator Summary - Executive Report

** 项目 **: Specialty Coffee Subscription Landing Page
** 生成时间 **: 2026-04-11T10:15:23Z
** 总工作量 **: 4 Executors
** 完成度 **: 100%

## 1. 执行摘要
[...综合概述...]

## 2. 进度仪表板
[...每个 Agent 的状态...]

## 3. 跨 Agent 一致性检查
[...Content、Design、SEO 对齐情况...]

## 4. 差距与冲突报告
[...信息缺口和 Agent 冲突...]

## 5. 整合说明
[...文件依赖关系和加载顺序...]

## 6. 关键决策记录
[...用户在所有阶段的决策...]

## 7. 项目指标
[...内容、代码、性能数据...]

## 8. 下一步行动
[...准备进入 Phase 4 的建议...]
```

### Step 3.5.2: 处理 Orchestrator 发现的问题

根据 Orchestrator 报告采取相应行动：

** 场景 A: 完成度 < 75% **
- Orchestrator 会警告进入 Phase 4 可能信息不足
- 建议: 返回 Phase 3，重新运行缺失的 Agent

** 场景 B: 发现信息缺口**
- Orchestrator 列出 [QUESTION] 和 [GAP] 标记
- 建议: 在 Phase 4 中向用户提问，或补充到 brief.md

**场景 C: 发现 Agent 冲突**
- 例如: CTA 文本不一致、FAQ 数量不匹配
- 建议: Phase 4 标记为 MEDIUM/HIGH 优先级问题

**场景 D: 全部正常 ✅**
- 完成度 100%，无严重冲突
- Phase 4 可以进行标准审查

### Step 3.5.3: 动态 Skill 发现 (增强功能)

在 Phase 3 执行期间，Designer Agent 可以自动发现和加载相关 skills：

```bash
# 为 Designer 自动发现相关 skills
~/.claude/skills/ai-office-landing/discover-skills.sh auto-designer

# 输出示例:
# [SKILL-DISCOVERY] 为 Designer 发现 5 个候选 skills:
#   - color-palettes
#   - typography-guide
#   - layout-templates
#   - web-design-guidelines
#   - ui-ux-pro-max

# 加载特定 skill 供 Designer 使用
~/.claude/skills/ai-office-landing/discover-skills.sh load designer color-palettes
```

** 加载 skill 后: **
- Designer Agent 可以在 `~/.claude/skills/ai-office-landing/context/designer/` 访问 skill 内容
- 参考 skill 的洞察来丰富 design-spec.md
- Enhancement: 在 design-spec.md 中添加 "## Design References" 章节，说明哪些 skills 辅助了决策

---

## Phase 4: Critic Review + Clarification Loop
### Step 4.0: 成本预估与限额检查nn```bashn# 成本预估 - Phase 4nestimate_phase_cost 4n# 如果接近限额，提示用户选择nif should_warn_about_cost 4; thenn  log_warn "⚠️  注意：执行 Phase 4 后可能接近限额"n  prompt_for_cost_action 4nfin# 显示当前状态ndisplay_cost_headern```nn---nn

### Step 4.1: Independent Critic Invocation

```bash
log "Phase 4: Critic 独立审查..."

# 检查是否人工模式
if [[ "$HUMAN_CRITIC" == "true" ]]; then
  log "人工 Critic 模式已启用"
  log "请手动审查并创建 ai-office/critique.md"
  write_state "current_phase" "4"
  write_state "checkpoint" '{"phase": 4, "text": "等待人工审查"}'  
  exit 0
fi

# 独立 Critic Agent
# (实际由主 Claude 执行)
critic_prompt=$(assemble_critic_prompt)

# 执行审查
# Agent({
#   description: "Independent Critic",
#   prompt: "$critic_prompt",
#   output_path: "ai-office/critique.md"
# })

write_state "current_phase" "4"
write_state "checkpoint" '{"phase": 4, "text": "Critic 审查完成"}'
```

### Step 4.2: Analyse Critic Findings

```bash
# 读取 critique.md
CRITICAL_COUNT=$(grep -c "^### \[CRITICAL\]" ai-office/critique.md || echo 0)
HIGH_COUNT=$(grep -c "^### \[HIGH\]" ai-office/critique.md || echo 0)
MEDIUM_COUNT=$(grep -c "^### \[MEDIUM\]" ai-office/critique.md || echo 0)

log "Critic 发现: CRITICAL: $CRITICAL_COUNT, HIGH: $HIGH_COUNT, MEDIUM: $MEDIUM_COUNT"

# 如果有 CRITICAL 或 HIGH，需要用户介入
if [[ $CRITICAL_COUNT -gt 0 ]] || [[ $HIGH_COUNT -gt 0 ]]; then
  log "需要你的澄清..."
  
  # 提取所有需要用户决策的 findings
  extract_critic_questions > ai-office/questions/critic-questions.md
  
  # 问用户
  ask_user_clarification "critic"
  
  # 用户的回答将用于修复 outputs
  # 例如：用户确认 "爆款" 禁用词应该替换为 "热销"
  
  # 回到 Phase 3，重新生成受影响的 outputs
  identify_affected_tasks()
  
  write_state "current_phase" "3"
  # 只重新生成受影响的 task
  execute_with_qa "$affected_task_id"
  
  # 然后回到 Phase 4 (再次审查)
  goto_phase_4
else
  log "✓ 无重大问题，进入 Phase 5"
fi
```

**Critic 可能提出的问题:**
```markdown
### [HIGH] Copy.md 的 Hero 主标题

**问题:** 使用了禁用词 "爆款"
**依据:** brief.constraints.禁用词 列表包含 "爆款"
**建议:** 替换为 "热销产品" 或 "人气单品"

→ 你的决定是？
```

```markdown
### [HIGH] design-spec.md 的配色

**问题:** style-tokens.md 定义 `color-brand-primary: #2A2A2A`, 
但 design-spec.md 使用 `color-brand-primary: #333333`
**依据:** 风格令牌不匹配

→ 请确认使用哪个值？
```

**用户回答后：**
- 更新 critique.md，标记为 "用户已确认"
- 应用修复到相关 output
- 重新运行独立 Critic 确认修复

---

## Phase 5: Final Delivery

```bash
log "Phase 5: 最终交付"

cat << EOF
╔══════════════════════════════════════════════════════════════╗
║              AI Office — 交付物已生成                        ║
╚══════════════════════════════════════════════════════════════╝

📁 项目目录: ${PWD}/ai-office/

📦 交付文件:
EOF

# 列出所有交付物
for file in ai-office/outputs/*; do
  if [[ -f "$file" ]]; then
    SIZE=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
    echo "   ✓ $(basename $file) (${SIZE} bytes)"
  fi
done

echo ""
echo "📝 决策日志: ai-office/decision-log.md"
echo "   记录了所有阶段的关键决策、用户输入和修改"

echo ""
echo "✅ 对话式交互记录:"
if [[ -f "ai-office/user-qa-log.md" ]]; then
  echo "   共收集 $(grep -c '### Q:' ai-office/user-qa-log.md) 次用户回答"
fi

# 清除 state.json（完成）
rm -f ai-office/state.json
log "AI Office 对话式工作流完成!"
```

---

## Resume & Recovery

### State-Based Recovery

```bash
# 启动时检查
if [[ "$1" == "--resume" ]]; then
  if [[ -f "ai-office/state.json" ]]; then
    CURRENT_PHASE=$(read_state "current_phase")
    CURRENT_TASK=$(read_state "current_task")
    
    log "恢复对话式工作流..."
    log "当前阶段: Phase $CURRENT_PHASE, Task: $CURRENT_TASK"
    
    # 如果有 pending questions，先处理
    if [[ $(read_state "pending_questions" | jq length) -gt 0 ]]; then
      log "有 $(read_state "pending_questions" | jq length) 个待回答问题"
      handle_pending_questions
    fi
    
    goto_phase "$CURRENT_PHASE"
  else
    log "没有找到 state.json，从头开始"
    CURRENT_PHASE=0
  fi
fi
```

### Checkpoint System

每个关键节点都可回退到 Checkpoint：

```bash
goto_checkpoint() {
  local checkpoint=$(read_state "checkpoint")
  local phase=$(echo "$checkpoint" | jq -r '.phase')
  
  log "回到 Checkpoint: Phase $phase"
  
  case $phase in
    1) execute_phase_1 ;;  # 重新生成 brief
    2) execute_phase_2 ;;  # 重新生成 tokens + tasks
    3) execute_task_1 ;;   # 从 Copywriter 开始
    4) execute_phase_4 ;;  # 重新审查
  esac
}
```

---

## Cost Expectations

**对话式模式 vs 批处理模式:**

| 模式 | 交互次数 | Token 消耗 | 适用场景 |
|---|---|---|---|
| **批处理 (v1)** | 1次批量提问 + 等待 | ~100k | 需求清晰，用户熟悉流程 |
| **对话式 (v2)** | 多次小批次提问 | ~130k | 需求模糊，需要逐步澄清 |

**对话式模式的额外消耗:**
- 每个 Agent 可追加问题：平均 1-2 个问题
- 每个问题需要额外的 Agent 调用：~3k tokens/question
- 状态管理 & 增量交付：~5k tokens overhead

**总成本估算:**
```
基础流程: 100k
问题收集 (5个): +15k
用户回答处理: +10k
状态管理: +5k
总计: ~130k tokens
```

---

## Implementation Notes

**v1.0 (批处理模式):**
- ✅ 5维度结构化访谈
- ✅ Style Tokens 自动转译
- ✅ 4专家并行执行
- ✅ 独立 Critic 审查

**v2.0 (对话式交互):**
- ✅ 状态管理 (state.json)
- ✅ 增量问题收集
- ✅ 双向沟通循环
- ✅ 用户问答日志
- ✅ Checkpoint 恢复

**v2.1 (计划):**
- 实时预览 (每个阶段后生成预览链接)
- 视觉反馈 (Designer 可生成 mood board)
- 多轮 Critic 审查

---

## User Experience Flow

### 首次使用

```
用户: /landing
AI: 开始 AI Office 对话式工作流

Phase 1: 初始访谈
→ 问你 5 个维度的问题
→ 生成 brief.md v1
→ Designer 问: 有 logo 吗?

用户: 有，在 /images/logo.png

AI: 已记录，继续 Phase 2...
(生成 style-tokens 和 tasks)

Phase 3: Copywriter 执行
→ 生成 copy.md
→ 无问题，继续

Phase 3: Designer 执行
→ 生成 design-spec.md
→ Frontend 问: Hero 图用照片还是插画?

用户: 用照片，真实产品图

AI: 已更新，继续...

Phase 3: Frontend 执行
→ 生成 index.html
→ 无问题，继续

Phase 3: SEO 执行
→ 生成 meta.md
→ Critic 审查...

Phase 4: Critic 发现
→ [HIGH] copy.md 用了禁用词 "爆款"
→ 你的决定?

用户: 改为 "人气单品"

AI: 已修正，重新生成 copy.md
→ Critic 再次审查... 通过!

Phase 5: 交付
→ 所有文件已生成
✅ 完成！
```

### 恢复会话

```
用户: /landing --resume
AI: 恢复对话式工作流
当前在 Phase 3 - Designer (等待回答)

待回答问题:
1. 是否有品牌主色值? (目前用的是 style-tokens 推导的 #2A2A2A)

用户: 主色是 #3A2618

AI: 已更新 style-tokens.md
继续 Designer 任务...
✓ Designer 完成
→ Frontend 开始...
```

---

## User Q&A Log

所有用户回答都被记录到 `ai-office/user-qa-log.md`：

```markdown
# 用户问答日志

## Phase 1 - 初始访谈

### Q1 (goal): 主要转化动作是什么?
**A:** 让用户留下邮箱

### Q2 (style): 情绪关键词?
**A:** 克制、手作、产地

## Phase 1 - 追加问题

### Q6 (Phase 1.3): Designer 问：是否有品牌 logo?
**A:** 有，在 /images/logo.png

## Phase 3 - Copywriter

### Q7 (Task 1): Copywriter 问: Slogan 长度限制?
**A:** 主标题 ≤ 8 字

(...)
```

这份日志：
- 让你看到 AI 如何理解你的需求
- 可以作为项目文档的一部分
- 下次迭代时可以参考

---

**AI Office v2.0 — 让 Agent 协作像聊天一样自然。**

---
> Source: [aimonj0729-ai/ai-office-landing-skill](https://github.com/aimonj0729-ai/ai-office-landing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
