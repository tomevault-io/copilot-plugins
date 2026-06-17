## fc-examples

> FC 语言代码示例库


# FC 语言开发示例

本文档包含 FC 语言的完整开发示例，每个示例都可以独立使用。

---

## 新手避坑指南

### 示例 1：列表操作

```fc
// 完整正确示例
import "StdLibrary.fcc" as std
import "List.fcc" as list              // 必须 import

graph ListExample {
    MyList List<int>
    
    event OnAwake() {
        MyList = list.New(0, 10)       // 使用命名空间前缀
        list.Append(MyList, 42)
        list.Append(MyList, 100)
        
        var length = list.Length(MyList)
        LogInfo("列表长度: " + length)
        
        for index, value in MyList {
            LogInfo("索引 " + index + " 的值: " + value)
        }
    }
}
```

**常见错误：**
```fc
// 错误：缺少 import "List.fcc"
var myList = New(0, 10)
Append(myList, 42)  // 错误：缺少命名空间前缀
```

---

### 示例 2：组件访问

```fc
// 正确的组件访问
import "StdLibrary.fcc" as std

graph PlayerHandler {
    event OnAwake() {
        // 正确：实例<组件>.属性
        thisEntity<Player>.Health = 100.0
        thisEntity<Player>.RunSpeedScale = 1.5
        thisEntity<Transform>.Position = Vector3{0, 1, 0}
        
        var playerName = thisEntity<Entity>.Name
        LogInfo("玩家名称: " + playerName)
    }
}
```

**常见错误：**
```fc
// 错误 1：使用点号访问组件
thisEntity.Player.Health = 100.0

// 错误 2：缺少组件类型
thisEntity.Health = 100.0

// 错误 3：属性后加尖括号
thisEntity<Player>.Health<float> = 100.0
```

---

### 示例 3：遍历玩家

```fc
import "StdLibrary.fcc" as std

graph PlayerManager {
    event OnAwake() {
        // GetAllPlayers() 是内置函数，无需 import
        for index, player in GetAllPlayers() {
            // player 已经是实例，直接用尖括号访问组件
            player<Player>.RunSpeedScale = 1.5
            player<Player>.Health = 100.0
            player<Transform>.Position = Vector3{index, 1, 0}
            
            var name = player<Entity>.Name
            LogInfo("玩家 " + index + ": " + name)
        }
    }
}
```

**常见错误：**
```fc
for index, player in GetAllPlayers() {
    player.Health = 100.0                     // 错误：使用点号
    player.Transform.Position = Vector3{0, 0, 0}  // 错误
}
```

---

### 示例 4：数学运算

```fc
import "StdLibrary.fcc" as std
import "Math.fcc" as math              // 必须 import

graph MathExample {
    event OnAwake() {
        // 使用命名空间前缀
        var randomNum = math.RandomInt(1, 100)
        var distance = math.Distance(Vector3{0,0,0}, Vector3{10,0,0})
        var normalized = math.Normalize(Vector3{1, 2, 3})
        
        LogInfo("随机数: " + randomNum)
        LogInfo("距离: " + distance)
    }
}
```

**常见错误：**
```fc
var randomNum = RandomInt(1, 100)     // 错误：缺少 import 和命名空间
var distance = Distance(pos1, pos2)   // 错误
```

---

### 示例 5：触发器事件

```fc
import "StdLibrary.fcc" as std

graph TriggerHandler {
    event OnEntityEnter(enterEntity entity<Entity>) {
        // 先检查组件是否存在
        if HasComponent(enterEntity, typeof(Player)) {
            // 转换为 Player 类型
            var player = enterEntity<Player>
            
            // 访问组件
            var playerName = player<Player>.Name
            player<Player>.Health = 150.0
            var position = player<Transform>.Position
            
            LogInfo("玩家进入触发器: " + playerName)
        }
    }
}
```

**常见错误：**
```fc
event OnEntityEnter(enterEntity entity<Entity>) {
    var player = enterEntity<Player>        // 错误：未检查就转换
    var name = enterEntity.Name             // 错误：使用点号
}
```

---

### 示例 6：Map 操作

```fc
import "StdLibrary.fcc" as std
import "Map.fcc" as map                // 必须 import

graph MapExample {
    PlayerScores Map<string, int>
    
    event OnAwake() {
        PlayerScores = map.New()
        
        PlayerScores["Alice"] = 100
        PlayerScores["Bob"] = 85
        
        // ContainKey 是内置函数
        if ContainKey(PlayerScores, "Alice") {
            var score = PlayerScores["Alice"]
            LogInfo("Alice的分数: " + score)
        }
        
        // 获取所有键
        var allKeys = map.GetAllKeys(PlayerScores)
        for index, key in allKeys {
            LogInfo("玩家: " + key + ", 分数: " + PlayerScores[key as string])
        }
    }
}
```

---

### 示例 7：物理检测

```fc
import "StdLibrary.fcc" as std
import "Physics.fcc" as physics        // 必须 import

graph PhysicsExample {
    event OnAwake() {
        var center = Vector3{0, 0, 0}
        var radius = 10.0
        
        for index, player in GetAllPlayers() {
            if physics.IsInsideSphere(player, center, radius) {
                var playerName = player<Player>.Name
                LogInfo(playerName + " 在范围内")
            }
        }
        
        // 射线检测
        physics.SingleRaycast(
            Vector3{0, 1, 0},
            Vector3{0, 0, 1},
            100.0,
            List<int>{},
            false,
            out var hitEntity,
            out var hitPoint,
            out var hitDistance,
            out var hitNormal
        )
        
        if hitEntity != nil {
            LogInfo("射线击中了: " + hitEntity<Entity>.Name)
        }
    }
}
```

---

## 基础示例

### 图形定义

```fc
import "StdLibrary.fcc" as std

// 普通图形
graph MyScript {
    PlayerCount int = 0
    
    event OnAwake() {
        LogInfo("Script initialized")
    }
}

// 静态图形（全局）
static graph GlobalManager {
    MaxPlayers int = 10
}
```

---

### 实体和组件

```fc
import "StdLibrary.fcc" as std

// 访问当前实体组件
graph EntityHandler {
    event OnAwake() {
        thisEntity<Transform>.Position = Vector3{0, 1, 0}
        thisEntity<Transform>.Rotation = Quaternion{0, 0, 0, 1}
        thisEntity<BasicAppearance>.DiffuseColor = #FF0000FF
    }
}

// 检查组件
graph ComponentChecker {
    event OnAwake() {
        if HasComponent(thisEntity, typeof(Player)) {
            thisEntity<Player>.RunSpeedScale = 1.5
            LogInfo("Player component found")
        }
    }
}
```

---

### 预制体操作

```fc
import "StdLibrary.fcc" as std

graph PrefabCreator {
    event OnAwake() {
        // 创建场景方块
        CreateFromPrefab(out var cube, EResPrefab.SceneCube)
        cube<Transform>.Position = Vector3{5, 0, 5}
        
        // 创建炸弹
        CreateFromPrefab(out var bomb, EResPrefab.SceneBomb)
        bomb<Transform>.Position = thisEntity<Transform>.Position
    }
}
```

---

### UI 操作

```fc
import "StdLibrary.fcc" as std
import "Hud.fcc" as hud

graph UICreator {
    MainUI entity<CustomHud>
    
    event OnAwake() {
        CreateCustomUI(out var ui, thisEntity<Player>, EResUI.BombHUD)
        MainUI = ui
        MainUI<CustomUI>.IsVisible = false
    }
    
    event OnMyGameStart(level int) {
        MainUI<CustomUI>.IsVisible = true
    }
}

// 按钮点击事件
graph ButtonHandler {
    ClickCount int = 0
    
    event OnTapped(player entity<Player>) {
        ClickCount += 1
        LogInfo("Button clicked by: " + player<Player>.Name)
    }
}
```

---

### 异步编程

```fc
import "StdLibrary.fcc" as std

graph AsyncHandler {
    event OnAwake() {
        // 等待异步完成
        wait DelayedAction()
        
        // 异步执行，不等待
        start BackgroundTask()
    }
    
    async func DelayedAction() {
        WaitForMillisecond(2000)
        LogInfo("Delayed action completed")
    }
    
    async func BackgroundTask() {
        WaitForNextFrame()
        LogInfo("Background task running")
    }
}

// 带输出参数
graph AsyncCalculator {
    event OnAwake() {
        wait CalculateSum(10, 20, out var result)
        LogInfo("Result: " + result)
    }
    
    async func CalculateSum(a int, b int, out var sum int) {
        WaitForMillisecond(1000)
        sum = a + b
    }
}
```

---

### 标签系统

```fc
import "StdLibrary.fcc" as std

graph TagManager {
    event OnAwake() {
        AddTag(thisEntity, "KeyPlayer")
        AddTag(thisEntity, "SpecialUnit")
        
        if HasTag(thisEntity, "KeyPlayer") {
            LogInfo("This entity has the key")
        }
        
        RemoveTag(thisEntity, "SpecialUnit")
    }
}
```

---

### 事件分发

```fc
import "StdLibrary.fcc" as std
import "EditorGenLib.fcc" as gen

graph EventDispatcher {
    event OnAwake() {
        // 分发给全局实体
        DispatchEvent(OnMyGameStart, globalEntity, List<object>{1})
        
        // 分发给特定实体
        DispatchEvent(gen.OnSceneCubeCreated, thisEntity, nil)
    }
}

// 分发给所有玩家
graph PlayerEventDispatcher {
    func NotifyAllPlayers(message string) {
        for index, player in GetAllPlayers() {
            DispatchEvent(OnPlayerMessage, player, List<object>{message})
        }
    }
}
```

---

### 战斗系统

```fc
import "StdLibrary.fcc" as std
import "Combat.fcc" as combat

graph CombatHandler {
    event OnEliminated(attackerEntity entity<Entity>, attackedType DamageType, hitBodyPart HitBodyPart) {
        LogInfo("Player eliminated")
        
        if HasTag(thisEntity, "KeyPlayer") {
            RemoveTag(thisEntity, "KeyPlayer")
            CreateFromPrefab(out var keyTrigger, EResPrefab.KeyTrigger)
            keyTrigger<Transform>.Position = thisEntity<Player>.Position
        }
    }
}

// 消除和复活
graph PlayerLifecycle {
    async func EliminateAndRevive(player entity<Player>) {
        Eliminate(player)
        WaitForMillisecond(3000)
        Revive(player)
    }
}
```

---

### 传送和移动

```fc
import "StdLibrary.fcc" as std

graph TeleportManager {
    func TeleportToSpawn(player entity<Player>) {
        var spawnPoint = Vector3{0, 1, 0}
        var spawnRotation = Quaternion{0, 0, 0, 1}
        Teleport(player, spawnPoint, spawnRotation)
    }
    
    func TeleportToWinArea(player entity<Player>) {
        var winnerView = EResSceneIsleLand.WinnerView as entity<LevelObject>
        Teleport(player, winnerView<Transform>.Position, winnerView<Transform>.Rotation)
    }
}
```

---

### 游戏流程

```fc
import "StdLibrary.fcc" as std

graph GamePhaseManager {
    event OnMyGameStart(level int) {
        LogInfo("Game started - Level: " + level)
        SetAllPlayersMoveStatus(false)
        wait InitializeLevel(level)
        SetAllPlayersMoveStatus(true)
    }
    
    async func InitializeLevel(level int) {
        WaitForMillisecond(1000)
        LogInfo("Level initialized")
    }
    
    event OnMyGameEnd() {
        LogInfo("Game ended")
        WaitForMillisecond(3000)
        EndCurrentPhase()
    }
}

// 倒计时
graph CountdownManager {
    async func StartCountdown(seconds int) {
        for i = seconds, 0, -1 {
            var message = Format("Game starts in: %v", List<int>{i})
            
            for index, player in GetAllPlayers() {
                NotifyShowTips(player, message, #FFFFFFFF, 1000)
            }
            
            WaitForMillisecond(1000)
        }
        
        DispatchEvent(OnMyGameStart, globalEntity, List<int>{1})
    }
}
```

---

### 资源清理

```fc
import "StdLibrary.fcc" as std

graph ResourceCleaner {
    MyUI entity<CustomHud>
    
    event OnDestroy() {
        if MyUI != nil {
            Destroy(MyUI)
            MyUI = nil
        }
        LogInfo("Resources cleaned up")
    }
    
    async func DelayedDestroy(target entity, delay int) {
        WaitForMillisecond(delay)
        Destroy(target)
    }
}
```

---

参考资源：
- Import 映射表：@fc-import-map.mdc
- 完整语法：@fc-syntax.mdc
- 官方文档：https://ffcraftland.garena.com/en/docs/

---
> Source: [WindGod-Project-For-Garena/Climbing-World](https://github.com/WindGod-Project-For-Garena/Climbing-World) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
