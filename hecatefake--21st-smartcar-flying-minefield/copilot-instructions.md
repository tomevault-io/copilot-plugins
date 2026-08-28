## 21st-smartcar-flying-minefield

> > 本文件是本仓唯一项目级 Agent 行为权威。用户名是 Hecate，始终使用中文回复，不使用 emoji。

# Agent 工作区契约

> 本文件是本仓唯一项目级 Agent 行为权威。用户名是 Hecate，始终使用中文回复，不使用 emoji。

## 1. 冷启动

开始任何任务时按顺序读取：

1. 本文件。
2. [规则路由器](.claude/rules/00-router.md)，按任务命中项读取规则与 lesson。
3. 目标核的 `project/code/PROJECT.md`；需要运行细节时再读同目录 `ARCHITECTURE.md`，涉及人机交互时读 `INTERACTION.md`。
4. 跨核事实从 [docs/README.md](docs/README.md) 进入，不从其它核 PROJECT 拼接。
5. HecateFlow 任务读取 [manifest](.hecateflow/project.json) 并按生命周期 skill 路由。

零上下文仍无法确定目标核时，先搜索路径与公开符号；只有仓库无法裁决且选择会改变实现时才询问用户。

## 2. 授权与Git

- 默认只读。查看、审查、调研、评估、计划、分析和解释不授权写文件。
- 只有用户明确要求修改、实现、落地或开始实施时，才可写入其范围内文件。
- `manifest`、注入配置和Git分别需要明确授权；普通源码授权不自动包含它们。
- 未经单独确认不得stage、commit、push、创建或合并PR。
- 禁止`git add .`和`git add -A`；只能显式暂存本批文件。
- 不得回退、覆盖或删除用户及其它Agent的改动。非本任务改动按路径归因并保持原状。
- 简单更改默认直接在当前本地`main`工作树实施，不自动fetch、切分支、创建worktree或PR；开始编辑前仍须确认分支和dirty状态，并保护不相关改动。
- 多阶段、范围庞大、预计长耗时反复迭代、需要并行写入，或跨模块/跨核且隔离能实质降低污染风险时，主Agent先说明原因、基线、拟建分支与目录并征得用户明确同意；同意前不得创建分支/worktree，同意后将该任务的读取、编辑、测试与验证工作目录迁移到新worktree。用户直接明确命令创建worktree视为本次同意，仅讨论是否使用不算授权。
- 在本地`main`实施仍不授权Git写；用户确认发布`main`后才同时推送并核对`origin/main`和`gitee/main`。worktree功能分支是否推送、创建PR或合并均分别确认，不再默认创建PR。
- 提交说明格式：`AIG_<scope>_<中文描述>`。
- 子代理和外部模型永远不拥有Git；最终整合、验证与Git只由主Agent执行。

完整流程见[git-workflow](.claude/rules/git-workflow.md)和[agent-orchestration](.claude/rules/agent-orchestration.md)。

### 2.1 临场盲盒模式

- 用户明确说出`进入临场盲盒模式`后，进入当前对话内持续生效的五分钟快速通道；用户说出`退出临场盲盒模式`后恢复常规流程。
- 激活后，每条具体编码任务自动授权其直接相关业务源码写入，但不授权Git、manifest、规则、SDK/startup/system或链接脚本。
- 该模式跳过计划文件、多路复审、常规子代理、完整测试矩阵和长期文档同步；默认主Agent单写，仅在30秒内可拆成完全互斥文件且确有净收益时启用最多一个写入子代理。
- 盲盒参数只做编译期硬编码，不接菜单或Flash持久化；改动后保留45秒微验证，并明确报告需要烧录的目标板卡。
- ISR、执行器、共享内存、协议或Flash任务仍可实施，但强制主Agent单写，且不得绕过用户资产保护、嵌入式安全红线、证据边界和Git独立授权门。
- 临场盲盒模式不因任务复杂度自动提议、创建或迁移worktree，默认继续使用激活时的当前工作树；只有用户明确命令创建worktree时才执行，且创建耗时计入五分钟预算。

完整契约见[blind-box-mode](.claude/rules/blind-box-mode.md)。

## 3. 目标识别

| 目标 | 职责 | 项目入口 |
|---|---|---|
| Core0 | 无人机主控：cm7_0飞控/传感器，cm7_1追车视觉/KVM根菜单 | [PROJECT](core0_cyt4bb7/project/code/PROJECT.md) |
| Core1 | 无人机副控：cm7_0跨板通信，cm7_1信标视觉/规划/三摄协调 | [PROJECT](core1_cyt4bb7/project/code/PROJECT.md) |
| Core2 | 麦轮车：cm7_0底盘控制，cm7_1 RS-485/LoRa通信 | [PROJECT](core2_cyt4bb7/project/code/PROJECT.md) |
| Core3 | CYT4BB7双核后向观察板：cm7_0专线/跨核桥，cm7_1视觉/交互 | [PROJECT](core3_cyt4bb7/project/code/PROJECT.md) |
| Core4 | CYT4BB7双核前向观察板：cm7_0专线/跨核桥，cm7_1视觉/交互 | [PROJECT](core4_cyt4bb7/project/code/PROJECT.md) |
| CoreT3 | 独立录制调试板：MT9V03X经WiFi SPI上传PC | [PROJECT](coreT3_cyt4bb7/project/code/PROJECT.md) |

关键词路由：

- 飞控、姿态、电机、ToF、光流、追车定位 → Core0。
- 信标、路径规划、三来源连续性、RS-485主站、Core3/Core4 Observer专线 → Core1。
- 麦轮、编码器、电流/速度环、LoRa、底盘模式 → Core2。
- 后向相机、Core3 KVM/屏幕/RGB、Core3专线 → Core3；前向相机、Core4 KVM/屏幕/RGB、Core4专线 → Core4；归档CYT2BL3只从`_legacy/core3_cyt2bl3`查阅。
- 原始图像录制、WiFi SPI、逐飞助手TCP → CoreT3。

`_legacy/`仅供历史查阅，默认不编辑、不参与活跃事实和构建。

## 4. 高危同名文件

`motor.c/h`、`IMU.c/h`、`PID.c/h`、`lowPassFilter.c/h`、`commonDef.h`在不同核语义不同。编辑前必须在回复中公告目标完整路径；除非用户明确要求，不在一次任务中跨核修改同名高危文件。

跨核共享库的同源/分叉关系见[LIBRARY_VERSIONS](docs/LIBRARY_VERSIONS.md)。

## 5. 嵌入式安全红线

- ISR只做清标志、最小采样、计数或入队；禁止显示、printf、Flash、阻塞串口、大块复制、视觉算法和复杂状态机。
- 逐飞`interrupt_global_disable()`/`interrupt_global_enable()`存在已确认计数漂移恶性bug，应用代码严禁调用。
- 全局临界区使用真实PRIMASK保存/恢复；能屏蔽单一IRQ时不得扩大为全局关中断。
- ISR与前台共享状态必须明确`volatile`、所有权、原子性和发布顺序。
- M7共享内存必须维护DCache Clean/Invalidate、magic/seq和一致性读取；读取碰撞不得伪造成合法全零事实。
- 所有外部数据、浮点控制量和执行器输出必须做有限数、范围、新鲜度和状态门控；不确定性fail-closed。
- 硬件/外设只有一个owner负责初始化、ISR、状态和直接控制；调用方使用公开facade、消息或snapshot。
- 禁止修改逐飞/SeekFree/SDK、startup/system和链接脚本，除非用户明确授权对应底层任务。

详细规则见[embedded-safety](.claude/rules/embedded-safety.md)和[hardware-ownership](.claude/rules/hardware-ownership.md)。

## 6. 编码与接口

- 所有C/H和Markdown使用UTF-8无BOM；外部SeekFree文件导入时先识别编码再转换。
- 裸机C接口以owner facade为边界：生命周期、语义命令、周期入口、一次性snapshot。
- 调用方不得持有owner context、包含internal头、写入owner字段或依赖厂商DTO布局。
- Snapshot按值复制并携带必要的`seq/valid/age`；禁止返回ISR仍可能修改的内部对象指针。
- 新增通用算法优先复用现有实现；确需新增时对象式封装在`util/`，不在调用点重复展开。
- 新增/移动源码必须同步EWP、include路径和`.clangd`，并在编码阶段结束后集中构建受影响IAR配置。

详见[embedded-c-style](.claude/rules/embedded-c-style.md)和[source-integration](.claude/rules/source-integration.md)。

## 7. 文档与证据

- 软件当前行为以活跃源码、公开头、codec、schema和真实调用链为准；文档负责解释和导航。
- 物理接线不能只由源码裁决，必须结合板卡/原理图和实线证据。
- `CODE-CONFIRMED`不代表能构建或板端通过；当前证据统一登记在[EVIDENCE_STATUS](docs/EVIDENCE_STATUS.md)。
- 代码、接口、参数、协议、owner或UI语义变化时，同批更新其权威文档；不执行固定的全仓文档清单。
- PROJECT只保存当前目标入口；历史决策进ADR，已确认复用事故进lesson，一次性过程不进入长期文档。
- 相关编辑前搜索[lessons INDEX](.hecateflow/lessons/INDEX.md)，落实命中的避免动作。

事实边界见[FACT_OWNERSHIP](docs/FACT_OWNERSHIP.md)，文档地图见[docs/README](docs/README.md)。

## 8. 协作与验证

- 主Agent先自主读取仓库事实，再按复杂度使用只读子代理扩大覆盖；跨核、ISR、共享内存、协议、安全、删除和治理任务必须多路复审。
- 写入worker只在用户已授权、方案完整、文件范围互斥时使用；不得写规则边界或越过分配范围。
- 子代理结论必须由主Agent用源码和只读命令亲验。
- 每个编码阶段完成后运行最小源码与静态检查；文档治理运行`tools/docs_audit.py`，视觉改动只按需运行保留的`^`灯板或信标回放。所有代码闭合后再集中IAR构建，避免频繁编译。
- 无IAR或硬件时明确记录`NEEDS-EVIDENCE`，禁止以文档审计、视觉回放或静态检查冒充构建或实机PASS。
- 每次业务代码修改完成后，交付报告必须提醒用户需要烧录的目标板卡；只使用`Core0`、`Core1`、`Core2`、`Core3`、`Core4`、`CoreT3`粒度，不细分`cm7_x`，因为各板均采用双核联合下载。
- 完成后报告修改文件、实现摘要、验证结果、证据债务、复审结论、建议提交说明和显式暂存清单，然后等待Git确认。

任务生命周期见[task-lifecycle](.claude/rules/task-lifecycle.md)。

---
> Source: [HecateFake/21st-smartcar-flying-minefield](https://github.com/HecateFake/21st-smartcar-flying-minefield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
