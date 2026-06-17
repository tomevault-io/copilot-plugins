## fc-syntax

> FC 语言完整语法和 API 参考


# FC 语言完整编程参考

基于 Free Fire Craftland 官方文档: https://ffcraftland.garena.com/en/docs/api

---

## 核心语法差异（重要）

### 组件访问语法

**正确模式：**
```fc
thisEntity<Transform>.Position = Vector3{0, 1, 0}
player<Player>.Health = 100.0
enterEntity<Entity>.Name
```

**错误模式（禁止）：**
```fc
thisEntity.Transform.Position      // 错误：不能用点号
player.Health                      // 错误：缺少尖括号
entity<Transform>.Position         // 错误：entity是类型不是实例
```

**记忆口诀：** 实例<组件>.属性

**高频组件：**
```fc
thisEntity<Transform>.Position    // Vector3
thisEntity<Transform>.Rotation    // Quaternion
player<Player>.Health             // float
player<Player>.Name               // string
thisEntity<Entity>.Active         // bool
```

**类型转换示例：**
```fc
event OnEntityEnter(enterEntity entity<Entity>) {
    if HasComponent(enterEntity, typeof(Player)) {
        var player = enterEntity<Player>
        var playerName = player<Player>.Name
        LogInfo("玩家进入: " + playerName)
    }
}
```

---

## 目录

1. 核心概念
2. 语法参考
3. 类型系统
4. 内置库API参考
5. 组件系统
6. 事件系统
7. 编程模式

---

## 核心概念

### FC 语言设计理念
FC（Free Fire Code Language）是面向数据设计的 DSL，用于 Free Fire UGC 游戏开发。

特点：
- 面向数据设计（而非面向对象）
- 实体-组件架构（Entity-Component）
- 强类型系统
- 异步编程支持
- 跨平台兼容

### 文件类型

| 扩展名 | 类型 | 用途 |
|--------|------|------|
| .fcc | 库定义文件 | 定义组件、类型、API、事件、枚举 |
| .fcg | 脚本逻辑文件 | 编写 graph、变量、函数、事件处理 |

### 平台声明

```fc
// 客户端脚本
[platform_client]
graph ClientScript { }

// 服务端脚本（默认）
[platform_server]
graph ServerScript { }
```

---

## 语法参考

### 导入系统

```fc
// 导入内置库
import "StdLibrary.fcc" as std
import "List.fcc" as list
import "Math.fcc" as math

// 导入编辑器生成库
import "EditorGenLib.fcc" as gen

// 导入用户自定义库
import "MyCustomLib.fcc" as MyLib
```

### 变量声明

```fc
// 局部变量
var num int = 10
var name = "Player"     // 类型推断
var pos Vector3         // 未初始化

// 脚本成员变量
graph MyGraph {
    PlayerCount int = 0
    GameStarted bool = false
}
```

### 函数定义

```fc
// 普通函数
func Add(a int, b int) int {
    return a + b
}

// 异步函数
async func DelayedAction(delay int, out var result string) {
    WaitForMillisecond(delay)
    result = "完成"
}

// 无返回值函数
func LogMessage(msg string) {
    LogInfo(msg)
}
```

### 流程控制

```fc
// 条件语句
if health > 50 {
    LogInfo("健康")
} else if health > 20 {
    LogInfo("一般")
} else {
    LogInfo("危险")
}

// for 循环（左闭右开区间）
for i = 0, 10, 1 {
    LogInfo("计数: " + i)
}

// for range 循环
for index, value in myList {
    LogInfo("索引: " + index + ", 值: " + value)
}

// while 循环
while playerCount < maxPlayers {
    SpawnPlayer()
    playerCount = playerCount + 1
}
```

### 异步编程

```fc
graph AsyncExample {
    event OnGameStart() {
        // 等待异步完成
        wait DelayedSpawn(3000, out var success)
        
        // 异步执行，不等待
        start BackgroundTask()
    }
    
    async func DelayedSpawn(delay int, out var success bool) {
        WaitForMillisecond(delay)
        success = true
    }
}
```

---

## 类型系统

### 基本类型

| 类型 | 描述 | 示例 |
|------|------|------|
| bool | 布尔类型 | true, false |
| int | 32位整型 | 42, -10 |
| int64 | 64位整型 | 9223372036854775807 |
| float | 单精度浮点 | 3.14, -0.5 |
| string | 字符串 | "Hello World" |
| LocString | 本地化字符串 | 多语言文本 |

### 数学类型

```fc
// 向量类型
var pos2D Vector2 = Vector2{1.0, 2.0}
var pos3D Vector3 = Vector3{1.0, 2.0, 3.0}

// 四元数（旋转）
var rotation Quaternion = Quaternion{0.0, 0.0, 0.0, 1.0}

// 访问分量
var x = pos3D.X
var y = pos3D.Y
var z = pos3D.Z
```

### 容器类型

```fc
// 列表
var numbers List<int> = List<int>{}
numbers.Add(1)
numbers.Add(2)

// 映射
var playerScores Map<string, int> = Map<string, int>{}
playerScores["Player1"] = 100
playerScores["Player2"] = 85
```

### 实体类型

```fc
// 泛型实体类型
var player entity<Player>
var box entity<LevelObject>

// 访问组件
var playerName = player<Entity>.Name
var boxPosition = box<Transform>.Position

// 当前实体
var currentPos = thisEntity<Transform>.Position
```

---

## 内置库 API 参考

### 核心库 (StdLibrary)

```fc
// 日志函数
LogInfo(content object)
LogWarning(content object)
LogError(content object)

// 时间函数
WaitForMillisecond(time int)
WaitForSecond(time float)
WaitForFrame(frameCount int)

// 时间获取
GetCurrentTimestamp() int64
GetGameTime() float
```

### 数学库 (Math)

```fc
import "Math.fcc" as math

// 随机数
math.RandomInt(min, max) int
math.RandomFloat(min, max) float

// 基础数学
math.Abs(value) Number
math.Sqrt(value) float
math.Max(a, b) Number
math.Min(a, b) Number
math.Clamp(value, min, max) Number

// 三角函数
math.Sin(degree) float
math.Cos(degree) float
math.Tan(degree) float

// Vector3 运算
math.Dot(vecA, vecB) float
math.Cross(vecA, vecB) Vector3
math.Normalize(vec) Vector3
math.Distance(vecA, vecB) float
math.LerpVector3(a, b, t) Vector3

// 四元数运算
math.QuaternionToEulerAngle(quat) Vector3
math.EulerAngleToQuaternion(euler) Quaternion
math.LookRotation(forward, upward) Quaternion
```

### 物理库 (Physics)

```fc
import "Physics.fcc" as physics

// 射线检测
physics.SingleRaycast(startPos, direction, distance, layerMask, includeTrigger, out hitEntity, out hitPoint, out hitDistance, out hitNormal)

// 区域检测
physics.BoxOverlap(halfExtents, center, rotation, layerMask, includeTrigger, out hitEntities)
physics.SphereOverlap(radius, center, layerMask, includeTrigger, out hitEntities)

// 区域判断
physics.IsInsideSphere(entity, center, radius) bool
physics.IsInsideBox(entity, center, length, width, height, rotation) bool

// 刚体力学
physics.AddForce(target, force, forceMode)
physics.AddTorque(target, torque, forceMode)
```

### 玩家库 (Player)

```fc
// 玩家操作
KillPlayer(player entity<Player>)
RevivePlayer(player entity<Player>)
TeleportPlayer(player entity<Player>, position Vector3)
SetPlayerHealth(player entity<Player>, health float)

// 玩家查询
GetAllPlayers() List<entity<Player>>
GetPlayerByIndex(index int) entity<Player>
GetLocalPlayer() entity<Player>
```

### 列表库 (List)

```fc
import "List.fcc" as list

list.New(length, capacity) List<object>
list.Append(target, value)
list.Length(target) int
list.RemoveAt(target, index)
list.Clear(target)
list.Contains(target, value) bool
```

### 映射库 (Map)

```fc
import "Map.fcc" as map

map.New() Map<object, object>
map.Remove(target, key)
map.ContainKey(target, key) bool
map.GetAllKeys(target) List<object>
```

---

## 组件系统

### 常用组件

```fc
// Transform 组件
component Transform {
    Position Vector3
    Rotation Quaternion
    Scale Vector3
}

// Player 组件
component Player {
    AccountID UUID
    NickName string
    TeamID int
    Health float
    MaxHealth float
    Speed float
}

// Entity 组件（所有实体都有）
component Entity {
    Name string
    Active bool
    Tag string
}

// Rigidbody 组件
component Rigidbody {
    Type RigidbodyType
    UseGravity bool
    Mass float
    Velocity Vector3
}
```

### 组件操作

```fc
// 检查组件
HasComponent(entity, typeof(Player)) bool

// 添加脚本
AddScript(entity, scriptID)

// 标签系统
AddTag(entity, "KeyPlayer")
RemoveTag(entity, "KeyPlayer")
HasTag(entity, "KeyPlayer") bool
```

---

## 事件系统

### 生命周期事件

```fc
event OnAwake()          // 实体创建时
event OnStart()          // 游戏开始时
event OnDestroy()        // 实体销毁时
event OnUpdate()         // 每帧更新
```

### 游戏事件

```fc
event OnGameStart()
event OnGameEnd()
event OnPlayerJoin(player entity<Player>)
event OnPlayerLeave(player entity<Player>)
```

### 物理事件

```fc
event OnTriggerEnter(other entity<Entity>)
event OnTriggerExit(other entity<Entity>)
event OnCollisionEnter(other entity<Entity>)
event OnCollisionExit(other entity<Entity>)
```

### 战斗事件

```fc
event OnEliminated(attacker entity<Entity>, damageType DamageType, hitBodyPart DamagedBodyPartType)
event OnTakeDamage(attacker entity<Entity>, damageType DamageType, damage int, hitBodyPart DamagedBodyPartType)
```

### 事件分发

```fc
// 分发事件
DispatchEvent(eventID, targetEntity, List<object>{param1, param2})

// 示例
DispatchEvent(OnMyGameStart, globalEntity, List<int>{1})
```

---

## 编程模式

### 状态机模式

```fc
enum GameState {
    MainMenu = 0
    Playing = 1
    Paused = 2
}

graph GameStateManager {
    CurrentState GameState = GameState.MainMenu
    
    func ChangeState(newState GameState) {
        var oldState = CurrentState
        CurrentState = newState
        OnStateExit(oldState)
        OnStateEnter(newState)
    }
}
```

### 对象池模式

```fc
graph BulletPool {
    PoolSize int = 100
    AvailableBullets List<entity<LevelObject>>
    
    func GetBullet() entity<LevelObject> {
        if AvailableBullets.Count > 0 {
            var bullet = AvailableBullets[0]
            AvailableBullets.RemoveAt(0)
            SetObjectActive(bullet, true)
            return bullet
        }
        return null
    }
}
```

---

## 最佳实践

### 代码组织

```fc
// 文件头部：import 语句
import "StdLibrary.fcc" as std
import "List.fcc" as list
import "Math.fcc" as math

// graph 定义
graph MyScript {
    // 成员变量
    PlayerCount int = 0
    
    // 事件处理
    event OnAwake() {
        InitializeGame()
    }
    
    // 函数定义
    func InitializeGame() {
        PlayerCount = GetAllPlayers().Count
    }
}
```

### 性能优化

```fc
// 避免频繁内存分配
graph ObjectManager {
    NearbyObjectsCache List<entity<LevelObject>>
    
    event OnUpdate() {
        NearbyObjectsCache.Clear()  // 复用列表
    }
}

// 异步批处理
async func SpawnMultipleEnemiesAsync() {
    var batchSize = 5
    for batch = 0, 20, 1 {
        for i = 0, batchSize, 1 {
            CreateEnemy()
        }
        WaitForMillisecond(50)
    }
}
```

### 错误处理

```fc
func GetPlayerSafely(index int) entity<Player> {
    var allPlayers = GetAllPlayers()
    if index < 0 || index >= allPlayers.Count {
        LogWarning("无效的玩家索引: " + index)
        return null
    }
    return allPlayers[index]
}
```

---

参考资源：
- 官方文档：https://ffcraftland.garena.com/en/docs/
- VS Code 插件：craftlandstudio.ffugclanguage (v1.6.0)
- Import 映射表：@fc-import-map.mdc
- 代码示例：@fc-examples.mdc

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
