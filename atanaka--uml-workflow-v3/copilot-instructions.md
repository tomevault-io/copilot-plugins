## uml-workflow-v3

> 10-step UML workflow: scenario→activity→usecase→class→state→sequence→validation→security→code→test→traceability. Caching, resume, single-step modes.


# UML Workflow v3

完全統合されたtoken効率化UMLワークフロー。キャッシュ管理・段階的実行・UML(.uml)出力最適化をすべて自動化。

## 🎯 Overview / 概要

このスキルは3つの最適化機能を統合：

1. **キャッシュシステム（項目1）**: 中間成果物の自動キャッシュと再利用
2. **段階的実行（項目2）**: 必要なステップのみ実行
3. **UML(.uml)最適化（項目3）**: クラスモデルは常に2形式の `.uml`（OMG + Eclipse UML2）を生成。クラスモデル以外の `.uml` はデフォルトOFF（40%高速化）

**最大75%のtoken削減を実現**

---

## 🤖 EXECUTION INSTRUCTIONS FOR CLAUDE / Claude実行指示

**CRITICAL**: When a user requests to use this skill, Claude MUST follow this exact workflow.

### Trigger Patterns / トリガーパターン

Execute this workflow when user says:
- "uml-workflow-v3で〜を生成"
- "uml-workflow-v3を使って〜"
- "token効率化ワークフローで〜"
- Any mention of this skill by name

### Quick Execution Steps / クイック実行手順

```
0. Detect environment (PHASE 0.0) → resolve {SKILL_DIR}/{OUTPUT_DIR} and tool names
1. Determine project_name from user's request
2. Ask questions using the question tool (Web: ask_user_input_v0 / Local: AskUserQuestion)
3. Execute run_workflow.py via shell (Web: bash_tool / Local: Bash) with user's answers
4. Read execution plan JSON
5. Call each sub-skill in sequence
6. Present results (Web: present_files / Local: SendUserFile)
```

**Detail**: See CLAUDE_QUICK_GUIDE.md for concise reference, or follow detailed phases below.

### Execution Workflow (MUST FOLLOW) / 実行ワークフロー

Claude will execute this skill by following these phases in order:

0. **Environment Detection** - Detect Web vs Local, resolve {SKILL_DIR}/{OUTPUT_DIR} and tool names (PHASE 0.0)
1. **Initialization** - Set up Python environment
2. **User Dialogue** - Ask configuration questions using the question tool (Web: ask_user_input_v0 / Local: AskUserQuestion)
3. **Configuration** - Build execution config using Python scripts
4. **Execution Plan** - Display plan and get confirmation
5. **Step-by-Step Execution** - Execute each step with cache management
6. **Completion** - Present results and statistics

Each phase is detailed below with exact commands to execute.

### ⚠️ CRITICAL: 2-Phase Auto-Split Architecture / 2フェーズ自動分割

**Problem**: Running all 10 steps in a single conversation exhausts the context window (~200K tokens). Step 8 (code generation) requires reading the largest sub-skill PIPELINE.md (~14K tokens) plus generating substantial code output, which causes failures around Step 8.

**Solution**: The workflow automatically splits into 2 phases:

| Phase | Steps | Purpose | Context Usage |
|-------|-------|---------|---------------|
| **Phase A** | 1-7 | Modeling & Validation | ~80K tokens |
| **Phase B** | 8-10 | Code Gen, Test Gen, Traceability | ~60K tokens |

**AUTO-SPLIT RULES**:

1. **Full Workflow mode**: After completing Step 7, Claude MUST:
   - Present all Phase A outputs via `present_files`
   - Cache all outputs
   - Inform user: "Phase A (Modeling) complete. Please start a **new conversation** and say: `uml-workflow-v3で{project_name}のStep 8から再開`"
   - **STOP execution.** Do NOT proceed to Step 8 in the same conversation.

2. **Resume from Step 8+ mode**: Claude loads cached artifacts and executes Steps 8-10 in a fresh context.

3. **Models-only mode**: Steps 1-7 only — no split needed.

4. **Exception**: If user explicitly says "continue in this conversation" or "このまま続行", Claude may attempt Steps 8+ but should warn about potential context limits.

---

## 🌐 PHASE 0.0: Environment Detection & Mapping (MUST RUN FIRST) / 環境自動判定

**CRITICAL — このスキルは Web版 (Claude.ai Skills sandbox) と ローカル版 (Claude Code) の両方で動作する。**
他のどのフェーズよりも先に、必ず環境を判定してプレースホルダとツールを確定すること。

このドキュメント本文では、環境に依存するパスを次のプレースホルダで表記している。
PHASE 0.0 で実際の値に解決してから、以降のすべてのコマンドで置換して使うこと。

| プレースホルダ | 意味 | Web版 の値 | ローカル版 の値 |
|---|---|---|---|
| `{SKILL_DIR}` | スキル本体ディレクトリ | `/mnt/skills/user/uml-workflow-v3` | `~/.claude/skills/uml-workflow-v3` |
| `{OUTPUT_DIR}` | 成果物の出力先 | `/mnt/user-data/outputs` | 作業中のプロジェクトディレクトリ（CWD） |
| `{WORK_DIR}` | コード生成先のルート | `/home/claude` | 作業中のプロジェクトディレクトリ（CWD） |

### Step 0.0.1: 環境判定コマンドを実行

```bash
python3 - <<'PY'
import sys, os
# scripts ディレクトリを探す（Web → ローカルの順）
for cand in ("/mnt/skills/user/uml-workflow-v3/scripts",
             os.path.expanduser("~/.claude/skills/uml-workflow-v3/scripts")):
    if os.path.isdir(cand):
        sys.path.insert(0, cand); break
from env_paths import is_web_environment, get_skill_dir, get_output_dir
print("ENV        :", "WEB" if is_web_environment() else "LOCAL")
print("SKILL_DIR  :", get_skill_dir())
print("OUTPUT_DIR :", get_output_dir())
PY
```

- 出力の `SKILL_DIR` を `{SKILL_DIR}`、`OUTPUT_DIR` を `{OUTPUT_DIR}` および `{WORK_DIR}` として以降で使用する。
- ⚠️ **ローカル版では `{OUTPUT_DIR}` = カレントワーキングディレクトリ**。成果物を特定フォルダに出したい場合は、先に対象プロジェクトへ `cd` するか、環境変数 `UML_WORKFLOW_OUTPUT_DIR` を設定してから実行すること。

### Step 0.0.2: ツール対応表（環境で読み替える）

本文中の例は Web版のツール名で書かれている。**ローカル版 (Claude Code) では必ず右列のツールに読み替えること。**
特に質問ツールはこの読み替えを怠ると質問が出ずにスキップされる（XMI質問が出なかった既知不具合の原因）。

| 用途 | Web版 (本文の表記) | ローカル版 (Claude Code) |
|---|---|---|
| ユーザーへの質問 | `ask_user_input_v0` | **`AskUserQuestion`** |
| 成果物の提示 | `present_files` | **`SendUserFile`** |
| ファイル閲覧 | `view` | **`Read`** |
| シェル実行 | `bash_tool` | **`Bash`** |

> 💡 ローカル版で `AskUserQuestion` を使う際は、本文の選択肢（キャッシュ/実行モード/**XMI生成**/テスト生成/技術スタック）をそのまま options として提示すること。

---

## 🛡 PHASE 0.5: Resume Mode Prerequisite Check (CRITICAL)

**When this phase applies**: User invokes "Resume from Step 8+ mode" — e.g.,
- 「uml-workflow-v3で{project}のStep 8から再開」
- "uml-workflow-v3 resume from step 8 for {project}"
- Any request where `mode_param == "resume"` and `start_step >= 8`

**Why this exists**: Phase A produces ~21 artifacts (JSON, PlantUML, Markdown specs, etc.) across 7 steps. In real-world Resume sessions, users often upload only a subset (e.g., 5 key JSON files) believing them sufficient. Phase B can still run on partial inputs but the resulting SSoT is permanently inconsistent. This check makes the inconsistency visible BEFORE Step 8 starts.

### Required Action

**BEFORE executing Step 8, Claude MUST run**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from verify_step_outputs import verify_phase_a_completeness, format_phase_a_report

result = verify_phase_a_completeness('{project_name}')
print(format_phase_a_report(result))
sys.exit(0 if result['is_complete'] else 1)
"
```

### Decision Tree

- **Exit code 0** (`is_complete == True`): Phase A artifacts are complete → proceed to Step 8.
- **Exit code 1** (`is_complete == False`): Claude MUST:
  1. Display the verification report (already printed by `format_phase_a_report`) to the user.
  2. **STOP execution**. Do NOT run Step 8.
  3. Ask the user explicitly which of the following they want:
     - (a) Upload the missing artifacts to `{OUTPUT_DIR}/` and re-run the check, or
     - (b) Re-run the affected Phase A step(s) in a fresh session, or
     - (c) Explicitly waive the check and proceed despite incompleteness (last resort — Claude must warn that the resulting SSoT will be permanently inconsistent).

### What Counts as "Phase A Complete"

The following must all hold:

- All Expected Output files for Steps 1–7 exist on disk (see `STEP_EXPECTED_OUTPUTS` in `verify_step_outputs.py`)
- `usecase-specifications/` directory exists and is non-empty
- Cross-check passes: JSON UC count in `{project}_usecase-output.json` equals the number of `.md` files in `usecase-specifications/`

### This check is non-negotiable

Do NOT skip this verification even if artifacts "look sufficient" or the user is in a hurry. The check itself takes <1 second; the consequences of skipping it (silent SSoT corruption, lost auditability) are far worse than any inconvenience.

---

## 📋 PHASE 0: Initialization / 初期化

**Objective**: Determine project name and set up Python environment

### Step 0.1: Determine Project Name

Claude analyzes the user's request to infer a project name:
- Extract from user's message (e.g., "受注システム" → "order-system")
- Convert to kebab-case (lowercase with hyphens)
- If unclear, ask user directly

**Example**:
```
User: "受注管理システムを生成"
→ project_name = "order-management"

User: "generate an inventory system"
→ project_name = "inventory-system"
```

### Step 0.2: Set Up Python Environment

Claude executes the following bash command to verify Python scripts are accessible:

```bash
ls -la {SKILL_DIR}/scripts/
```

If the directory doesn't exist, try fallback:
```bash
ls -la {SKILL_DIR}/scripts/
```

**Expected output**: 4 Python files should be listed
- workflow_cache_helper.py
- execution_mode_manager.py
- unified_workflow_executor.py
- interactive_workflow_executor.py

If scripts are not found, inform user and suggest re-uploading the skill.

---

## 📋 PHASE 1: User Dialogue / ユーザー対話

**Objective**: Collect execution configuration from user

Claude uses the question tool to ask the following questions
(**Web: `ask_user_input_v0` / Local Claude Code: `AskUserQuestion`** — see PHASE 0.0 mapping; the **XMI generation** question below MUST be asked in both environments):

### Question Set / 質問セット

```python
ask_user_input_v0({
    "questions": [
        {
            "question": "キャッシュを使用しますか？",
            "type": "single_select",
            "options": [
                "はい（推奨）- 前回の成果物を再利用してtoken節約",
                "いいえ - すべて新規生成",
                "クリア - キャッシュを削除して新規生成"
            ]
        },
        {
            "question": "実行モードを選択してください",
            "type": "single_select",
            "options": [
                "フルワークフロー（全10ステップ実行）",
                "指定ステップから再開",
                "モデルのみ生成（コード生成なし）",
                "バリデーションのみ実行"
            ]
        },
        {
            "question": "クラスモデル以外のモデル（アクティビティ/ユースケース/状態機械）も UML(.uml) で出力しますか？",
            "type": "single_select",
            "options": [
                "いいえ（推奨・デフォルト）- クラスモデルのみ UML 出力（高速）",
                "はい - 全モデルを .uml でも出力（UMLツール連携が必要な場合）"
            ]
        }
    ]
})
```

> 📐 **UML(.uml) 出力ポリシー（重要）**
> - **クラスモデルは指示がなくても常に2形式の `.uml` を生成する（デフォルトON）**:
>   - OMG 形式: `{project_name}_class-model.omg.uml`
>   - Eclipse UML2 (EMF/Papyrus) 形式: `{project_name}_class-model.emf.uml`
> - **クラスモデル以外（activity / usecase / statemachine）の `.uml` は、上の質問で「はい」を選んだ場合のみ生成する。** 既定では生成しない。
> - 出力する UML モデルファイルの拡張子は **すべて `.uml`**（`.xmi` は使用しない）。

### Follow-up Questions / 追加質問

Based on execution mode selection:

**If "指定ステップから再開" selected**:
```python
ask_user_input_v0({
    "questions": [{
        "question": "どのステップから再開しますか？",
        "type": "single_select",
        "options": [
            "Step 2: アクティビティ図 → ユースケース",
            "Step 3: ユースケース → クラス図",
            "Step 4: クラス図 → ステートマシン図",
            "Step 5: ユースケース → シーケンス図",
            "Step 6: モデルバリデーション",
            "Step 7: セキュリティ設計",
            "Step 8: コード生成",
            "Step 9: テスト生成",
            "Step 10: トレーサビリティマトリクス",
            "Step 10: トレーサビリティマトリクス"
        ]
    }]
})
```

**If mode includes code generation**:
```python
ask_user_input_v0({
    "questions": [
        {
            "question": "テストコードを生成しますか？",
            "type": "single_select",
            "options": [
                "はい（推奨）",
                "いいえ"
            ]
        },
        {
            "question": "バックエンドフレームワークを選択してください",
            "type": "single_select",
            "options": [
                "TypeScript + Express（推奨・軽量）",
                "TypeScript + NestJS（大規模向け）",
                "Python + FastAPI",
                "Java + Spring Boot"
            ]
        },
        {
            "question": "フロントエンドフレームワークを選択してください",
            "type": "single_select",
            "options": [
                "React + TypeScript + Vite + Tailwind CSS（推奨）",
                "Vue 3 + TypeScript + Vite",
                "フロントエンドは生成しない"
            ]
        },
        {
            "question": "アーキテクチャを選択してください",
            "type": "single_select",
            "options": [
                "モノリス（推奨・シンプル）",
                "マイクロサービス",
                "サーバーレス"
            ]
        }
    ]
})
```

> ⚠️ **収集したテックスタック選択値は会話コンテキストに記録し、Step 8 で usecase-to-code-v1 を呼び出す際に Case A として渡すこと。Step 8 で再度ユーザーに質問してはならない。**
```

---

## 📋 PHASE 2: Configuration Build / 設定構築

**Objective**: Create execution configuration using Python scripts

### Step 2.1: Map User Responses to Parameters

Based on user's answers, Claude determines:

```python
# Cache setting
cache_param = "yes"  # if user selected "はい（推奨）"
cache_param = "no"   # if user selected "いいえ"
cache_param = "clear" # if user selected "クリア"

# Execution mode
mode_param = "full"          # if "フルワークフロー"
mode_param = "resume"        # if "指定ステップから再開"
mode_param = "models_only"   # if "モデルのみ生成"
mode_param = "validate_only" # if "バリデーションのみ"

# Start step (if resume mode)
start_step = 2  # if user selected "Step 2: アクティビティ図 → ユースケース"
start_step = 3  # if user selected "Step 3: ユースケース → クラス図"
# ... up to step 9

# UML(.uml) generation for NON-class models (class model is ALWAYS dual-.uml by default)
xmi_flag = ""        # if user selected "いいえ（推奨・デフォルト）" → class model only
xmi_flag = "--xmi"   # if user selected "はい" → also emit activity/usecase/statemachine .uml

# Test generation
test_flag = ""          # if user selected "はい（推奨）"
test_flag = "--no-tests" # if user selected "いいえ"
```

### Step 2.2: Execute Configuration Script

Claude executes the run_workflow.py script using bash_tool:

```bash
python3 {SKILL_DIR}/scripts/run_workflow.py \
  {project_name} \
  --cache {cache_param} \
  --mode {mode_param} \
  [--start-step {start_step}] \
  [{xmi_flag}] \
  [{test_flag}]
```

**Example**:
```bash
python3 {SKILL_DIR}/scripts/run_workflow.py \
  order-system \
  --cache yes \
  --mode full
```

**Important**: If the scripts directory is not found at `{SKILL_DIR}/scripts/`, try the fallback location:
```bash
python3 {SKILL_DIR}/scripts/run_workflow.py \
  {project_name} ...
```

### Step 2.3: Parse Script Output

The script outputs:

1. **Cache Status** - Shows which steps have cached data
2. **Execution Summary** - Lists steps that will be executed
3. **Token Savings Estimate** - Predicted token reduction
4. **Execution Plan JSON** - Saved to `{OUTPUT_DIR}/workflow_execution_result_{project_name}.json`

Claude reads this output and extracts:
- List of steps to execute
- List of steps to skip (cached)
- Token savings estimate

---

## 📋 PHASE 3: Execution Plan Display / 実行計画の表示

**Objective**: Show the user what will happen

Claude displays the plan to the user:

```
========================================
Execution Plan - {project_name}
========================================
Mode: {execution_mode}
Cache: {enabled/disabled}
Class UML (OMG + Eclipse UML2): always ON
Non-class UML (.uml): {enabled/disabled}

Phase A (Modeling - Steps 1-7):
  🟢 Step 1: scenario-to-activity-v1
  🟢 Step 2: activity-to-usecase-v1
  💾 Step 3: usecase-to-class-v1 (from cache)
  🟢 Step 4: class-to-statemachine-v1
  🟢 Step 5: usecase-to-sequence-v1
  🟢 Step 6: model-validator-v1
  🟢 Step 7: security-design-v1

Phase B (Code Generation - Steps 8-10):
  → Will execute in a NEW conversation after Phase A
  🟢 Step 8: usecase-to-code-v1
  🟢 Step 9: usecase-to-test-v1
  🟢 Step 10: traceability-matrix-v1

⚠️ Full Workflow = Phase A now → Phase B in new conversation
========================================
```

Claude then proceeds to Phase A execution automatically (no additional confirmation needed).

---

## 📋 PHASE 4: Step-by-Step Execution / ステップ実行

**Objective**: Execute each step with cache management

### Step 4.1: Load Execution Plan

Claude reads the execution plan from the JSON file:

```python
import json

with open(f'{OUTPUT_DIR}/workflow_execution_result_{project_name}.json') as f:
    plan = json.load(f)

steps_to_execute = plan['steps_executed']
steps_from_cache = plan['steps_from_cache']
```

### Step 4.2: Execute Each Step

For each step in the workflow:

#### General Pattern

```python
for step_name in workflow_steps:
    
    # Check if step should be executed
    if step_name in steps_to_skip:
        print(f"⏭️ Step skipped: {step_name}")
        continue
    
    # Check if cached
    if step_name in steps_from_cache:
        print(f"💾 Using cached: {step_name}")
        # Cache already restored by run_workflow.py
        continue
    
    # Execute the step
    print(f"⚙️ Executing: {step_name}")
    execute_sub_skill(step_name, project_name, config)
    
    # Cache the output
    cache_step_outputs(step_name, project_name)
```

> **Sub-skill reference**: Each step's full implementation is in its `PIPELINE.md` under `references/` (renamed from SKILL.md so the uploaded skill has exactly ONE SKILL.md at the root).
> Before executing a step, Claude MUST read: `view {SKILL_DIR}/references/{skill-name}/PIPELINE.md`

#### Step 1: scenario-to-activity-v1

**Execution**:

Claude reads `references/scenario-to-activity-v1/PIPELINE.md`, then executes:

```
Input: User's business scenario (from initial request)
Project name: {project_name}
Language: Auto-detect (Japanese/English)
Non-class UML (.uml): {config.generate_xmi}  # default OFF; activity .uml only if enabled
Output directory: {OUTPUT_DIR}
```

**Expected Outputs**:
- `{project_name}_activity-data.json`
- `{project_name}_activity.puml`
- `{project_name}_activity-model.uml` (**only if non-class UML output enabled**; default: not generated)

**Cache Storage** (automatic via run_workflow.py or manual):

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'scenario_to_activity', 'activity-data',
           '{OUTPUT_DIR}/{project_name}_activity-data.json')
cache_file('{project_name}', 'scenario_to_activity', 'activity-puml',
           '{OUTPUT_DIR}/{project_name}_activity.puml')
"
```

#### Step 2: activity-to-usecase-v1

**Pre-requisites Check**:

```bash
# Check if activity-data.json exists
if [ ! -f "{OUTPUT_DIR}/{project_name}_activity-data.json" ]; then
    # Try to restore from cache
    python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from unified_workflow_executor import UnifiedWorkflowExecutor
executor = UnifiedWorkflowExecutor('{project_name}')
from execution_mode_manager import SkillStep
executor.restore_from_cache(SkillStep.SCENARIO_TO_ACTIVITY)
"
fi
```

**Execution**:

Claude reads `references/activity-to-usecase-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_activity-data.json
Non-class UML (.uml): {config.generate_xmi}  # default OFF; usecase .uml only if enabled
```

**Expected Outputs**:
- `{project_name}_usecase-output.json`
- `{project_name}_usecase-diagram.puml`
- `usecase-specifications/*.md` (individual use case files)
- `{project_name}_usecase-model.uml` (**only if non-class UML output enabled**; default: not generated)

**Cache Storage**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'activity_to_usecase', 'usecase-output',
           '{OUTPUT_DIR}/{project_name}_usecase-output.json')
cache_file('{project_name}', 'activity_to_usecase', 'usecase-diagram',
           '{OUTPUT_DIR}/{project_name}_usecase-diagram.puml')
"
```

**⚠️ MANDATORY — Output Completeness Verification**:

After caching, Claude MUST verify that all use cases in JSON have corresponding `.md` files. This guards against LLM output truncation that has caused real-world incidents (e.g., library-management: 12 UCs in JSON but only 3 .md files generated).

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from verify_step_outputs import verify_step2_outputs

result = verify_step2_outputs('{project_name}')
print(f'JSON UC count: {result[\"json_uc_count\"]}')
print(f'.md file count: {result[\"md_file_count\"]}')
if not result['is_complete']:
    print('=== STEP 2 VERIFICATION FAILED ===')
    print(result['error_message'])
    if result['missing_uc_ids']:
        print('Missing UC IDs:', result['missing_uc_ids'])
    sys.exit(1)
print('✅ Step 2 verified: all UCs have spec files')
"
```

**If verification fails**: Claude MUST regenerate the missing `.md` files (read JSON, regenerate spec files for `missing_uc_ids` using the Cockburn template from `references/activity-to-usecase-v1/PIPELINE.md` Step 6b), then re-run the verification. Do NOT proceed to Step 3 until verification passes.

#### Step 3: usecase-to-class-v1

**Pre-requisites Check**:

```bash
# Needs: usecase-output.json AND activity-data.json (for original scenario)
```

**Execution**:

Claude reads `references/usecase-to-class-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_usecase-output.json
Original scenario: {project_name}_activity-data.json
Class UML (OMG + Eclipse UML2): ALWAYS generated (default ON)
```

**Expected Outputs**:
- `{project_name}_domain-model.json` ⭐ **CRITICAL - Single Source of Truth**
- `{project_name}_class.puml`
- `{project_name}_architecture-overview.md`
- `{project_name}_class-model.omg.uml` ⭐ **ALWAYS generated — OMG XMI format (.uml)**
- `{project_name}_class-model.emf.uml` ⭐ **ALWAYS generated — Eclipse UML2 (EMF/Papyrus) format (.uml)**

**Cache Storage**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'usecase_to_class', 'domain-model',
           '{OUTPUT_DIR}/{project_name}_domain-model.json')
cache_file('{project_name}', 'usecase_to_class', 'class-puml',
           '{OUTPUT_DIR}/{project_name}_class.puml')
"
```

#### Step 4: class-to-statemachine-v1

**Pre-requisites Check**:

```bash
# Needs: domain-model.json
```

**Execution**:

Claude reads `references/class-to-statemachine-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_domain-model.json
Language: Inherit from domain model
```

**Expected Outputs**:
- `{project_name}_statemachine.puml` (all state machines)
- `{project_name}_statemachine-{EntityName}.puml` (individual entities)

**Cache Storage**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'class_to_statemachine', 'statemachine-puml',
           '{OUTPUT_DIR}/{project_name}_statemachine.puml')
"
```

#### Step 5: usecase-to-sequence-v1

**Pre-requisites Check**:

```bash
# Needs: usecase-output.json AND domain-model.json
```

**Execution**:

Claude reads `references/usecase-to-sequence-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_usecase-output.json
Domain model: {project_name}_domain-model.json
Language: Inherit from use cases
```

**Expected Outputs**:
- `{project_name}_sequence.puml` (all sequences)
- `{project_name}_sequence-{UC-ID}.puml` (per use case)

**Cache Storage**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'usecase_to_sequence', 'sequence-puml',
           '{OUTPUT_DIR}/{project_name}_sequence.puml')
"
```

#### Step 6: model-validator-v1

**Pre-requisites Check**:

```bash
# Needs: All models in {OUTPUT_DIR}/{project_name}_*
```

**Execution**:

Claude reads `references/model-validator-v1/PIPELINE.md`, then executes:

```
Input: All generated models
Language: Inherit from models
```

**Expected Outputs**:
- `{project_name}_validation-report.md`
- `{project_name}_validation-summary.json`

**Cache Storage**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'model_validator', 'validation-report',
           '{OUTPUT_DIR}/{project_name}_validation-report.md')
"
```

#### Step 7: security-design-v1

**Pre-requisites Check**:

```bash
# Needs: domain-model.json AND usecase-output.json AND validation-report.md
```

**Execution**:

Claude reads `references/security-design-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_domain-model.json
       {project_name}_usecase-output.json
       {project_name}_validation-report.md (optional)
Language: Inherit from models
```

**Expected Outputs**:
- `{project_name}_security-design.md` (セキュリティ設計書)
- `{project_name}_security-config.json` (セキュリティ設定)

**Cache Storage**:

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import cache_file

cache_file('{project_name}', 'security_design', 'security-design',
           '{OUTPUT_DIR}/{project_name}_security-design.md')
cache_file('{project_name}', 'security_design', 'security-config',
           '{OUTPUT_DIR}/{project_name}_security-config.json')
"
```

**Security Design Scope**:
- 認証・認可設計（Authentication & Authorization）
- データ保護・暗号化方針
- API セキュリティ（Rate limiting, CORS, Input validation）
- OWASP Top 10 対策
- セキュリティロール・権限マトリクス
- 監査ログ設計

---

### ⚠️ AUTO-SPLIT CHECKPOINT — After Step 7 / Step 7完了後の自動分割

**CRITICAL**: After Step 7 completes, Claude MUST execute the following:

```
IF execution_mode == "full" AND steps_include(8, 9, 10):
    1. Cache all Step 1-7 outputs (if not already cached)
    2. Present all Phase A files to user via present_files
    3. Display Phase A completion summary:
    
    ========================================
    ✅ PHASE A COMPLETE (Modeling & Validation)
    ========================================
    Project: {project_name}
    Steps completed: 1-7
    
    Generated Artifacts:
      📄 Activity Diagram
      📋 Use Cases (JSON + MD specifications)
      📊 Class Diagram (domain-model.json — SSoT)
      🔄 State Machine Diagrams
      🔀 Sequence Diagrams
      ✅ Validation Report
      🔒 Security Design + Config
    
    All outputs cached for Phase B.
    
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    📢 NEXT: Start a NEW conversation and say:
    
    「uml-workflow-v3で{project_name}のStep 8から再開」
    
    Tech stack: {selected_backend} + {selected_frontend}
    Architecture: {selected_architecture}
    Tests: {yes/no}
    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    
    4. STOP. Do NOT proceed to Step 8.
    
ELSE IF execution_mode == "models_only":
    → Proceed to PHASE 5 (Completion)
    
ELSE IF execution_mode == "resume" AND start_step >= 8:
    → FIRST run PHASE 0.5 (Resume Mode Prerequisite Check) — see top of this file
    → Only after verification passes, proceed to Step 8 (Phase B)
```

#### Step 8: usecase-to-code-v1

**Pre-requisites Check**:

```bash
# Needs: domain-model.json AND usecase-output.json AND security-config.json
# These should be in cache from Phase A — restore if needed
python3 -c "
import sys, os
sys.path.append('{SKILL_DIR}/scripts')
from workflow_cache_helper import restore_all_cached_files
restore_all_cached_files('{project_name}')
"
ls {OUTPUT_DIR}/{project_name}_domain-model.json
ls {OUTPUT_DIR}/{project_name}_usecase-output.json
```

**⚠️ MANDATORY — Phase A Completeness Verification (fail-safe)**:

If PHASE 0.5 was skipped for any reason (e.g., Step 8 invoked directly without "resume" mode classification), run the same check here as a defensive measure. This is a no-op when PHASE 0.5 already passed.

```bash
python3 -c "
import sys
sys.path.append('{SKILL_DIR}/scripts')
from verify_step_outputs import verify_phase_a_completeness, format_phase_a_report

result = verify_phase_a_completeness('{project_name}')
print(format_phase_a_report(result))
if not result['is_complete']:
    sys.exit(1)
"
```

If this check fails, STOP and follow the PHASE 0.5 Decision Tree (top of this file).

**Execution**:

> ⚠️ **IMPORTANT — テックスタックの質問は Phase 1 で実施済み**
>
> Phase B（Step 8から再開）の場合、ユーザーのメッセージからテックスタック情報を取得する。
> Phase A完了時に表示されたテックスタック情報をユーザーがコピーして送ってくるため、
> そこから backend_framework, frontend_framework, architecture を読み取る。
> **Step 8 でユーザーに再度質問してはならない**。

Claude reads `references/usecase-to-code-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_domain-model.json
       {project_name}_usecase-output.json
       {project_name}_security-config.json
Tech stack: {phase1_selected_stack}
Architecture: {phase1_architecture}
```

**⚠️ フロントエンド生成の必須確認（最重要）**

`frontend_framework != "none"` の場合、コード生成後に以下を**必ず実行**して実際のファイル存在を確認すること:

```bash
find {WORK_DIR}/{project_name}/frontend/src -type f | sort
```

期待される最低限のファイル:
- `frontend/src/App.tsx` (または同等)
- `frontend/src/types/index.ts`
- `frontend/src/api/client.ts`
- `frontend/src/pages/` 以下に各ユースケース対応ページ

**もし上記ファイルが存在しない場合は、フロントエンドコードが未生成なので、即座に生成すること。Step 9 に進んではならない。**

**Expected Outputs**:
- Complete application in `{project_name}/` directory
  - `backend/` — ドメイン・サービス・ルート
  - `frontend/` — UI コンポーネント・ページ・API クライアント
  - `README.md`
  - `docker-compose.yml`

**Note**: Code generation outputs are typically NOT cached (too large and frequently modified)

#### Step 9: usecase-to-test-v1

**Pre-requisites Check**:

```bash
# Needs: domain-model.json AND usecase-output.json
```

**Execution**:

Claude reads `references/usecase-to-test-v1/PIPELINE.md`, then executes:

```
Input: {project_name}_domain-model.json
       {project_name}_usecase-output.json
Test frameworks: Jest/Vitest/Playwright/Cypress (auto-selected based on tech stack)
```

**Expected Outputs**:
- Unit tests
- Integration tests
- E2E tests
- Test documentation

---

#### Step 10: traceability-matrix-v1

**Pre-requisites Check**:

```bash
# Needs: usecase-output.json (required)
# Recommended: domain-model.json, security-config.json, src/, tests/
```

**Execution**:

Claude reads `references/traceability-matrix-v1/PIPELINE.md`, then executes:

```
Input: All pipeline artifacts in output directory
       {project_name}_usecase-output.json (required)
       usecase-specifications/UC-*.md (required)
       {project_name}_domain-model.json (recommended)
       {project_name}_security-config.json (recommended)
       src/ directory (if code generated)
       tests/ directory (if tests generated)
```

**Expected Outputs**:
- `{project_name}_traceability-matrix.json` (machine-readable full matrix)
- `{project_name}_traceability-matrix.md` (human-readable report with gap analysis)

**Note**: This step reads all previously generated artifacts. It does not participate in caching since it aggregates from other cached outputs.

---

## 📋 PHASE 5: Completion and Presentation / 完了・成果物提示

**Objective**: Present results to user and provide guidance

### Phase A Completion (Steps 1-7)

If executing Phase A (modeling), follow the AUTO-SPLIT CHECKPOINT above.
Present all model artifacts and instruct user to start Phase B.

### Phase B Completion (Steps 8-10) / Full Completion

Claude collects all generated files:

```bash
ls -la {OUTPUT_DIR}/{project_name}*
ls -la {WORK_DIR}/{project_name}/
```

Claude uses the present_files tool and displays:

```
========================================
✅ WORKFLOW COMPLETE
========================================
Project: {project_name}
Phases completed: A + B (Full)

Generated Artifacts:
  📄 UML Models: 8 files
  📊 Diagrams: 5 types
  💻 Application Code: backend + frontend
  🧪 Test Code: unit + integration + E2E
  📊 Traceability Matrix
  
Cache Status: All steps cached for next run.

💡 Next run: 
  - Add features: "uml-workflow-v3で{project_name}に{新機能}を追加"
  - Models only: Select "モデルのみ" mode
  - Validate only: Select "バリデーションのみ" mode
========================================
```

---

## 🚨 ERROR HANDLING / エラーハンドリング

- **Missing pre-requisites**: Auto-restore from cache → if fail, inform user to run from earlier step
- **Sub-skill failure**: Offer retry/skip/abort options via `ask_user_input_v0`
- **Cache corruption**: Clear cache with `clear_project_cache('{project_name}')` and re-run

## ✅ SUCCESS CRITERIA / 成功基準

1. ✅ All selected steps executed without errors
2. ✅ All expected output files generated
3. ✅ Validation report shows no critical issues
4. ✅ Files presented to user via present_files
5. ✅ Cache updated for next run

**This completes the SKILL.md implementation. Claude can now execute this workflow fully automatically.**

---
> Source: [atanaka/uml-workflow-v3](https://github.com/atanaka/uml-workflow-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
