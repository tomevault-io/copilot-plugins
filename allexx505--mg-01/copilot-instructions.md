## mg-01

> 编辑游戏代码时自动加载的上下文规则


# 游戏代码开发规则

## game.js 代码段落索引（类 → 行号范围）
- 常量/枚举: 8-28 | 工具类/Vec2: 29-48
- 粒子/特效: 49-107 | 摄像机(Camera): 108-129
- 平台(Platform): 130-161 | 光柱(LightBeam): 162-237
- 动态光柱(MovingBeam): 238-289 | 信件碎片(LetterFragment): 290-358
- 灵猫AI(Cat): 359-558 | 亡魂NPC(Soul): 559-710
- 玩家(Player): 711-918 | 存档(GameSave): 919-968
- 功德(MeritSystem): 969-979 | 检查点(Checkpoint): 980-1042
- 第1层数据(Level1): 1043-1081 | 第2层数据(Level2): 1082-1137
- 圆角矩形: 1138-1152 | 教学系统(Tutorial): 1153-1303
- Game主类: 1304-2506 | 启动代码: 2507-2512

## 关键约束
- 单文件架构，无 import/export
- `window.game` 测试钩子不可删除（Playwright 依赖）
- 所有实体类必须有 update(dt) 和 draw(ctx, camX) 方法

## 物理常量速查
GRAVITY=900 | PLAYER_SPEED=200 | PLAYER_SPRINT=300 | JUMP_VELOCITY=-420 | SPRINT_COST=10

## 状态枚举速查
STATE: TITLE, PLAYING, DIALOGUE, WIN, TRANSITION
PLAYER_STATE: ALIVE, DISSOLVING, FLATTENED, FROZEN

## 开发规则
- 添加新系统前，先确认 specs/systems/ 有对应 spec
- 添加新关卡数据前，先确认 specs/levels/ 有对应 spec
- 需要深入了解架构时，读 ARCHITECTURE.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allexx505) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
