## fc-import-map

> FC 语言完整语法和 API 参考


# FC 语言 Import 映射表

使用原则：每次使用函数前，查询此表补充 import 语句

---

## 核心规则

### 必须 Import 的情况
1. 非 StdLibrary 的函数 - 必须查表并添加 import
2. 非 StdLibrary 的事件 - 必须查表并添加 import（重要！）
3. List、Map 操作 - 必须 import 对应库
4. 数学计算函数 - 必须 import "Math.fcc"
5. 物理检测函数 - 必须 import "Physics.fcc"
6. 战斗相关函数 - 必须 import "Combat.fcc"

### 无需 Import 的情况
1. StdLibrary 内置函数 - 如 LogInfo(), WaitForMillisecond(), GetAllPlayers()
2. StdLibrary 内置事件 - 如 OnAwake(), OnGameStart(), OnUpdate(), OnDestroy()
3. 语言关键字和基本类型 - 如 Vector3, Quaternion, bool, int
4. 编辑器生成的枚举/资源 - 使用 import "EditorGenLib.fcc"

### 事件 Import 示例
```fc
// 错误：缺少 import
graph ButtonHandler {
    event OnTapped(player entity<Player>) {  // 错误：OnTapped 需要 import "Hud.fcc"
        LogInfo("Button tapped")
    }
}

// 正确：添加 import
import "StdLibrary.fcc" as std
import "Hud.fcc" as hud                      // 添加 import

graph ButtonHandler {
    event OnTapped(player entity<Player>) {  // 正确
        LogInfo("Button tapped")
    }
}
```

---

## 函数到库映射表

### List 操作（List.fcc）
```fc
import "List.fcc" as list

New()           → list.New(length, capacity)
Append()        → list.Append(targetList, value)
Length()        → list.Length(targetList)
RemoveAt()      → list.RemoveAt(targetList, index)
Clear()         → list.Clear(targetList)
Reverse()       → list.Reverse(targetList)
Clone()         → list.Clone(targetList)
Insert()        → list.Insert(targetList, index, value)
Shuffle()       → list.Shuffle(targetList)
IsEqual()       → list.IsEqual(listA, listB)
Contain()       → list.Contain(targetList, value)
AppendRange()   → list.AppendRange(targetList, sourceList)
Remove()        → list.Remove(targetList, value)
IndexOf()       → list.IndexOf(targetList, value)
LastIndexOf()   → list.LastIndexOf(targetList, value)
Max()           → list.Max(targetList, out isSuccess, out result)
Min()           → list.Min(targetList, out isSuccess, out result)
```

### Map 操作（Map.fcc）
```fc
import "Map.fcc" as map

New()           → map.New()
Remove()        → map.Remove(targetMap, key)
Clear()         → map.Clear(targetMap)
Length()        → map.Length(targetMap)
ContainKey()    → map.ContainKey(targetMap, key)    // 也可用内置的 ContainKey()
GetAllKeys()    → map.GetAllKeys(targetMap)
```

### 数学运算（Math.fcc）
```fc
import "Math.fcc" as math

// 常量
NaturalConstantType.Pi  → 3.14159265
NaturalConstantType.E   → 2.71828183

// 随机数
RandomInt()             → math.RandomInt(min, max)
RandomFloat()           → math.RandomFloat(min, max)
CreateRandomSeed()      → math.CreateRandomSeed(seed)
Next()                  → math.Next(randomGen, min, max)

// 基础数学
Abs()                   → math.Abs(value)
Sqrt()                  → math.Sqrt(value)
Negate()                → math.Negate(value)
Floor()                 → math.Floor(value)
Ceil()                  → math.Ceil(value)
Round()                 → math.Round(value)
Max()                   → math.Max(a, b)
Min()                   → math.Min(a, b)
Clamp()                 → math.Clamp(value, min, max)
Lerp()                  → math.Lerp(a, b, t)

// 三角函数
Sin()                   → math.Sin(degree)
Cos()                   → math.Cos(degree)
Tan()                   → math.Tan(degree)
ASin()                  → math.ASin(value)
ACos()                  → math.ACos(value)
ATan2()                 → math.ATan2(y, x)

// 幂运算
Exponentiation()        → math.Exponentiation(base, exponent)
Logarithm()             → math.Logarithm(baseValue, realNumber)

// Vector3 运算
Dot()                   → math.Dot(vecA, vecB)
Cross()                 → math.Cross(vecA, vecB)
Normalize()             → math.Normalize(vec)
Magnitude()             → math.Magnitude(vec)
Distance()              → math.Distance(vecA, vecB)
Angle()                 → math.Angle(vecA, vecB)
Projection()            → math.Projection(vecA, vecB)
LerpVector3()           → math.LerpVector3(vecA, vecB, t)
SlerpVector3()          → math.SlerpVector3(vecA, vecB, t)
Vector3Reflect()        → math.Vector3Reflect(direction, normal)

// Vector2 运算
Vector2Dot()            → math.Vector2Dot(vecA, vecB)
Vector2Distance()       → math.Vector2Distance(vecA, vecB)
Vector2Angle()          → math.Vector2Angle(vecA, vecB)
Vector2Normalize()      → math.Vector2Normalize(vec)
Vector2Magnitude()      → math.Vector2Magnitude(vec)
Vector2Reflect()        → math.Vector2Reflect(direction, normal)

// 四元数运算
QuaternionToEulerAngle()    → math.QuaternionToEulerAngle(quat)
EulerAngleToQuaternion()    → math.EulerAngleToQuaternion(euler)
AxisAngleToQuaternion()     → math.AxisAngleToQuaternion(axis, angle)
QuaternionToAxisAngle()     → math.QuaternionToAxisAngle(quat, out axis, out angle)
MultiplyQuaternion()        → math.MultiplyQuaternion(quatA, quatB)
LookRotation()              → math.LookRotation(forward, upward)
SlerpQuaternion()           → math.SlerpQuaternion(quatA, quatB, t)
QuaternionAngle()           → math.QuaternionAngle(quatA, quatB)
QuaternionDot()             → math.QuaternionDot(quatA, quatB)
QuaternionInverse()         → math.QuaternionInverse(quat)
QuaternionFromToRotation()  → math.QuaternionFromToRotation(from, to)
QuaternionRotateTowards()   → math.QuaternionRotateTowards(from, to, maxDegrees)
RotateVector()              → math.RotateVector(vec, deltaQuat)

// 方向转换
DirectionToEulerAngle()     → math.DirectionToEulerAngle(direction)
```

### 字符串操作（Strings.fcc）
```fc
import "Strings.fcc" as strings

Format()        → strings.Format(content, paramsList)
FormatStr()     → strings.FormatStr(content, paramsList)
Connect()       → strings.Connect(left, right)
Length()        → strings.Length(str)
Replace()       → strings.Replace(target, oldValue, newValue)
Find()          → strings.Find(target, findValue, startIndex)
Sub()           → strings.Sub(target, startIndex, length)
Split()         → strings.Split(target, sepStr)
FindAt()        → strings.FindAt(target, indexsList)
ToUpper()       → strings.ToUpper(target)
ToLower()       → strings.ToLower(target)
Strip()         → strings.Strip(target)
Join()          → strings.Join(stringList, separator)
```

### 物理检测（Physics.fcc）
```fc
import "Physics.fcc" as physics

// 射线检测
SingleRaycast()     → physics.SingleRaycast(startPos, direction, distance, layerMask, includeTrigger, out hitEntity, out hitPoint, out hitDistance, out hitNormal)
MultiRaycast()      → physics.MultiRaycast(startPos, direction, distance, layerMask, includeTrigger, out hitEntities, out hitPoints, out hitDistances, out hitNormals)

// 区域检测 - Cast（移动扫描）
BoxCast()           → physics.BoxCast(halfExtents, center, rotation, direction, maxDistance, layerMask, includeTrigger, out hitEntities)
SphereCast()        → physics.SphereCast(radius, center, direction, maxDistance, layerMask, includeTrigger, out hitEntities)
CapsuleCast()       → physics.CapsuleCast(radius, height, center, rotation, direction, maxDistance, layerMask, includeTrigger, out hitEntities)

// 区域检测 - Overlap（静态范围）
BoxOverlap()        → physics.BoxOverlap(halfExtents, center, rotation, layerMask, includeTrigger, out hitEntities)
SphereOverlap()     → physics.SphereOverlap(radius, center, layerMask, includeTrigger, out hitEntities)
CapsuleOverlap()    → physics.CapsuleOverlap(radius, height, center, rotation, layerMask, includeTrigger, out hitEntities)

// 区域判断
IsInsideSphere()    → physics.IsInsideSphere(entity, center, radius)
IsInsideCylinder()  → physics.IsInsideCylinder(entity, center, radius, height, rotation)
IsInsideBox()       → physics.IsInsideBox(entity, center, length, width, height, rotation)
IsInsideSector()    → physics.IsInsideSector(entity, center, radius, cutRadius, height, angle, rotation)

// 刚体力学
AddForce()              → physics.AddForce(target, force, forceMode)
AddForceAtPosition()    → physics.AddForceAtPosition(target, force, position, forceMode)
AddTorque()             → physics.AddTorque(target, torque, forceMode)
MovePosition()          → physics.MovePosition(target, position)
MoveRotation()          → physics.MoveRotation(target, rotation)

// 刚体状态
Sleep()                     → physics.Sleep(target)
Wakeup()                    → physics.Wakeup(target)
IsSleeping()                → physics.IsSleeping(target)
SetSleepThreshold()         → physics.SetSleepThreshold(target, threshold)
GetSleepThreshold()         → physics.GetSleepThreshold(target)
SetGlobalSleepThreshold()   → physics.SetGlobalSleepThreshold(threshold)
GetGlobalSleepThreshold()   → physics.GetGlobalSleepThreshold()

// 角色控制器
MoveCCT()       → physics.MoveCCT(target, movement)
TeleportCCT()   → physics.TeleportCCT(target, position)
```

### 战斗系统（Combat.fcc）
```fc
import "Combat.fcc" as combat

DealDamage()            → combat.DealDamage(target, attacker, damageType, damage)
Eliminate()             → combat.Eliminate(target)
EliminateAtNextFrame()  → combat.EliminateAtNextFrame(target)
```

### 类型转换（Convert.fcc）
```fc
import "Convert.fcc" as convert

ToString()              → convert.ToString(target)
IntToFloat()            → convert.IntToFloat(value)
NumToString()           → convert.NumToString(value, style)
StringToInt()           → convert.StringToInt(target, out result, out isSuccess)
StringToFloat()         → convert.StringToFloat(target, out result, out isSuccess)
Vector3ToVector2()      → convert.Vector3ToVector2(vec3)
Vector2ToVector3()      → convert.Vector2ToVector3(vec2)
EulerAngleToVector3()   → convert.EulerAngleToVector3(euler)
BitwiseListToInt()      → convert.BitwiseListToInt(bitArray)
BitwiseIntToList()      → convert.BitwiseIntToList(value)
```

### 时间系统（Time.fcc）
```fc
import "Time.fcc" as time

GetLocalTimestamp()     → time.GetLocalTimestamp()       // [platform_client]
PauseFixedUpdate()      → time.PauseFixedUpdate()
ResumeFixedUpdate()     → time.ResumeFixedUpdate()
```

### 动画系统（Animation.fcc）
```fc
import "Animation.fcc" as anim

PlayAnimation()         → anim.PlayAnimation(target, state, playSpeed, loopType)
StopAnimation()         → anim.StopAnimation(target, state)
SetBodyPartWeight()     → anim.SetBodyPartWeight(target, bodyPartName, weight)
```

---

## 事件到库映射表

### UI 事件（Hud.fcc）
```fc
import "Hud.fcc" as hud

// 按钮交互事件
OnTapped()                  → import "Hud.fcc"
OnClick()                   → import "Hud.fcc"
OnPressed()                 → import "Hud.fcc"
OnPress()                   → import "Hud.fcc"
OnReleased()                → import "Hud.fcc"
OnRelease()                 → import "Hud.fcc"
OnDoubleTapped()            → import "Hud.fcc"
OnDoubleClick()             → import "Hud.fcc"

// UI 状态事件
OnUIShow()                  → import "Hud.fcc"
OnHudShow()                 → import "Hud.fcc"
OnUIHide()                  → import "Hud.fcc"
OnHudHide()                 → import "Hud.fcc"

// 其他 UI 事件
OnTextChange()              → import "Hud.fcc"
OnCountdownFinished()       → import "Hud.fcc"
OnBuiltInCloseButtonTapped() → import "Hud.fcc"
OnBuiltInButtonTapped()     → import "Hud.fcc"
OnInternalButtonClick()     → import "Hud.fcc"
OnCameraRaycast()           → import "Hud.fcc"
OnClientCameraRayCast()     → import "Hud.fcc"
OnFriendsUpdate()           → import "Hud.fcc"
OnFriendRelationChange()    → import "Hud.fcc"
```

### 玩家事件（Player.fcc）
```fc
import "Player.fcc" as player

// 玩家连接事件
OnPlayerJoin()              → import "Player.fcc"
OnPlayerQuit()              → import "Player.fcc"
OnDisconnect()              → import "Player.fcc"
OnDisconnected()            → import "Player.fcc"
OnReconnect()               → import "Player.fcc"
OnReconnected()             → import "Player.fcc"
OnUserQuitOrUserMacthEnd()  → import "Player.fcc"

// 玩家动作事件
OnJump()                    → import "Player.fcc"
OnJumpStop()                → import "Player.fcc"
OnCrouch()                  → import "Player.fcc"
OnCreep()                   → import "Player.fcc"
OnSprint()                  → import "Player.fcc"
OnSprintStop()              → import "Player.fcc"
OnDash()                    → import "Player.fcc"
OnMove()                    → import "Player.fcc"
OnIdle()                    → import "Player.fcc"
OnFall()                    → import "Player.fcc"
OnFallStop()                → import "Player.fcc"
OnStandUP()                 → import "Player.fcc"

// 玩家状态事件
OnPlayerRevived()           → import "Player.fcc"
OnPlayerRevive()            → import "Player.fcc"
OnRevived()                 → import "Player.fcc"
OnSurfing()                 → import "Player.fcc"
OnSurfingStop()             → import "Player.fcc"
OnStartFire()               → import "Player.fcc"
OnUseAvatarSkill()          → import "Player.fcc"
```

### 战斗事件（Combat.fcc）
```fc
import "Combat.fcc" as combat

// 伤害和击杀事件
OnPreTakeDamage()           → import "Combat.fcc"
OnTakeDamage()              → import "Combat.fcc"
OnPreDealDamage()           → import "Combat.fcc"
OnDealDamage()              → import "Combat.fcc"
OnEliminated()              → import "Combat.fcc"
OnEliminate()               → import "Combat.fcc"
OnKnockedDown()             → import "Combat.fcc"
OnBeKnockedDown()           → import "Combat.fcc"
OnAssist()                  → import "Combat.fcc"

// 全局战斗事件
OnPlayerBeEliminated()      → import "Combat.fcc"
OnPlayerDead()              → import "Combat.fcc"
OnPlayerAssist()            → import "Combat.fcc"
OnPlayerDealDamage()        → import "Combat.fcc"
OnPlayerDamage()            → import "Combat.fcc"
OnPlayerEliminate()         → import "Combat.fcc"
OnPlayerKill()              → import "Combat.fcc"
OnPlayerBeKnockedDown()     → import "Combat.fcc"
OnTeamAce()                 → import "Combat.fcc"
OnACE()                     → import "Combat.fcc"
```

### 物理事件（Physics.fcc）
```fc
import "Physics.fcc" as physics

// 碰撞事件
OnCollisionEnter()          → import "Physics.fcc"
OnCollisionExit()           → import "Physics.fcc"
OnCharacterControllerHit()  → import "Physics.fcc"
OnControllerColliderHit()   → import "Physics.fcc"

// 触发器事件
OnEntityEnter()             → import "Physics.fcc"
OnEntityEnterTrigger()      → import "Physics.fcc"
OnEntityExit()              → import "Physics.fcc"
OnEntityLeaveTrigger()      → import "Physics.fcc"

// 刚体事件
OnWakeUp()                  → import "Physics.fcc"
OnWakeup()                  → import "Physics.fcc"
OnSleep()                   → import "Physics.fcc"
```

### 物品和武器事件（Items.fcc）
```fc
import "Items.fcc" as items

// 装备事件
OnEquipArmor()              → import "Items.fcc"
OnEquipWeapon()             → import "Items.fcc"
OnWeaponBeEquiped()         → import "Items.fcc"
OnBeEquiped()               → import "Items.fcc"
OnEquiped()                 → import "Items.fcc"
OnEquip()                   → import "Items.fcc"

// 使用物品事件
OnUseConsumable()           → import "Items.fcc"
OnUseItem()                 → import "Items.fcc"
OnUseSupply()               → import "Items.fcc"
OnUsed()                    → import "Items.fcc"

// 获取和丢弃事件
OnAccessItem()              → import "Items.fcc"
OnObtainItem()              → import "Items.fcc"
OnAccessWeapon()            → import "Items.fcc"
OnObtainWeapon()            → import "Items.fcc"
OnAccessEquipment()         → import "Items.fcc"
OnPlayerEquip()             → import "Items.fcc"
OnDropWeapon()              → import "Items.fcc"
OnDropConsumable()          → import "Items.fcc"
OnDropItem()                → import "Items.fcc"
OnPickedUp()                → import "Items.fcc"
OnPickup()                  → import "Items.fcc"
OnDropped()                 → import "Items.fcc"
OnDrop()                    → import "Items.fcc"

// 武器操作事件
OnFire()                    → import "Items.fcc"
OnReload()                  → import "Items.fcc"
OnWeaponStopFiring()        → import "Items.fcc"
OnStopFire()                → import "Items.fcc"
OnAiming()                  → import "Items.fcc"
OnAimed()                   → import "Items.fcc"
OnSwitchHandHeld()          → import "Items.fcc"
OnPlayerHandHeld()          → import "Items.fcc"

// 武器击中事件
OnBulletHit()               → import "Items.fcc"
OnHit()                     → import "Items.fcc"
OnHitByBullet()             → import "Items.fcc"
OnHitted()                  → import "Items.fcc"

// 特殊物品事件
OnLandmineActive()          → import "Items.fcc"
OnLandmineTrigger()         → import "Items.fcc"
OnHookGunFire()             → import "Items.fcc"
OnFires()                   → import "Items.fcc"
OnThrowableExplode()        → import "Items.fcc"
```

### 关卡物件事件（LevelObject.fcc）
```fc
import "LevelObject.fcc" as levelobject

// 玩家进入/离开事件
OnPlayerEnter()             → import "LevelObject.fcc"
OnPlayerEnterTrigger()      → import "LevelObject.fcc"
OnPlayerLeave()             → import "LevelObject.fcc"
OnPlayerLeaveTrigger()      → import "LevelObject.fcc"

// 载具事件
OnEnterVehicle()            → import "LevelObject.fcc"
OnEnteringVehicle()         → import "LevelObject.fcc"
OnLeaveVehicle()            → import "LevelObject.fcc"
OnLeavingVehicle()          → import "LevelObject.fcc"
OnDriveVehicle()            → import "LevelObject.fcc"
OnDrivingVehicle()          → import "LevelObject.fcc"
OnEntered()                 → import "LevelObject.fcc"
OnBeEntered()               → import "LevelObject.fcc"
OnLeft()                    → import "LevelObject.fcc"
OnBeLeft()                  → import "LevelObject.fcc"
OnDriven()                  → import "LevelObject.fcc"
OnBeDrived()                → import "LevelObject.fcc"
OnDrived()                  → import "LevelObject.fcc"

// 机关事件
OnUsePortal()               → import "LevelObject.fcc"
OnTeleporting()             → import "LevelObject.fcc"
OnTeleportEntity()          → import "LevelObject.fcc"
OnBeTeleported()            → import "LevelObject.fcc"
OnBouncedByTire()           → import "LevelObject.fcc"
OnBouncesOffTire()          → import "LevelObject.fcc"
OnBounceEntity()            → import "LevelObject.fcc"
OnBeBouncesOffTire()        → import "LevelObject.fcc"
OnSpringing()               → import "LevelObject.fcc"
OnHitFly()                  → import "LevelObject.fcc"
OnBeHitFly()                → import "LevelObject.fcc"
OnHitFlyEntity()            → import "LevelObject.fcc"
OnHitFlying()               → import "LevelObject.fcc"
OnExplode()                 → import "LevelObject.fcc"
```

### 场景事件（Scene.fcc）
```fc
import "Scene.fcc" as scene

OnSceneLoad()               → import "Scene.fcc"
OnSceneLoadSuccessed()      → import "Scene.fcc"
OnSceneSwitch()             → import "Scene.fcc"
OnSceneSwitchSuccessed()    → import "Scene.fcc"
OnPlayerSwitchScene()       → import "Scene.fcc"
OnSwitchArea()              → import "Scene.fcc"
```

### 工作流事件（Workflow.fcc）
```fc
import "Workflow.fcc" as workflow

OnRoundStart()              → import "Workflow.fcc"
OnRoundEnd()                → import "Workflow.fcc"
OnPhaseStart()              → import "Workflow.fcc"
OnPhaseEnd()                → import "Workflow.fcc"
```

### 播放器事件（Playable.fcc）
```fc
import "Playable.fcc" as playable

OnEachLoop()                → import "Playable.fcc"
OnStart()                   → import "Playable.fcc"
OnEnd()                     → import "Playable.fcc"
OnResume()                  → import "Playable.fcc"
OnPause()                   → import "Playable.fcc"
```

### 输入系统事件（InputSystem.fcc）
```fc
import "InputSystem.fcc" as inputsystem

OnScreenTouchBegin()        → import "InputSystem.fcc"
OnScreenTouchEnd()          → import "InputSystem.fcc"
```

### 其他事件
```fc
// Morph.fcc
OnLevelObjectBeMorphed()    → import "Morph.fcc"
OnLevelObjectToBeMorphed()  → import "Morph.fcc"

// Premium.fcc
OnPremiumChange()           → import "Premium.fcc"
OnPlayerEnterPremiumCenter() → import "Premium.fcc"
OnPlayerLeavePremiumCenter() → import "Premium.fcc"

// Time.fcc
OnFixedUpdate()             → import "Time.fcc"
OnGamePause()               → import "Time.fcc"
OnGameResume()              → import "Time.fcc"
```

---

## 内置函数和事件（无需 Import）

以下函数和事件在 StdLibrary 中，可以直接使用，无需 import：

### 内置函数
```fc
// 日志
LogInfo(content)
LogWarning(content)
LogError(content)

// 时间等待
WaitForMillisecond(time)
WaitForSecond(time)
WaitForFrame(frameCount)
WaitForNextFrame()

// 玩家查询
GetAllPlayers()
GetPlayerByAccountID(accountID)

// 实体操作
CreateFromPrefab(out var entity, prefabID)
Destroy(entity)
HasComponent(entity, componentType)
AddScript(entity, scriptID)

// 事件分发
DispatchEvent(eventID, targetEntity, paramsList)

// 传送
Teleport(player, position, rotation)

// 标签系统
AddTag(entity, tag)
RemoveTag(entity, tag)
HasTag(entity, tag)

// Map 内置操作
ContainKey(map, key)        // 检查键是否存在

// 格式化
Format(content, paramsList)  // 内置函数版本
```

### 内置事件（无需 Import）
```fc
// 生命周期事件（StdLibrary）
OnAwake()           // 实体创建时
OnStart()           // 游戏开始时（已废弃，不建议使用）
OnGameStart()       // 游戏开始时
OnUpdate()          // 每帧更新
OnGameEnd()         // 游戏结束时
OnDestroy()         // 实体销毁时
OnEnable()          // 实体启用时
OnDisable()         // 实体禁用时

// 生成事件（StdLibrary）
OnSpawn()           // 生成物品时
OnPlotStart()       // 剧情开始时
OnPlotEnd()         // 剧情结束时
OntriggerPlotOption() // 触发剧情选项时
```

---

## 编辑器生成库（EditorGenLib.fcc）

```fc
import "EditorGenLib.fcc" as gen
// 或
import Res from "EditorGenLib.fcc"

// 枚举资源
EResPrefab.SceneCube            // 预制体枚举
EResSceneIsleLand.WinnerView    // 场景物件枚举
EResUI.BombHUD                  // UI 资源枚举
EResSprite.ui_icon_skill        // 图标资源枚举

// 动态访问
Res.FCG["ScriptName"]           // 访问脚本资源
```

---

## 快速检查清单

每次编写代码时，按此顺序检查：

1. 识别函数类型
   - 这个函数是内置的吗？
   - 是 -> 无需 import
   - 否 -> 继续下一步

2. 查找函数所属库
   - List 操作 -> import "List.fcc" as list
   - Map 操作 -> import "Map.fcc" as map
   - 数学计算 -> import "Math.fcc" as math
   - 字符串 -> import "Strings.fcc" as strings
   - 物理检测 -> import "Physics.fcc" as physics
   - 战斗系统 -> import "Combat.fcc" as combat
   - 类型转换 -> import "Convert.fcc" as convert

3. 添加 import 语句
```fc
// 在文件开头添加
import "StdLibrary.fcc" as std
import "List.fcc" as list
import "Math.fcc" as math
```

4. 使用命名空间调用
```fc
// 推荐：使用命名空间前缀
list.Append(myList, value)
math.RandomInt(1, 10)
strings.Length(myString)

// 避免：直接调用（会导致编译错误）
Append(myList, value)      // 错误
RandomInt(1, 10)           // 错误
```

---

## 常见错误

### 错误示例 1：忘记 import
```fc
graph BadExample {
    event OnAwake() {
        var myList = New(0, 10)        // 错误：New 未定义
        Append(myList, 42)             // 错误：Append 未定义
    }
}
```

**修复：**
```fc
import "StdLibrary.fcc" as std
import "List.fcc" as list

graph GoodExample {
    event OnAwake() {
        var myList = list.New(0, 10)   // 正确
        list.Append(myList, 42)        // 正确
    }
}
```

### 错误示例 2：混淆内置和库函数
```fc
import "Math.fcc" as math

graph BadExample {
    event OnAwake() {
        LogInfo("游戏开始")            // 正确：内置函数
        var dist = Distance(posA, posB)  // 错误：应该用 math.Distance()
    }
}
```

**修复：**
```fc
import "StdLibrary.fcc" as std
import "Math.fcc" as math

graph GoodExample {
    event OnAwake() {
        LogInfo("游戏开始")                     // 正确：内置函数
        var dist = math.Distance(posA, posB)    // 正确：使用命名空间
    }
}
```

---

## 调试技巧

遇到"未定义符号"错误时：

1. 检查是否是内置函数
   - 查看本文档"内置函数"部分
   - 内置函数无需 import

2. 查询函数映射表
   - 在本文档中搜索函数名
   - 找到对应的库和用法

3. 检查命名空间
   - 确保使用了正确的命名空间前缀
   - 例如：list.Append() 而不是 Append()

4. 检查拼写
   - FC 语言区分大小写
   - 例如：ContainKey 而不是 ContainsKey

---

基于 Free Fire Craftland builtin 库生成
VS Code 插件：craftlandstudio.ffugclanguage

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
