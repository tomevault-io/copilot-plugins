## footnote

> 叙事系统开发规范


# 叙事系统开发规范

## 核心架构

### 系统组成
```
NarrativeEngine (核心引擎)
├── DialogueManager (对话管理)
├── CardSystem (卡片系统)
├── ForeshadowTracker (伏笔追踪)
├── WorldState (世界状态)
└── ZoneManager (区域管理)
```

### 数据流
```
YAML数据 → DataLoader → NarrativeEngine → UI渲染
                              ↓
                         WorldState
                              ↓
                         SaveManager
```

## 对话系统

### 对话数据格式 (YAML)
```yaml
# src/data/dialogues/c0.yaml
dialogues:
  C0_Z1_ENTRY:
    speaker: CENHUI
    text: CENHUI_MONO_01  # 引用对白词库
    next: C0_Z1_CHOICE_1
    
  C0_Z1_CHOICE_1:
    speaker: SYSTEM
    text: SYS_BASE_04
    choices:
      - label: "按流程执行"
        next: C0_Z1_FLOW
      - label: "先看看周围"
        next: C0_Z1_EXPLORE
        effect:
          r: 1  # 增加R值
          
  C0_Z1_FLOW:
    speaker: CENHUI
    text: CENHUI_MONO_02
    trigger:
      card: CARD_C0_02  # 触发获得卡片
    next: null  # 对话结束
```

### 对白键引用
```typescript
// 对白词库映射
interface IDialogueKeys {
  // 岑回
  CENHUI_MONO_01: '先按流程走。';
  CENHUI_MONO_02: '别急，记下来。';
  // ... 参考 design/对白词库 v1.md
}

// 渲染时解析
function resolveDialogueText(key: string): string {
  return dialogueKeys[key] ?? key;
}
```

### DialogueManager 接口
```typescript
interface IDialogueManager {
  // 开始对话
  start(dialogueId: string): Promise<void>;
  
  // 显示下一条
  next(): Promise<boolean>;
  
  // 选择分支
  choose(choiceIndex: number): Promise<void>;
  
  // 跳过当前对话
  skip(): void;
  
  // 事件
  on(event: 'start' | 'text' | 'choice' | 'end', callback: Function): void;
}
```

## 卡片系统

### 卡片数据格式
```yaml
# src/data/cards/c0.yaml
cards:
  CARD_C0_01:
    id: CARD_C0_01
    name: 身份识别卡：岑回
    type: archive
    chapter: C0
    zone: C0-Z1
    
    front:
      - "岑回 / 外勤巡检"
      - "通行级别：灰"
      - "所属：维修局外勤"
      - "签发日期：20██-██-██（已更正）"
    
    detail:
      - "备注：例外权限待评估"
      - "提示：按流程执行 / 避免越界"
      - ""
      - "校验行："
      - "— 人像比对：通过"
      - "— 指纹比对：通过"
      - "— 叙述一致性：通过（已对齐）"
      - ""
      - "（极淡小字，加载时闪 0.2s）"
      - "字段：——"
    
    fx:
      - type: taint
        target: "已更正"
        effect: smudge
      - type: flash
        target: "字段：——"
        duration: 0.2
    
    states:
      aligned:
        trigger: therapy_accepted
        override:
          detail:
            - "状态：已对齐"
            - "（内容已覆盖）"
      field_accepted:
        trigger: cf_z3_complete
        append:
          detail:
            - "字段：◦◦◦（已占位）"
```

### 卡片类型
```typescript
type CardType = 
  | 'archive'    // 档案类（身份、记录）
  | 'item'       // 物品类（旧灯芯等）
  | 'map'        // 地图类（版本差异）
  | 'prayer'     // 祷文类（首字链）
  | 'receipt'    // 回执类（系统生成）
  | 'verdict'    // 判词类（F21等）
  | 'diary';     // 日记类（阿棠碎片）
```

### CardSystem 接口
```typescript
interface ICardSystem {
  // 获得卡片
  collect(cardId: string): void;
  
  // 检查是否拥有
  has(cardId: string): boolean;
  
  // 获取卡片数据
  get(cardId: string): ICard | null;
  
  // 获取全部卡片（按章节/类型筛选）
  getAll(filter?: ICardFilter): ICard[];
  
  // 更新卡片状态
  updateState(cardId: string, state: string): void;
  
  // 事件
  on(event: 'collect' | 'stateChange', callback: Function): void;
}
```

## 伏笔系统

### 伏笔数据格式
```yaml
# src/data/foreshadows.yaml
foreshadows:
  F01:
    id: F01
    name: 薄墙回声
    
    stages:
      plant:
        zone: C0-Z3
        trigger: wall_listen
        description: 长按墙听回声
        
      deepen:
        zone: C2-Z2
        trigger: depth_perception_wall
        description: 深度感知看到空腔线框
        requires: [plant]
        
      misread:
        expected: "有人藏在墙里"
        truth: "表示压缩造成的结构空腔残留"
        
      resolve:
        zone: CF-Z3
        trigger: field_accepted
        description: 部分空腔不再被更正抹平
        requires: [deepen]
        
    assets:
      - echo_sound_effect
      - depth_wireframe_mask
      - settlement_card_structure_normal
```

### ForeshadowTracker 接口
```typescript
interface IForeshadowTracker {
  // 触发伏笔阶段
  trigger(foreshadowId: string, stage: 'plant' | 'deepen' | 'resolve'): void;
  
  // 检查伏笔状态
  getState(foreshadowId: string): ForeshadowState;
  
  // 检查是否可触发
  canTrigger(foreshadowId: string, stage: string): boolean;
  
  // 获取待回收伏笔
  getPendingResolutions(): IForeshadow[];
  
  // 验证终局伏笔回收
  validateEndgame(): IValidationResult;
}

interface ForeshadowState {
  planted: boolean;
  deepened: boolean;
  resolved: boolean;
  plantedAt?: string;  // Zone ID
  resolvedAt?: string;
}
```

## Zone系统

### Zone数据格式
```yaml
# src/data/zones/c0.yaml
zones:
  C0-Z1:
    id: C0-Z1
    name: 宿舍走廊
    chapter: C0
    type: life  # life | structural | causal | conflict
    
    focus: 2d_basic  # 玩法焦点
    
    characters:
      - CENHUI
      
    entry:
      dialogue: C0_Z1_ENTRY
      
    interactions:
      - id: identity_card
        type: examine
        position: [200, 400]
        trigger:
          dialogue: C0_Z1_CARD
          card: CARD_C0_01
          
      - id: notice_board
        type: examine
        position: [400, 300]
        trigger:
          foreshadow: [F03, plant]
          
    exits:
      - to: C0-Z2
        position: [700, 600]
        condition: dialogue_complete
        
    r_opportunities:
      - id: breakfast_special
        description: 点"今日特别"而非固定套餐
        r_value: 1
        
    p_cost: 0  # 本Zone的P消耗
```

### ZoneManager 接口
```typescript
interface IZoneManager {
  // 加载Zone
  load(zoneId: string): Promise<IZone>;
  
  // 进入Zone
  enter(zoneId: string): Promise<void>;
  
  // 退出Zone
  exit(): Promise<void>;
  
  // 获取当前Zone
  getCurrent(): IZone | null;
  
  // 检查Zone状态
  getState(zoneId: string): ZoneState;
  
  // 解锁Zone
  unlock(zoneId: string): void;
  
  // 标记完成
  complete(zoneId: string): void;
  
  // 事件
  on(event: 'enter' | 'exit' | 'complete', callback: Function): void;
}

type ZoneState = 'locked' | 'unlocked' | 'visited' | 'completed';
```

## 世界状态

### WorldState 结构
```typescript
interface IWorldState {
  // 隐藏计数器
  counters: {
    r: number;  // 无收益残差
    p: number;  // 观察者压力
    w: number;  // 世界可读性
  };
  
  // 章节进度
  chapter: ChapterID;
  
  // 能力解锁状态
  abilities: {
    depthPerception: boolean;
    depthIntervention: boolean;
    timeIntervention: boolean;
  };
  
  // Zone状态
  zones: Record<string, ZoneState>;
  
  // 对话完成记录
  dialoguesCompleted: Set<string>;
  
  // 选择记录（用于分支追踪）
  choices: IChoiceRecord[];
  
  // 深度伤痕记录
  depthScars: IDepthScar[];
  
  // 时间节点
  timeNodes: ITimeNode[];
  
  // 时间污染度
  timeContamination: number;
}
```

### R/P/W 计算规则
```typescript
// R (无收益残差) - 来源于无奖励行为
function calculateR(actions: IAction[]): number {
  return actions.filter(a => 
    a.type === 'no_reward' && 
    a.completed
  ).reduce((sum, a) => sum + (a.rValue ?? 1), 0);
}

// P (观察者压力) - 来源于能力使用
function calculateP(abilities: IAbilityUse[]): number {
  return abilities.reduce((sum, a) => {
    switch (a.type) {
      case 'DEPTH_PERCEPTION': return sum + 0;  // 感知不加P
      case 'DEPTH_INTERVENTION': return sum + 2;
      case 'TIME_INTERVENTION': return sum + 3;
      default: return sum;
    }
  }, 0);
}

// W (世界可读性) - 由R和P共同影响
function calculateW(r: number, p: number): number {
  const base = 100;
  const rPenalty = Math.floor(r / 3) * 5;  // R每3点降低5
  const pPenalty = Math.floor(p / 5) * 10; // P每5点降低10
  return Math.max(0, base - rPenalty - pPenalty);
}
```

## 结局判定

### 结局条件
```typescript
function determineEnding(state: IWorldState): EndingType {
  const { r, p, w } = state.counters;
  
  // 结局C: 成为系统（隐藏结局）
  if (p >= 15 && r >= 10 && w >= 30) {
    return 'C_BECOME_SYSTEM';
  }
  
  // 结局B: 真实释放
  if (p >= 20 || w < 30) {
    return 'B_RELEASE_TRUTH';
  }
  
  // 结局A: 平面稳定（默认）
  return 'A_STABLE_PLANE';
}
```

### F21判词触发条件
```typescript
function checkF21Trigger(state: IWorldState): boolean {
  // R ≥ 6 或完成指定无收益链
  return state.counters.r >= 6 || 
         hasCompletedNoRewardChain(state);
}

function hasCompletedNoRewardChain(state: IWorldState): boolean {
  const requiredActions = [
    'C2-Z4_SIGNPOST',      // 路标修补
    'C3-Z4_EMPTY_CHAIR',   // 空椅子
    'C4-Z6_USELESS_MAP',   // 无人需要的地图
  ];
  return requiredActions.every(id => 
    state.choices.some(c => c.id === id && c.completed)
  );
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/angelash) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
