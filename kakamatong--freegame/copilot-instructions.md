## freegame

> 本文档包含在此基于Skynet的游戏服务器项目上工作的智能编程代理的重要信息。

# AGENTS.md - 游戏服务器开发指南

本文档包含在此基于Skynet的游戏服务器项目上工作的智能编程代理的重要信息。

## 项目概述

这是一个基于Skynet框架使用Lua构建的多玩家游戏服务器。它采用集群架构，为不同的游戏功能（登录、游戏、匹配、用户等）提供独立的服务，并支持与客户端的WebSocket通信。

## 构建和开发命令

### 构建命令
```bash
# 构建Skynet（包括cjson库等自定义修改）
cd skynet && make

# 清理构建产物
cd skynet && make clean

# 完全清理包括依赖项
cd skynet && make cleanall
```

### 运行命令
```bash
# 启动所有游戏服务
./run.sh

# 停止所有服务
./stop.sh

# 构建生产部署包
./output.sh
```

### 测试命令
```bash
# 运行特定测试文件（示例）
lua test_vote_disband.lua

# 运行Skynet内置测试
cd skynet && lua test/testtimer.lua
cd skynet && lua test/testecho.lua
```

### 单独服务命令
```bash
# 启动单独的服务（用于调试）
./skynet/skynet config/configLogin      # 登录服务
./skynet/skynet config/configGame       # 游戏服务
./skynet/skynet config/configGate       # 网关服务
./skynet/skynet config/configMatch      # 匹配服务
./skynet/skynet config/configUser       # 用户服务
./skynet/skynet config/configDB         # 数据库服务
```

### 协议说明
- **sproto协议**: 存放在 `proto/` 目录，无需编译，运行时自动加载
- 修改 sproto 文件后重启服务即可生效，无需重新构建Skynet

## 代码风格指南

### 文件结构和组织
- **服务**: `src/services/` - 单独的Skynet服务
- **游戏**: `src/services/games/` - 特定游戏逻辑（如10001/、10002/）
- **配置**: `src/config/` - 配置文件
- **库**: `src/lualib/` - 共享工具库
- **协议**: `proto/` - 协议定义（sproto格式）

### 命名约定
- **文件**: snake_case（如 `private_room.lua`, `game_config.lua`）
- **变量**: 本地变量使用snake_case，常量使用UPPER_SNAKE_CASE
- **函数**: 函数使用snake_case，类/对象构造函数使用PascalCase
- **服务**: 使用描述性名称并以服务类型结尾（如 `match_server.lua`, `user_server.lua`）
- **配置键**: UPPER_SNAKE_CASE（如 `USER_STATUS`, `SVR_NAME`）

### Lua代码风格

#### 导入和Require
```lua
-- 标准顺序：skynet模块优先，然后是第三方库，最后是本地模块
local skynet = require "skynet"
local cluster = require "skynet.cluster"
local sprotoloader = require "sprotoloader"
local log = require "log"
local cjson = require "cjson"
local gameConfig = require "gameConfig"
```

#### 函数和方法
```lua
-- 普通函数
local function calculateScore(player_data)
    return player_data.level * 100 + player_data.exp
end

-- 对象方法
function Player:updateScore(new_score)
    self.score = new_score
    self:saveToDatabase()
end

-- 服务命令
function CMD.getuserData(userid)
    return call(svrUser, "userData", userid)
end
```

#### 表和对象
```lua
-- 配置表
local config = {
    GAME_STATUS = {
        WAITING = 0,
        START = 1,
        END = 2
    },
    DEFAULT_TIMEOUT = 30
}

-- 对象创建
local Player = {}
Player.__index = Player

function Player:new(userid)
    local obj = {
        userid = userid,
        score = 0,
        status = "offline"
    }
    setmetatable(obj, self)
    return obj
end
```

### 错误处理
```lua
-- 使用pcall进行安全操作
local ok, result = pcall(some_function, arg1, arg2)
if not ok then
    log.error("Function failed: %s", result)
    return {code = 0, msg = "Operation failed"}
end

-- 始终验证输入
if not userid or userid <= 0 then
    return {code = 0, msg = "Invalid userid"}
end

-- 服务响应应遵循一致的格式
return {code = 1, msg = "Success", data = result}
return {code = 0, msg = "Error description"}
```

### 日志指南
```lua
-- 使用适当的日志级别
log.debug("Detailed debug info: %s", details)
log.info("User %d joined game %d", userid, gameid)
log.warn("Slow operation detected: %d ms", duration)
log.error("Database connection failed: %s", error_msg)
log.fatal("Critical system error, shutting down")

-- 在日志中包含上下文
log.info("Room %d: Player %d performed action %s", roomid, userid, action)
```

### 服务通信模式
```lua
-- 通过全局助手进行集群调用
local result = call(svrUser, "getUserData", userid)
call(svrGame, "updatePlayerState", userid, new_state)

-- 直接服务调用
local db_service = skynet.localname(CONFIG.SVR_NAME.DB)
local data = skynet.call(db_service, "lua", "query", sql)

-- 发后即忘的消息
send(svrMatch, "playerDisconnected", userid)
```

### 配置管理
- 所有配置都在`src/config/gameConfig.lua`中
- 使用分组的配置表（如`mysql`、`redis`、`USER_STATUS`）
- 环境特定配置应具有后缀（`.prod`、`.dev`）
- 常量应具有描述性且自文档化

### 协议处理
```lua
-- 客户端请求处理（在agent.lua中）
function REQUEST:getGameData(args)
    -- 验证参数
    if not args.gameid then
        return {code = 0, msg = "Missing gameid"}
    end
    
    -- 处理请求
    local game_data = call(svrGame, "getGameData", args.gameid)
    
    return {code = 1, data = game_data}
end

-- 注册客户端协议
skynet.register_protocol {
    name = "client",
    id = skynet.PTYPE_CLIENT,
    unpack = function(msg, sz)
        local str = skynet.tostring(msg, sz)
        return host:dispatch(str, sz)
    end,
    dispatch = function(fd, _, type, ...)
        -- 处理客户端消息
    end
}
```

### 测试指南
- 测试文件应该是独立且可执行的
- 使用描述性的测试函数名
- 包含断言来验证预期行为
- 测试成功和错误场景
- 示例测试结构可以在`test_vote_disband.lua`中找到

### 性能考虑
- 使用`STAT`工具进行性能监控
- 为外部调用实现适当的超时处理
- 为数据库操作使用连接池
- 监控长时间运行服务的内存使用

### 数据库模式
```lua
-- 通过db服务进行Redis操作
call(svrDB, "dbRedis", "set", key, value, expire_time)
local result = call(svrDB, "dbRedis", "get", key)

-- MySQL操作
local sql = "SELECT * FROM users WHERE userid = ?"
local results = call(svrDB, "dbMySQL", "query", sql, userid)
```

### 重要注意事项
- 所有服务都在Skynet actor模型中运行
- 使用`skynet.timeout()`进行延迟操作，不使用`sleep()`
- 从不在服务处理程序中使用阻塞操作
- 始终优雅地处理断开连接
- 使用全局助手（`call`、`send`、`callTo`、`sendTo`）进行集群通信
- 配置文件使用环境特定的覆盖（prod/dev）

### 语言要求
- **回复语言**: 无论用户使用何种语言提问，始终使用中文回复用户

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kakamatong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kakamatong)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
