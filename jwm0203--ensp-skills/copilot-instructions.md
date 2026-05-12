## ensp-skills

> 本项目用于对华为交换机/路由器进行自动化配置管理，主要应用于实验室及教学环境。通过 Python 脚本远程连接设备并批量下发配置命令，实现网络拓扑的快速部署、实验环境重置等任务。

# CLAUDE.md — 网络设备自动化配置项目

## 项目概述

本项目用于对华为交换机/路由器进行自动化配置管理，主要应用于实验室及教学环境。通过 Python 脚本远程连接设备并批量下发配置命令，实现网络拓扑的快速部署、实验环境重置等任务。

## 环境信息

- **Python 路径**: `C:/ProgramData/miniconda3/python.exe`（所有 Python 调用必须使用此路径，不要用 `python` 或 `python3`）
- **目标设备**: 华为交换机、路由器（VRP 系统）
- **连接方式**: SSH / Telnet
- **操作系统**: Windows 宿主机

## ⚠️ 网络设备配置核心规则（必须严格遵守）

对华为设备执行任何配置时，以下 4 条规则缺一不可。

---

### Rule 1：配置前必须进入 system-view

所有配置命令都需要在系统视图下执行。每次配置任务必须形成完整闭环：

```
system-view          ← 进入系统视图
<配置命令序列>        ← 执行配置
return               ← 退出到用户视图
save                 ← 保存配置
Y                    ← 确认保存
```

---

### Rule 2：处理确认提示（[Y/N]）

以下命令执行后会弹出 `[Y/N]` 确认提示，**必须发送 `Y`**：

- `undo ospf <id>`
- `undo interface`
- `undo vlan`
- `reset ospf process`
- **所有 `undo` 类命令**（通用规则）
- `save`

**不发送确认会导致设备阻塞，后续所有命令都会被当作确认输入吞掉。**

---

### Rule 3：每条命令后必须读取回显 ✅（最关键）

这是整个配置流程的核心保障。**严禁连续发送命令而不读取回显。**

```python
for cmd in commands:
    send(cmd)
    output = read()           # ✅ 必须读取回显
    if "[Y/N]" in output:     # 检测确认提示
        send("Y")
        output = read()       # 再次读取确认后的回显
    if "Error" in output:     # 检测错误
        log_error(cmd, output)
```

**绝对禁止：**
- ❌ 连续 send 多条命令不 read
- ❌ 假设命令成功跳过回显检查
- ❌ 忽略 `[Y/N]` 导致命令流错位

---

### Rule 4：错误检测与完整保存流程

回显中包含以下关键词时视为执行失败：

- `Error`
- `Unrecognized command`
- `Incomplete command`
- `%`（百分号开头的错误提示）

完整配置流程模板：

```python
def configure_device(connection, commands):
    errors = []

    # 1. 进入系统视图
    send("system-view")
    output = read()

    # 2. 逐条执行，每条都读回显
    for cmd in commands:
        send(cmd)
        output = read()
        if "[Y/N]" in output:
            send("Y")
            output = read()
        if any(kw in output for kw in ["Error", "Unrecognized", "Incomplete", "%"]):
            errors.append({"cmd": cmd, "output": output})

    # 3. 退出系统视图
    send("return")
    read()

    # 4. 保存配置
    send("save")
    output = read()
    if "[Y/N]" in output:
        send("Y")
        read()

    # 5. 汇报结果
    if errors:
        report_errors(errors)
    return errors
```

## 配置执行流程图

```
开始配置任务
  │
  ├─ 1. send("system-view") → read()
  │
  ├─ 2. for cmd in commands:
  │     ├─ send(cmd)
  │     ├─ output = read()            ← 必须
  │     ├─ "[Y/N]" → send("Y") + read()
  │     └─ "Error" → 记录错误
  │
  ├─ 3. send("return") → read()
  │
  ├─ 4. send("save") → read()
  │     └─ "[Y/N]" → send("Y") + read()
  │
  └─ 5. 返回执行结果
```

## 常见配置场景模板

### VLAN 配置

```python
commands = [
    "vlan batch 10 20 30",
    "interface GigabitEthernet0/0/1",
    "port link-type access",
    "port default vlan 10",
    "quit",
]
```

### OSPF 配置

```python
commands = [
    "ospf 1 router-id 1.1.1.1",
    "area 0",
    "network 192.168.1.0 0.0.0.255",
    "quit",
    "quit",
]
```

### 重置 OSPF（需要确认）

```python
# 注意：reset ospf process 会触发 [Y/N]
# Rule 3 的回显检测机制会自动处理
commands = ["reset ospf process"]
```

## Python 调用规范

```bash
# ✅ 正确
C:/ProgramData/miniconda3/python.exe script.py

# ❌ 错误
python script.py
python3 script.py
```

## 目录与文件规则

- **禁止修改根目录文件**：不得修改项目根目录下的任何已有文件（包括 `.md`、`.docx`、`.py` 等所有文件）。
- **脚本与检查文件放 `log/`**：所有新建的脚本工具（如 `.py`）及用于检查任务效果的文件（如 `check.md`、日志文件等）必须放在 `log/` 目录中。

## 教学/实验室场景注意事项


- 实验前建议用 `display current-configuration` 备份当前配置
- 实验结束后可能需要批量重置设备，`undo` 命令较多时务必注意 Rule 2 的确认处理
- 多台设备应逐台配置并汇总结果，不要并行操作
- 回显读取不完整时适当增加等待时间（教学环境设备性能可能较低）
- 设备 console 口连接时注意波特率等串口参数匹配

---
> Source: [JWM0203/ensp-skills](https://github.com/JWM0203/ensp-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
