## agentshire

> > 3D 小镇前端的场景管理、工作流编排、小游戏、NPC 行为、视觉效果。

# Game 层架构指南

> 3D 小镇前端的场景管理、工作流编排、小游戏、NPC 行为、视觉效果。

## 目录结构

```
town-frontend/src/game/
├── MainScene.ts           # 场景总控（1622行）：init + update循环 + 子系统编排
├── EventDispatcher.ts     # 65种GameEvent纯路由（switch → handler回调，零逻辑）
├── DialogManager.ts       # 对话流式显示 + 工作日志面板(activity/thinking/todo)
├── DailyScheduler.ts      # NPC日常调度 + AgentBrain绑定 + 夜间反思编排
├── SceneBootstrap.ts      # 启动流程（mock/live分支 + PublishedCitizenConfig加载）
├── GameClock.ts           # 24h循环（6时段，夜间3x加速，localStorage持久化）
├── WeatherSystem.ts       # 12种天气 + 10种日主题 + seeded random
├── DebugBindings.ts       # console调试绑定
│
├── minigame/              # ★ 小游戏系统
│   ├── MinigameSlot.ts        # 小游戏插槽接口（mount/start/stop/addWorkingNpc）
│   ├── BanweiGame.ts          # "班味消除"：orb生成/combo/boss/NPC压力/语音池
│   ├── BanweiRenderer.ts      # DOM渲染层：orb/boss/smoke/HUD/屏幕震动/粒子
│   └── BanweiNpcEffects.ts    # NPC压力视觉（去饱和+紫光+变形+减速）
│
├── workflow/              # 工作流编排
│   ├── Choreographer.ts       # 编排总控：接收Bridge的5种workflow_*意图事件
│   ├── BaseOrchestrator.ts    # 编排器基类（abort/delay/waitNpcArrival三级超时）
│   ├── WorkflowHandler.ts     # 状态存储 + 编排（工位分配/进场/完工/离场/恢复）
│   ├── SummonOrchestrator.ts  # 召唤集结（半弧形阵型+障碍物回避+传送兜底）
│   ├── BriefingOrchestrator.ts # 任务分配（逐个分配+列队行军+场景切换）
│   ├── CelebrationOrchestrator.ts # 庆祝（屏幕切换+彩纸+光柱+成果弹窗）
│   ├── SceneSwitcher.ts       # town/office/museum切换（fade+NPC迁移+防重入队列）
│   └── ModeManager.ts         # life/work模式 + 7个工作子状态
│
├── scene/                 # 3D 场景构建
│   ├── TownBuilder.ts         # 小镇（8建筑/14组路灯/20棵树/喷泉/花坛/长椅）
│   ├── OfficeBuilder.ts       # 办公室（10工位A-J + 访客区 + 白板）
│   ├── MuseumBuilder.ts       # 博物馆（6展台 + unlockStand效果）
│   ├── ScreenRenderer.ts      # 工位屏幕（Canvas2D→Texture，6种状态含代码动画）
│   └── VehicleManager.ts      # 车辆系统（对象池6辆，按时段调频，夜间开灯）
│
├── visual/                # VFX + 渲染
│   ├── VFXSystem.ts           # VFX Facade → 委托4个子模块
│   ├── SpawnEffects.ts        # 召唤冲击波 / 完成烟花 / 错误闪电 / 人格变换(4.2s)
│   ├── WorkEffects.ts         # 思考光环 / 工作粒子 / 文件图标 / 搜索雷达 / 连接光束
│   ├── CelebrationEffects.ts  # 部署烟花(5连发) / 彩纸(200个) / 光柱 / 技能仪式(8s)
│   ├── DebugEffects.ts        # 路径调试可视化
│   ├── Effects.ts             # 基础粒子（ripple/stars/sparks/pillar/exclamation）
│   ├── ParticlePool.ts        # GPU粒子池（512个，AdditiveBlending）
│   ├── EffectRegistry.ts      # 特效注册表 + 更新循环 + 相机震动
│   ├── CameraController.ts    # 跟随/拖拽/缩放/巡逻/办公室模式/动画过渡
│   ├── TimeOfDayLighting.ts   # 昼夜光照（10关键帧，smoothstep插值，天气覆盖）
│   ├── WeatherEffects.ts      # GLSL粒子天气（雨25000/雪12000/沙10000/水花/积雪/地雾/极光）
│   ├── PostProcessing.ts      # 后处理（RenderPass + UnrealBloomPass）
│   └── AssetLoader.ts         # GLTF加载器（批次并行6个，builtin/library/custom三源）
│
└── __tests__/
    ├── EventDispatcher.test.ts
    ├── DialogManager.test.ts
    ├── SceneSwitcher.test.ts
    ├── GameClock.test.ts
    ├── ModeManager.test.ts
    └── SceneBootstrap.loadConfig.test.ts
```

## 数据流

```
DataSource (Mock / DirectorBridge)
 │ onGameEvent
 ▼
MainScene.handleGameEvent()
 │ 委托
 ▼
EventDispatcher.dispatch(event)
 ├─ npc_spawn/despawn/phase/emoji/glow/anim  → MainScene NPC操作
 ├─ npc_work_done                             → WorkflowHandler.handleNpcWorkDone()
 ├─ workflow_summon/assign/go_office/publish/return → Choreographer.handleIntent()
 ├─ dialog_message/dialog_end                 → DialogManager
 ├─ npc_activity/activity_stream/activity_todo → DialogManager（工作日志）
 ├─ scene_switch                              → SceneSwitcher
 ├─ mode_change                               → ModeManager
 ├─ fx                                        → MainScene.onFx → VFXSystem
 ├─ restore_work_state                        → WorkflowHandler
 ├─ deliverable_card                          → MediaPreview
 ├─ skill_learned                             → SkillLearnCard + 技能仪式VFX
 └─ set_time/set_weather                      → GameClock / WeatherSystem
```

## MainScene update 循环

```
update(deltaTime):
  1. 门廊交互检测（pendingDoorInteraction → 距离检查 → 场景切换）
  2. gameClock.update()
  3. 按场景分支：
     town:   cameraCtrl + timeOfDayLighting + weatherSystem + vehicleManager
     office: cameraCtrl.updateOfficePan
     museum: cameraCtrl.update
  4. followBehavior + citizenChat（ms 单位）
  5. timeHUD
  6. 音频：ambientSound(仅town) + bgm(天气/时段/场景)
  7. npcManager.update()
  8. 仅town: DailyBehavior.update + encounterManager + casualEncounter
  9. vfx + bubbles + officeBuilder.updateScreens + minigame
```

## 小游戏系统

`MinigameSlot` 接口解耦小游戏与主场景。`BanweiGame` 是首个实现：

- **Orb 系统**：NPC 工作时 5-30s 随机生成班味球（light/medium/heavy），每 NPC 最多 6 个
- **Combo 系统**：连击 ≥2/4/7/10 触发不同级别文案，每档 30 条网络梗
- **Boss 系统**：每 6 个 orb 生成 Boss，3 阶段循环，有 HP 条和 dash 攻击
- **NPC 压力**：orb 数量 → stress 值 → 去饱和+紫光+变形+减速
- **生命周期**：work 模式进入 working 时 `start()`，离开时 `stop()`

## NPC 完成工作离场序列

Bridge 发 `npc_work_done`，前端 `WorkflowHandler.handleNpcWorkDone()` 编排：

1. 失败 → emoting(frustrated) + error VFX + 工位屏 error
2. 成功 → celebrating(cheer) + completionFirework + 绿色 glow
3. 等 LoopOnce 播完自动回 idle
4. 清除状态（emoji/glow/indicator）
5. 临时工 → fadeOut + despawn；常驻居民 → 说收工语 → walk 到门口 → fadeOut → 迁回小镇 → 恢复 DailyBehavior
6. 清工位屏 + 发 `workstation_released` 通知 Bridge 释放

## NPC 状态机

通过 `NPC.transitionTo(state)` 驱动，`STATE_TRANSITIONS` 限制合法转换。

| 状态 | 动画 | 自动转换 |
|------|------|---------|
| idle | idle | — |
| walking | walk | moveTo 到达后回 idle |
| working | typing/reading | — |
| thinking | idle(0.5x) | — |
| celebrating | cheer | LoopOnce 播完回 idle |
| emoting | wave/frustrated/dancing | LoopOnce 播完回 idle |
| departing | walk | 到达后由外部处理 |

`transitionCharacterKey()` 支持带淡入淡出的角色模型热切换。

## 天气系统

`WeatherSystem` 管理 12 种天气（clear/cloudy/overcast/foggy/drizzle/rain/heavyRain/storm/snow/blizzard/sandstorm/aurora），每天通过 seeded random 按权重滚 10 种日主题，每时段映射到具体天气。中间态路由确保平滑过渡（暴风→晴天需经 rain→cloudy→clear）。

天气驱动三个系统：
- `TimeOfDayLighting`：sunMul / ambMul / tint 覆盖
- `WeatherEffects`：GLSL 粒子（雨/雪/沙/地雾/极光等）
- `AmbientSoundManager`：程序合成音频层

## 常见改动

| 要做的事 | 改哪里 |
|---------|--------|
| 新增 GameEvent 处理 | `EventDispatcher.ts` 加 case + handler |
| 修改对话气泡 | `DialogManager.ts` |
| 修改场景切换 | `workflow/SceneSwitcher.ts` |
| 修改NPC日常 | `DailyScheduler.ts` + `npc/DailyBehavior.ts` |
| 修改启动流程 | `SceneBootstrap.ts` |
| 修改NPC离场动画 | `WorkflowHandler.handleNpcWorkDone()` |
| 修改工作流主线编排 | `Choreographer.ts` → 对应 Orchestrator |
| 修改小游戏 | `minigame/BanweiGame.ts`（逻辑）+ `BanweiRenderer.ts`（渲染） |
| 新增小游戏 | 实现 `MinigameSlot` 接口 → `MainScene.initModeSystem()` 注册 |
| 修改NPC状态/动画 | `npc/NPC.ts` — `transitionTo()` 驱动 |
| 新增NPC状态 | `NpcState` 类型 + `STATE_TRANSITIONS` + `onEnterState` |
| 修改天气 | `WeatherSystem.ts`(状态机) + `WeatherEffects.ts`(视觉) |
| 修改VFX | `visual/` 下对应子模块 + `VFXSystem.ts` Facade |
| 修改相机 | `visual/CameraController.ts` |
| 修改光照 | `visual/TimeOfDayLighting.ts` |
| 修改建筑/场景 | `scene/TownBuilder.ts` 或对应 Builder |
| 新增VFX触发 | `MainScene.onFx()` 的 switch |

---
> Source: [Agentshire/Agentshire](https://github.com/Agentshire/Agentshire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
