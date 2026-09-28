## 26nuedc-h

> 这是一个基于 MSPM0G3507 的嵌入式 C99 智能车工程，使用 CMake 与 ARM GNU 工具链构建。工程包含菜单、显示、灰度、编码器、电机、舵机、PID、循迹、Flash 参数保存等应用模块。

# Repository Guidelines

## 0. 项目速览

这是一个基于 MSPM0G3507 的嵌入式 C99 智能车工程，使用 CMake 与 ARM GNU 工具链构建。工程包含菜单、显示、灰度、编码器、电机、舵机、PID、循迹、Flash 参数保存等应用模块。

> **路径约定：** 除特别说明外，本文件中的 `project/code/...`、`build/...`、
> `cmake --build build/debug`、`flash.sh`/`flash.ps1` 等路径与命令均相对
> **`firmware/`** 目录（仓库根目录下主控固件工程所在位置）执行。
> `vision/`、`report/`、`tools/`、`docs/` 为仓库根目录下的平级子目录。

入口与关键调度：

```text
firmware/project/user/src/main.c        启动入口，只保留初始化和进入运行框架
firmware/project/user/src/isr.c         底层中断入口，调用逐飞 callback 表
firmware/project/code/core/app_runtime.c 主循环调度
firmware/project/code/core/app_interrupts.c 应用层 PIT/EXTI 绑定与中断任务调度中枢
```

`project/code` 下源码不再平铺，必须按职责放入三个子目录：

```text
project/code/
  core/      系统运行、调度、参数、Flash、PID、循迹、黑板等核心应用层
  device/    传感器、执行器、显示、按键、编码器等硬件封装层
  menu/      Easy Menu 框架、用户菜单页和菜单门面
```

典型文件职责：

```text
core/
  app_runtime.c       主循环调度
  app_interrupts.c    应用层 PIT/EXTI 中断绑定中枢
  app_blackboard.c    全局黑板数据
  app_tracking.c      循迹与速度 PID 运行逻辑
  app_balance.c       摄像头小球/步进滑轨位置维持外环
  app_ladrc.c         二阶对象LADRC、TD、LESO和带宽参数化反馈
  app_pid.c           PID算法；参数接口保持float，控制周期使用Q16.16定点内核
  app_param_store.c   可保存参数缓存与 Flash 读写统一入口
  app_flash_store.c   Flash 通用记录读写工具
  app_config.c        系统配置
  app_boot.c          启动日志
  app_system_load.c   系统负载统计
  app_control_uart.c  UART1 控制命令、参数配置、状态查询与遥测
  app_stepper_control.c 步进电机统一控制锁、位置环与速度/位置命令区

device/
  app_display.c       IPS 显示缓存与统一 flush
  app_keys.c          按键扫描与短按事件缓存
  app_beep.c          蜂鸣器非阻塞状态机
  app_motor.c         电机控制
  app_electromagnet.c 电磁铁 PWM API（当前不初始化、不注册菜单，B4 已释放）
  app_servo.c         舵机控制与 steer 查表插值
  app_encoder.c       编码器采样
  app_camera_uart.c   UART2 B15/B16钢球v5二进制协议解析与帧率统计
  app_zdt_stepper.c   UART1 B6/B5 Emm V5.0闭环步进驱动命令与响应
  app_gs08ra.c        八路灰度
  app_attitude.c      姿态调度与 IMU 数据处理
  app_battery.c       电池 ADC
  imu963ra_attitude.c IMU963RA 姿态解算封装

menu/
  app_menu.c          菜单门面，负责物理按键到 Easy Menu 的适配
  Easy_Menu*.c/.h     Easy Menu 框架源码
  Easy_Menu_User.c    所有菜单页面、菜单项、页面回调集中定义
```

当前主菜单注册 `ZDT Stepper` 子菜单，通过UART1 B6/B5控制Emm V5.0闭环步进驱动：

```text
Jog RPM：零点校准点动速度，1..5000 RPM可调，步长100 RPM，默认300 RPM
Jog step deg：零点校准页面的单次相对点动角度，默认100°
Center Cal：唯一校准入口；KEY2/KEY3短距离点动，KEY1把小球恰好静止的位置标定为0°
Sine Speed：KEY1启停±5000 RPM正弦测试，完整周期固定1 s；KEY4强制零速并返回
```

当前已知未解决问题：进入`Sine Speed`后仍可能出现约-2900°的固定中心迁移。
此前删除页面层`-2392°`补偿状态机后现象仍存在，说明根因不只在菜单进入流程，
还可能与Emm驱动器内部残留位置目标、速度模式切换或驱动坐标反馈有关。后续排查前
禁止再次添加经验偏移补偿，也不得把该偏移当成已经校准的机械零点。
零点校准收到驱动器0x0A“当前位置清零”成功应答后必须立即置`calibrated=1`并结束；
禁止再发送绝对位置0°进行二次验证，该多余位置命令会触发约-2.9k°残留目标，
导致校准永不完成，并使统一速度安全层把Sine、Rail/Ball Hold全部输出压成0。

`ZDT Stepper -> Position Hold`包含三个自动运行Show Page：

```text
Rail Zero Hold：不读取小球位置，只以驱动坐标0°为目标产生步进速度，用于排查-2.9k偏移
Ball Zero Hold：小球位置PD产生目标杆角，滑轨位置P再产生步进速度
Deadband 0.1cm：小球误差死区半宽，步长0.1 cm，默认4即±0.4 cm
Max rail deg：静止起滚Boost最大电机轴角，默认2100°，范围100..3000°
Run rail deg：小球已经滚动后的常规控制最大杆角，默认1200°
Max RPM：滑轨位置内环速度上限，默认900 RPM
Rail Kp：滑轨位置内环P增益，默认0.2
+5 / -5 Traverse：从中心±1 cm内自动执行+5 cm折返至-5 cm；正端和负端容差均为±0.8 cm，
                 负端速度≤1 cm/s连续200 ms判定稳定，要求5 s内完成。页面进入即启动，
                 KEY1可在启动条件不足时重试，KEY4立即停止并上锁。
Rail Self Test：进入页面后再次KEY1确认，依次执行0→正峰值→负峰值→0；
                 每次到位鸣笛并停留2 s，最终回零后自动返回
```

位置维持控制位于`app_balance.c`，只复用通用`app_pid_t`算法，不复用面向双轮编码器的
`app_position`业务模块。摄像头控制只在有效帧计数`total_frames`变化时更新，禁止对同一
45 Hz数据帧在100 Hz节拍中重复计算D项；外部`sequence`可能复用，不得作为唯一的新帧依据。
超过100 ms无有效摄像头数据时请求速度必须归零。往返页使用独立整数轨迹PD：起步阶段
使用-2600°非阻塞起滚脉冲，检测到小球运动后切换到P=30的轨迹控制；正程D=10，回程D=30，
滑轨内环Kp=0.3、限幅1500 RPM，滑轨位置内环死区为±5°。端点微调只在误差超过±0.8 cm
且球速不超过0.5 cm/s时使用短脉冲，不允许在容差内持续动作。
由于当前存在约-2.9k°驱动坐标异常，Center Cal和Position Hold页面退出只允许立即
零速、停止并上锁，禁止调用`app_zdt_stepper_return_zero_on_exit()`，否则回零状态长期
无法满足0°到位条件并通过`app_zdt_stepper_input_busy()`锁死包括Sine Speed在内的菜单按键。
Rail Self Test正常完成时仍按其内部状态机回到0°后再退出。
小球控制采用串级结构：外环位置PD输出目标滑轨角，内环以驱动器真实位置反馈输出RPM。
小球零点保持的2026-07-30联调基线为：外环P=90、D=60、死区±0.4 cm，
Boost最大角2600°、常规角度限幅1800°；滑轨内环P=0.25，速度上限1200 RPM。
实测从约+1.99 cm回到+0.20 cm，约4.6 s进入死区，仍有数次衰减振荡，
该组参数作为继续增加D阻尼前的可回退版本。
实测机械中位约±500°不足以起滚，约1500°接近静摩擦阈值，约2100°会快速滚动，
因此控制器采用“静止起滚Boost + 常规低倾角PD”：小球位于死区外至少0.1 cm、速度
不超过0.2 cm/s并持续300 ms时进入Boost。Boost角按位置误差从近端1600°平滑增加，
误差达到3 cm时使用配置的2100°最大角；速度达到0.4 cm/s后立即退出Boost，常规目标角
限制为±1200°。Boost持续2.5 s仍未起滚时不得退出零点维持：先降回常规倾角冷却
0.8 s，再重新执行300 ms静止判定并尝试Boost；每次连续失败把近端Boost角提高200°，
直到配置的最大角，检测到起滚或进入“死区+0.1 cm”迟滞区后清零重试次数。迟滞区
用于避免摄像头噪声在死区边缘反复触发大倾角。只有用户退出、
反馈失效、校准失效或安全边界阻止运动时才允许停止输出；禁止长期保持大倾角。
禁止恢复为“小球PD直接输出RPM”的持续积分结构。
D项使用0.05 s一阶低通滤波。
当摄像头协议`MOTION_VALID`有效时，D项直接使用其`velocity_centi_cm_s`
（0.01 cm/s）作为测量变化率；该位无效时才退回相邻位置帧差分。P项始终使用
`position_centi_cm`位置误差。摄像头只在新帧时更新目标角，滑轨内环每10 ms运行。
UART0支持`@BALL_P:<value>`、`@BALL_D:<value>`、`@BALL_PD:<P>,<D>`立即修改RAM参数，
`@BALL_PD?`查询当前参数。设置成功依次返回原指令、`@OK:BALL_PD`和实际生效的
`@BALL_PD:<P>,<D>`；`@BALL_DB`调整死区，`@BALL_ANGLE`调整Boost角，
`@BALL_RUN_ANGLE`调整常规控制角，`@BALL_MAX`调整内环速度上限，
`@BALL_RAIL_KP`调整滑轨内环P。`@BALL?`末尾包含常规角、内环P、Boost状态和重试等待状态。
当前机构实测负滑轨角使负位置小球向视觉零点运动，因此外环目标角必须反相。
这些参数当前不写Flash，上电恢复代码默认值。

常用串口调试统一使用`tools/ball_tune.py`，依赖见`tools/requirements.txt`：

```sh
python3 tools/ball_tune.py status
python3 tools/ball_tune.py set --kp 30 --kd 30 --boost-angle 2100 --run-angle 1200 --max-rpm 900
python3 tools/ball_tune.py run --seconds 8 --csv temp/ball.csv
python3 tools/ball_tune.py zero
python3 tools/ball_tune.py stop
```

`run`默认检查球位置、球速和杆角，任何异常或Ctrl-C都会先停止，随后自动将杆恢复到0°。
除非正在排查归零问题，不要使用`--no-return-zero`。

步进速度必须经过`app_zdt_stepper_request_safe_velocity_rpm()`统一安全门。该接口先检查
VELOCITY控制权锁，再检查校准状态、位置反馈、速度制动提前量以及
绝对坐标-5300°..+5300°硬碰撞边界。该对称范围为当前重新确认的可用行程；
调整该边界前必须重新进行机械测定，禁止仅为扩大控制量而放宽。
上层不得直接发送速度帧。历史17%（约±1640.5°）业务限幅已经删除；位置命令、
速度安全层、自检和测试功能直接使用-5300°..+5300°作为唯一行程边界。

正常业务只通过`app_zdt_stepper_set_balance_velocity_rpm()`提交-5000..5000 RPM
有符号速度；上位机只提供小球位置，目标点与当前位置的控制计算必须由MCU应用层完成。
驱动器真实位置按20 ms周期查询，速度帧与位置查询帧在主循环10 ms调度中交替发送；
反馈超过100 ms、尚未完成零点校准或继续朝已到达的边界运动时，实际速度强制为0，
但允许反向驶离边界。机械全行程固定为19300°，校准零点两侧强制使用机械半行程
-5300°..+5300°对称坐标限幅；这是当前重新确认的机械可用行程。
高速下限幅器额外按速度加入一个反馈周期的制动提前量。

零点校准时仍使用短距离相对位置命令完成按键点动，因为短按直接启用连续速度会在
按键松开后继续运动。该位置命令仅服务校准，不得作为正常平衡控制方式。校准、限幅、
点动速度、步长和正弦周期当前均为RAM运行时参数，每次上电需要重新校准，不写Flash。
菜单发送和`app_zdt_stepper_tick()`均发生在主循环，禁止把UART轮询发送接口直接放入
100 Hz控制中断。

ZDT发送权限由设备层`app_zdt_stepper_access_mode`统一管理：

```text
LOCKED：页面外默认状态，不周期发送速度或位置查询
CALIBRATION：只允许校准点动、清零和位置查询，禁止速度帧打断位置模式
VELOCITY：只允许速度目标与位置查询
HOMING：厂家回零独占状态，只允许周期查询0x3B回零状态
```

所有ZDT Show Page进入时必须显式`app_zdt_stepper_unlock()`取得对应权限，退出时必须
调用`app_zdt_stepper_safe_lock()`发送立即停止并上锁。回零期间菜单门面直接屏蔽全部
物理短按，不修改Easy Menu框架；屏幕状态栏显示`ZDT HOMING`。0x3B状态字bit2表示
正在回零、bit3表示回零失败；完成或失败后设备层保持LOCKED，必须由后续页面重新
取得权限。禁止页面绕过权限状态机直接发送运动帧。

当前零点校准使用驱动器多圈当前位置清零，不等同于厂家单圈/碰撞原点参数。退出
Center Cal或Sine Speed页面时调用`app_zdt_stepper_return_zero_on_exit()`：先发送
立即停止，等待至少10 ms后发送多圈绝对位置0命令，随后按20 ms查询真实位置。
进入±5°并连续三次反馈稳定才视为完成；30 s仍未到位则发送停止并标记失败。
自动返回期间按键全部无效，完成或失败后保持LOCKED。不得用厂家0x9A回零替代这个
退出回零流程，除非已经单独完成厂家原点参数配置。

Center Cal的KEY1确认采用不可打断状态机，禁止退化为直接发送0x0A：

```text
FE停止旧目标
  ↓ 至少10 ms
F6零速，明确退出旧FD位置控制
  ↓ 至少10 ms
0x0A把驱动器当前位置清零
  ↓ 等待成功应答
FD绝对位置0，覆盖驱动器内部残留目标
  ↓
0x36连续验证三次处于±5°
  ↓
校准成功
```

校准全过程由`app_zdt_stepper_input_busy()`锁定物理按键，3 s超时会发送停止并保留
未校准状态。这样可避免用户确认后立即返回，导致0x0A应答尚未处理、旧FD目标在
下一次使能时把滑轨拉回历史位置。

当前硬件实测进入Sine Speed并重新使能后，驱动器会固定恢复到约-2392°。在驱动器
内部原因暂不继续追查的前提下，Sine Speed页面包含专用软件补偿：等待位置到达
-2392°（允许±50°，最长等待3 s），随后通过安全回零状态机返回驱动坐标0。补偿
完成后只切换为F6零速，不再次发送F3使能；页面显示READY后KEY1才允许启动正弦。
该补偿只属于当前Sine Speed测试页，不得扩散到通用ZDT坐标或其它业务控制。

厂家STM32F1例程明确要求任意两条ZDT命令之间保留5～10 ms间隔；无间隔连续发送时，
驱动器可能把两帧判定为一条错误命令并返回`01 00 EE 6B`。因此页面进入时单独发送
F3使能，后续速度帧和位置查询帧按10 ms节拍交错。未来球位置控制即使按60 Hz更新
速度目标，也只允许写入最新目标，由设备层10 ms状态机限速发送；禁止同一调度点
连续调用两个发送接口，若业务确需连发，应增加非阻塞发送队列。

当前主菜单不注册舵机和蜂鸣器测试页，以减少非核心调试入口。该精简只作用于
`Easy_Menu_User.c` 的页面注册：`app_servo` API 仍保留供未来车型使用，
`app_beep` 仍负责校准、保存等业务提示，20 ms 蜂鸣器非阻塞 tick 也必须保留。

主菜单包含面向赛题验收的 `Answer` 页面。该页面只封装已经完成调试的能力，
后续赛题按 `Question N` 子页面逐项加入，不在答案页面内复制底层控制算法。
当前 `Question 2` 复用 Tracking 的灰度循迹、停车线计圈和运行计时：

```text
Question 2/
  Laps：目标圈数，1..99，步长1，默认1
  Speed %：循迹基础速度，15..40%，步长1%，默认35%
  Run：KEY1发车；达到目标圈数后停车并显示最终用时；KEY4停车并返回

Question 3：
  进入页面保持停止，第一次KEY1启动Ball Zero中心主动维持，不开始计时
  第二次KEY1释放中心维持并启动+5 cm到-5 cm往返，同时从零开始计时
  首次进入最终-5 cm的±1 cm范围时锁存并停止运行计时
  到达后LADRC继续维持最终点；稳定等待和后续保持时间不计入赛题运行时间
  页面持续显示锁存用时；KEY4先停止并归还步进owner锁，再启动非阻塞回逻辑0点
  回零期间由app_zdt_stepper_input_busy()屏蔽按键；到位或超时上锁后停在Answer

Question 4/
  Distance count：定距缓刹触发行程，50..100000 control count，步长50，默认2900；
                  可直接按键设置，也会被Sample A-B完成后的采样结果覆盖
  Speed %：A到B循迹基础速度，15..40%，步长1%，默认23%
  默认路径长度：2900 control count，可通过Sample A-B重新采样覆盖
  Sample A-B：KEY1从A点启动采样，再次KEY1在B点停车并记录平均编码器行程
  Run A-B：KEY1按已采样行程启动；到达目标距离自动停车并显示最终距离和用时
  Question 4和Challenge的目标行程是最终缓刹触发点，不是硬停车点；触发后继续
  受限循迹并按Task accel cps2把速度目标降到零，实际轮速低于停止阈值后才退出
  速度PID、冻结计时并恢复显示，因此允许停车位置超过目标行程。

Question 6/
  Position cm：小球目标位置，范围-10..+10 cm，按键步进0.1 cm

Challenge/
  Position X cm：正位置幅值，范围0.1..10.0 cm、步进0.1 cm，默认7.0 cm
  Distance count：总行程，默认与Question 4相同为2900 control count，
                  范围50..100000、步进50；Sample Path完成后用采样值覆盖
  Speed %：循迹速度，默认23%
  Sample Path：KEY1开始采样，再次KEY1停止并记录左右轮平均绝对行程
  Run：第一次KEY1启动+X位置维持，第二次KEY1启动定距循迹和计时
  运行中按a=当前行程/总行程计算小球目标target=X-2*X*a，
  小车到达总行程时停车、冻结计时并持续维持-X；KEY4停止全部任务、
  释放步进owner并触发非阻塞回零保护后返回Challenge
  参数和采样行程只保存在RAM，不写Flash
```

`codex/ball-ladrc`分支的小球零点保持默认使用二阶LADRC：`wc=2.5 rad/s`、
`wo=10 rad/s`、`b0=-0.2`、TD关闭。LADRC按约45 Hz摄像头有效新帧更新，
输出滑轨目标角；100 Hz滑轨位置P环和全部步进安全层保持不变。误差达到2 cm
才允许Boost；进入±0.4 cm且速度不超过0.05 cm/s持续200 ms后锁定零输出，
误差重新达到±0.6 cm才退出静置。PD基线仍可通过`@BALL_CTRL:0`切回。
`+5 / -5 Traverse`同样使用该二阶LADRC产生滑轨目标角；往返状态机只负责
切换+5/-5 cm目标、端点容差和完成判定，不得恢复独立整数PD或起步脉冲算法。

问题2、问题3、问题4参数和问题4的采样路径长度当前只在RAM中保存，不写Flash。进入
Run页只应用参数并保持停车，必须再次按KEY1才开始运动和计时；运行期间仍关闭
IPS硬件刷新，停车后恢复刷新并保留在当前赛题页面显示最终圈数/距离、时间和
DONE状态。`Tracking`页面只保留左右目标和反馈等底层诊断项，不再重复注册
Run、圈数、速度、Distance Sample和Distance Run等赛题业务入口。

当前启用2号小车映射。1号/2号小车的软硬件关系必须分别保留，切换车辆时只允许
修改设备层映射，禁止在PID层交换左右输出或修改控制符号。

1号小车（历史已验证映射）：

```text
物理左轮电机：B13 正向 / B12 反向，对应左编码器 E2 与左速度 PID。
物理右轮电机：A0 正向 / A1 反向，对应右编码器 E1 与右速度 PID。
```

2号小车（当前新控制板，方向与左右归属均按实测修正）：

```text
物理左轮电机：A1 正向 / A0 反向，对应左编码器 E2 与左速度 PID。
物理右轮电机：B12 正向 / B13 反向，对应右编码器 E1 与右速度 PID。
修正依据：Motor Control同时设置+35%时，旧映射使两轮后退且反馈均约-400 cps；
因此先交换每一路正反PWM。随后PID Debug零目标测试确认单独转动一侧会驱动
另一侧电机，证明2号小车左右电机执行通道相对1号小车发生互换；因此进一步
交换左右电机PWM归属，但编码器E2/E1与左/右PID对象的归属保持不变。
PID Debug选择ZERO时固定左右目标为0 count/s，用手动转动单轮检查通道联动：
转动左轮只能改变左反馈并驱动左PID/左电机，右轮同理。
```

两辆车均遵守：正电机命令表示车辆前进，正编码器反馈表示车辆前进；不得在PID层
通过交换左右输出或取反误差修补硬件映射。

不要把新增业务源码放到 `project/code` 根目录。

## 1. 构建、下载与代码风格

以下命令均在 `firmware/` 目录内执行。

常用编译：

```sh
cmake --preset debug
cmake --build build/debug
```

常用下载：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\flash.ps1 -FirmwareElf .\build\debug\TestDemo
```

```sh
./flash.sh
```

当前 CMake 递归收集 `project/code/*.c`：

```cmake
file(GLOB_RECURSE SEEKFREE_PROJECT_CODE_SOURCES CONFIGURE_DEPENDS
    "${SEEKFREE_ROOT}/project/code/*.c")
```

代码风格：

- C99。
- 4 空格缩进。
- 函数和控制块大括号独占一行。
- 优先使用工程固定宽度类型：`uint8`、`uint16`、`uint32`、`int16`、`int32`。
- 应用模块使用 `app_` 前缀。
- 设备强相关模块可保留设备名前缀，例如 `imu963ra_`、`gs08ra_`。
- 不要继续膨胀 `main.c`。
- 不要提交构建产物、IDE 配置、临时日志或本地示例。

新增或整理注释时，采用 `project/code/core/app_pid.c/.h` 的注释风格：

```text
头文件公开接口：
  使用 Doxygen 风格块注释。
  至少写清 @brief。
  有参数时写 @param。
  有返回值时写 @return。
  有调用时机、边界行为、副作用时写 @note。

源文件内部函数：
  简短辅助函数可用一到三行块注释说明目的。
  复杂计算函数要说明计算步骤、单位、边界处理和为什么这样处理。

结构体字段：
  使用行尾注释说明物理意义、单位或状态含义。

注释内容：
  优先解释“为什么”和“单位/边界/调用时机”，不要只复述函数名。
```

示例风格：

```c
/**
 * @brief 执行一次位置式 PID 计算。
 * @param pid PID 控制器对象指针。
 * @param target 目标值。
 * @param measurement 当前测量值，必须与 target 使用相同物理单位。
 * @param dt_s 本次控制周期，单位 s，必须大于 0。
 * @return 完成限幅后的控制量。
 * @note dt_s 非法时保持并返回上次输出。
 */
```

修改后至少执行 `cmake --build build/debug`。涉及时序、硬件、中断、Flash 或菜单刷新策略时，能上板则执行下载验证。

## 2. 当前调度事实

运行链路：

```text
main.c
  ↓
app_runtime_run()
  ↓
app_interrupts_init()
  ↓
统一绑定 PIT/EXTI
```

应用层中断绑定位置：

```text
project/code/core/app_interrupts.c
```

当前中断任务表：

```text
20 ms PIT:
  - app_keys_scan_periodic()
  - app_beep_tick(APP_KEYS_SCAN_PERIOD_MS)

10 ms / 100 Hz PIT:
  - 累计通用主循环10 ms pending节拍
  - app_encoder_update(APP_ATTITUDE_UPDATE_PERIOD_MS)
  - 根据控制模式选择：0 停止、1 循迹方向环+速度环、2 速度 PID 调试、
    3 位置环+速度环、4 惯性角度环+速度环
  - PID调试模式将左右归一化实际轮速、左右零目标和左右PID输出写入UART遥测队列
  - 从黑板读取左右轮最终速度命令并统一调用 app_motor_set_wheel_speed_percent()

EXTI B10:
  - app_encoder_left_phase_a_rising()

UART0 RX，A10-TX/A11-RX，115200 baud，优先级 2:
  - 缓存 PID 串口调参字节；协议解析、应用和 Flash 保存放在主循环
  - 物理链路使用 DAPLink 虚拟串口；printf 日志与控制协议共用该UART

UART1 RX，B6-TX/B5-RX，115200 baud，优先级 2:
  - 专用于Emm V5.0闭环步进驱动响应接收
  - 中断只缓存字节，四字节响应组帧和解释放在主循环
  - 运动命令发送只能从主循环调用，禁止从100 Hz控制中断直接发送

UART2 RX，B15-TX/B16-RX，115200 baud，优先级 2:
  - 中断只排空 FIFO 并缓存摄像头上位机字节
  - 主循环解析固定14字节钢球v5二进制帧：AA 5A 05、little-endian、CRC-16/MODBUS
  - 数据包含seq、status、位置0.01 cm、速度0.01 cm/s、加速度0.01 cm/s²
  - Camera UART Show Page显示球位置、速度、加速度、状态位、帧率和CRC/版本/溢出错误
  - VALID=0时不得使用位置；MOTION_VALID=0时不得使用速度和加速度
```

中断规则：

- 所有应用层中断资源绑定必须集中在 `app_interrupts.c`。
- 中断初始化必须在所有模块初始化完成后，最后统一调用 `app_interrupts_init()`。
- 不要在各业务模块 init 中私自 `pit_ms_init()` 或 `exti_init()`。
- 中断回调命名应体现周期或资源，不应被业务模块污染，例如 `app_interrupts_10ms_control_callback()`。
- 中断内不得执行阻塞延时、Flash 写入、菜单绘制、大量串口输出或长时间硬件通讯。

## 3. 显示刷新架构

显示层位于：

```text
project/code/device/app_display.c
```

当前显示为缓存式刷新：

```text
app_display_printf()
app_display_show_string()
app_display_clear()
app_display_show_menu_char()
    ↓
只写 app_display_line_buffer 等缓存
只标记 dirty
不直接产生 IPS 通讯

app_display_flush_dirty()
    ↓
统一把 dirty 行发送到 IPS 屏幕
产生实际屏幕通讯
```

`app_display_flush_dirty()` 是常规运行阶段唯一屏幕通讯入口。主循环控制屏幕刷新扰动时，只控制这个函数是否执行。

菜单侧刷新开关在 `app_menu.c`：

```c
menu_refresh_enabled
```

常规刷新点：

```c
if(menu_refresh_enabled)
{
    app_display_flush_dirty();
}
```

循迹运行时通过：

```c
app_menu_set_refresh_enabled(!app_tracking_get_run());
```

关闭菜单屏幕刷新，避免 IPS 通讯影响控制周期。

显示规则：

- 不要绕过 `app_display_flush_dirty()` 直接调用 IPS200Pro 显示通讯函数。
- Show Page 周期刷新时不要反复 `app_display_clear()`。
- 自定义页面应在 `Enter` 中清屏一次，在 `Period` 中只覆盖变化行。

## 4. 菜单系统架构

菜单采用分层结构：

```text
app_menu.c/.h
  应用菜单门面，负责物理按键与 Easy Menu 的唯一适配入口。

Easy_Menu_User.c
  页面、条目、父子关系和页面回调集中注册。

Easy_Menu_Core.c / Easy_Menu_Page.c / Easy_Menu_Item.c
  Easy Menu 框架源码，不承载具体业务页面。
```

禁止：

- 在 `app_menu.c` 中为电机、灰度、PID 等具体页面增加特判。
- 在 Easy Menu 框架源码中为单个业务页面添加特判。
- 通过页面标题字符串判断页面身份。
- 页面自行改变物理按键映射。

固定按键映射：

```text
KEY1 -> EASY_MENU_RIGHT -> 确认 / 执行
KEY2 -> EASY_MENU_DOWN  -> 下移 / 减小
KEY3 -> EASY_MENU_UP    -> 上移 / 增大
KEY4 -> EASY_MENU_LEFT  -> 返回 / 取消
```

功能页可以解释四个逻辑输入的业务含义，但必须保留 KEY4 作为返回/取消。

## 5. 新增菜单选项流程

新增菜单项前，agent 必须先判断或询问以下问题：

```text
1. 这是普通菜单项，还是需要自定义布局的 Show Page？
2. 按键是否使用默认映射 KEY1确认、KEY2下/减、KEY3上/增、KEY4返回？
3. 是否真的需要特殊按键解释？如果需要，只能在 Easy_Menu_User.c 的页面回调中解释逻辑输入，不能改物理映射。
4. 内容是否超过一屏？如果超过，是使用普通滚动菜单，还是 Show Page 内部自行分页/滚动？
5. 是否需要特殊折叠渲染？例如开启某个选项后才显示相关设置。
6. 是否需要保存参数？如果需要，保存是否由用户明确 Apply/Save 触发？
7. 是否涉及硬件通讯或耗时计算？如果涉及，是否需要降低刷新频率或拆出非阻塞状态机？
```

新增普通菜单项集中修改：

```text
project/code/menu/Easy_Menu_User.c
```

普通菜单适合：

```text
开关
数值修改
只读显示
点击执行动作
跳转页面
内容较多但结构简单的滚动菜单
```

Show Page 适合：

```text
自定义布局
实时数据页面
校准流程
PID 调参页
电机按键控制页
开启某个选项后动态渲染相关设置的折叠页面
需要自己处理输入和绘制的功能页
```

普通菜单项实现步骤：

```text
1. 声明 Item 对象。
2. 将 Item 加入对应页面 Item* 数组。
3. 在 Easy_Menu_Ui_Init() 中初始化 Item。
4. 业务动作通过 core/device 模块公开接口执行。
5. 如需保存，菜单只触发 app_param_store 的保存接口，不直接写 Flash。
```

Show Page 实现步骤：

```c
static void xxx_enter(void);
static void xxx_period(void *temp, Easy_Menu_Input_TYPE user_input);
static void xxx_exit(void);
```

职责：

```text
enter:
  初始化页面状态
  清屏一次
  绘制固定布局

period:
  处理 user_input
  更新动态数据
  只覆盖需要变化的行
  不要周期性整屏清屏
  不要阻塞
  不要写 Flash，除非是用户确认动作并且不在中断中

exit:
  清理状态
  必要时清屏或恢复模块状态
```

## 6. 新增中断流程

新增中断前，agent 必须先询问或确认：

```text
1. 中断来源是什么？PIT、EXTI、UART、ADC 还是其它？
2. 周期是多少？例如 1 ms、5 ms、10 ms、20 ms、100 ms。
3. 优先级要求是什么？是否必须高于当前 100 Hz 控制中断？
4. 中断内执行哪些任务？每个任务最坏耗时是否可控？
5. 是否包含硬件通讯？如 SPI/I2C/屏幕/Flash/串口批量输出，通常不应放入中断。
6. 是否只是设置 pending 标志，实际工作放主循环？
7. 是否会与现有 PIT/EXTI/编码器/速度 PID 资源冲突？
```

新增中断绑定位置：

```text
project/code/core/app_interrupts.c
```

初始化顺序要求：

```text
所有模块 init 完成
  ↓
最后统一调用 app_interrupts_init()
  ↓
再进入 while 主循环
```

禁止在模块初始化中提前启用中断，避免中断访问尚未初始化完成的模块状态。

新增 PIT 中断建议命名：

```c
static void app_interrupts_XXms_xxx_callback(uint32 event, void *ptr);
```

新增 EXTI 中断建议命名：

```c
static void app_interrupts_xxx_exti_callback(uint32 event, void *ptr);
```

中断内优先做：

```text
计数
置标志
读取极少量 GPIO
调用确定轻量、无阻塞的模块 tick/update
```

中断内避免：

```text
Flash 写入
屏幕刷新
printf 批量输出
I2C/SPI 长通讯
阻塞延时
复杂浮点计算
菜单绘制
```

## 7. 将某个任务移动到中断流程

当用户要求“把某任务移到中断”时，agent 必须先列出当前中断任务表，并询问或确认：

```text
1. 目标周期是多少？
2. 目标优先级是多少？
3. 是否可以与现有 10 ms / 100 Hz 控制中断共用？
4. 是否可以与现有 20 ms 低频任务共用？
5. 任务是否包含硬件通讯？
6. 任务是否必须在中断中完成，还是只需要中断置 pending 标志、主循环执行？
7. 若任务是解算类任务，解算输入采样周期和输出更新周期是否一致？
```

当前中断任务表必须作为评估基准(每次修改后实时更新下面text内容)：

```text
20 ms PIT:
  - 按键扫描
  - 蜂鸣器 tick

10 ms / 100 Hz PIT:
  - 通用主循环10 ms调度标志
  - 编码器采样
  - 多模式控制分支：0 失能、1 循迹、2 PID 调试、3 位置、4 惯性角度
  - 模式 1 执行方向误差计算和速度 PID
  - 模式 2 只执行速度 PID，并将四项遥测数据放入串口队列
  - 模式2的PID菜单可选择ZERO或SQUARE目标信号
  - ZERO：左右目标恒为0 count/s，用于单轮联动检查
  - SQUARE：0%/40%目标方波，完整周期8 s，每个电平保持4 s
  - app_tracking_pid_debug_isr_update()每10 ms执行完整闭环速度PID；切换信号时先清零左右PID历史、目标和电机命令
  - 模式 4 先由 app_angle_update_isr() 将融合相对yaw误差转换为onto_cps，
    再执行左右轮速度PID；角度环不得直接写电机PWM或百分比命令
  - 模式 4 的 @ANG:<degree> 表示相对姿态零点的绝对目标航向，不是相对当前
    航向累加；到位后继续闭环保持，退出Angle菜单或显式停止时必须清零退出
  - 速度、循迹方向、位置和惯性角度控制均调用app_pid_update_fixed()；
    float参数只允许在初始化或调参时转换，100Hz控制路径不得改回浮点PID运算
  - 读取黑板速度命令并统一下发左右轮电机

EXTI B10:
  - 左轮软件编码器 A 相上升沿

UART0 RX，A10-TX/A11-RX，115200 baud，优先级 2:
  - DAPLink PID串口调参字节接收和printf日志输出

UART1 RX，B6-TX/B5-RX，115200 baud，优先级 2:
  - Emm V5.0闭环步进驱动响应字节进入环形缓冲
  - 响应组帧放在主循环，不在中断内执行

UART2 RX，B15-TX/B16-RX，115200 baud，优先级 2:
  - 摄像头上位机接收字节进入环形缓冲
  - 二进制组帧、CRC、接收次数统计和菜单显示放在主循环，不在中断内执行
```

当前小球平衡阶段不使用IMU。`CMakeLists.txt`的`APP_ENABLE_IMU`默认OFF，非删除式
排除`app_attitude.c`、`imu963ra_attitude.c`和完整`app_angle.c`；主程序不初始化
IMU，100 Hz中断不调度姿态或角度环，菜单不注册Attitude/Angle入口。历史串口ABI
由轻量`app_angle_stub.c`占位，最终map中不得出现app_attitude、imu963ra_attitude
或FusionAhrs符号。需要恢复时不能只打开CMake选项，还必须恢复初始化、调度和菜单入口。

移动实施规则：

```text
1. 中断资源绑定写在 project/code/core/app_interrupts.c。
2. 原模块只暴露轻量的 tick/update/event 函数。
3. 如果任务耗时不可控，改为中断置标志，主循环消费。
4. 如果任务含硬件通讯，优先不要放中断；确需放入时必须说明风险。
5. 修改后更新本文件的当前中断任务表。
```

## 8. 新增模块功能流程

新增模块前，agent 应先判断模块类别：

```text
core:
  算法、控制、调度、参数、状态机、Flash 参数缓存等。

device:
  传感器、执行器、显示、按键、编码器、电机、舵机等硬件封装。

menu:
  页面注册、菜单回调、Easy Menu 适配。
```

新增模块时必须确认：

```text
1. 模块是否需要 init？
2. 模块是否需要周期 tick/update？
3. 周期任务放主循环还是中断？
4. 是否需要非阻塞状态机？
5. 是否需要写入全局黑板 app_blackboard？
6. 是否需要持久化参数？
7. 是否需要菜单配置或显示？
8. 是否涉及硬件通讯，通讯耗时是否会扰动控制周期？
```

推荐模块接口风格：

```c
void app_xxx_init(void);
void app_xxx_set_enabled(uint8 enabled);
uint8 app_xxx_get_enabled(void);
void app_xxx_tick(uint16 elapsed_ms);
void app_xxx_update(uint16 dt_ms);
void app_xxx_get_data(app_xxx_data *data);
```

如果模块需要跨线程/中断与主循环交流：

```text
少量共享状态：
  使用模块内 static volatile + 原子读写。

跨模块控制数据：
  使用 project/code/core/app_blackboard.c/.h。

可保存参数：
  使用 app_param_store，不要模块私自写 Flash。
```

## 8.1 全局参数黑板

全局黑板位于：

```text
project/code/core/app_blackboard.c/.h
```

黑板用于保存跨模块、跨主循环/中断边界共享的运行时数据，例如：

```text
循迹运行标志 tracking_run
左右轮目标速度
左右轮当前速度
灰度二值化数组
灰度循迹误差
差速方向半差值 onto_cps
后续方向环、速度环之间需要共享的实时状态
```

差速方向量采用 `onto_cps`，单位 count/s，各任务均可通过黑板接口写入：

```text
left_final_target_cps  = left_base_target_cps  + onto_cps
right_final_target_cps = right_base_target_cps - onto_cps
```

因此左右轮最终目标速度差为 `2 * onto_cps`。控制模式切换以及任务退出时必须将
`onto_cps` 清零，避免旧方向命令污染后续速度任务。循迹模式不使用 `app_angle`
或舵机方向环，而是在 `app_tracking.c` 内使用独立的 `app_pid_t direction_pd`
作为 PD 载体。方向位置不得只使用二值通道编号，必须融合每通道0..100归一化
`deal`值计算连续重心，误差单位为位置权重×100，范围-700..700。中心
`|error|=1.00`对应5%，正常最外侧`|error|=7.00`对应60%，中间连续插值；
PD输出再按基础速度换算为
`onto_cps`。初始 `Kp=1、Ki=0、Kd=0`，后续只整定P/D。
方向误差采用“左正右负”：物理最左侧至最右侧权重为
`7, 5, 3, 1, -1, -3, -5, -7`。方向 PD 的初始 P/D 参数集中定义在
`app_tracking.h` 的 `APP_TRACKING_DIRECTION_PD_KP` 和
`APP_TRACKING_DIRECTION_PD_KD`，不得通过交换电机左右通道修正方向符号。
二值结果继续负责丢线、停车线和多噪声段选择；存在多个不相邻连续段时，先选
通道数最多的段，同长度再选最接近上次有效位置的段。连续重心应包含选中段左右
各一个归一化邻近通道，使黑线位于探头间隙时仍能提前响应。方向PD默认使用
`Kp=1.0`；Kd仅用于动态变化修正，不能承担稳态最大差速贡献。串口命令`@Onto_P:<value>`和
`@Onto_d:<value>`立即修改运行时方向P/D，当前不写入Flash。

连续灰度误差映射与方向PD之间必须经过物理差速加速度平滑层。当前车型参数为
轮距220 mm、轮胎直径65 mm；半径500 mm圆弧的理论稳态半差速为基础速度的22%，
但22%只用于物理量级校验，禁止把它作为方向输出硬限幅。正常可见黑线最大请求
60%，丢线1.5 s后的紧急搜线保留100%差速。平滑层默认按300 mm/s²增加转向半差速，
按600 mm/s²回正或反向制动，并根据当前规划基础速度换算每个10 ms周期允许变化的
方向百分比。正常循迹还应按连续误差调度基础速度：`|error|<=1.00`保持用户设置，
随后线性下降，最外侧最低保持用户设置值的65%。

当前电机标识为`MG513XP28_12`。可查到的MG513霍尔版资料给出电机轴13 PPR，
型号中的P28按28:1减速比理解，因此理论输出轴为364 control count/rev；由于未找到
完整后缀型号的直接原厂数据表，最终值必须通过`Motor -> Encoder Rev Sample`
实测确认：KEY1清零后手动旋转一个轮胎恰好一圈，读取对应E1/E2累计绝对值。

Tracking Run 当前约定：

```text
基础速度：由具体Answer赛题页面设置；Question 2默认35%，Question 4默认23%，范围15..40%，步长1%
目标圈数：由Question 2的`Laps`设置，范围1..99，每次发车将当前圈数清零
停车线：灰度二值黑线有效通道数达到`Initialize -> Finish channels`时圈数+1；
阈值范围2..8、默认3、步进1；计数后冷却3 s，并要求离开停车线后才能再次触发。
该阈值只影响停车线/圈数判定，不得修改灰度方向误差的相邻连续段平均算法；
两个相邻通道有效时仍按两者权重平均值参与循迹。
目标圈数完成：立即停车并留在Question 2 Run页面显示最终用时；若发车时已经压在停车线上，必须先驶离再进入才计圈
Question 4 / Sample A-B：KEY1第一次启动循迹并记录左右累计编码器起点，第二次KEY1停车；
记录值为倍频补偿后左右轮绝对行程的等权平均control count
Question 5/6缓刹期间继续运行灰度方向环，但差速贡献限制为20%，避免低速剧烈拐弯；
两轮平均实际速度降到编码器极限速度的5%以下后才将onto清零并停止方向调节。
Question 2/5/6另有独立里程安全锁：发车时记录左右编码器起点，倍频补偿后的
左右绝对行程平均值达到`目标圈数 × Initialize/Lap safety count`时，即使停车线
漏检也触发公共缓刹流程；默认10000 control count/圈、步进50、范围
1000..30000。Question 2正常检测到停车线时仍急停，只有里程兜底触发时软刹；
该兜底不作用于Question 4和Challenge。
Question 4 / Run A-B：KEY1从当前位置启动循迹，100 Hz任务实时计算左右轮平均行程；
达到RAM中的采样记录后触发公共缓刹并留在当前页面显示最终用时，未采样时拒绝启动
Question 4 / Run A-B采用两次确认：第一次KEY1启动Ball Zero零点维持，第二次KEY1才启动
定距循迹与计时。目标行程只作为缓刹触发点，允许停车行程存在较大正偏差；
触发后复用Q5/Q6的公共缓刹函数，方向差速仍由受限灰度方向环计算；
触发时冻结计时，车轮实际停止后发布完成状态，但持续维持小球零点。
KEY4退出时必须依次停止定距任务、停止Ball Zero、释放步进owner，并调用
app_zdt_stepper_return_zero_on_exit()启动非阻塞回零；回零期间由busy状态锁定菜单按键。
Question 4/5/6和Challenge统一调用`app_tracking_soft_stop_isr_update()`完成缓刹：
以Task accel cps2把左右规划目标斜坡降到0；实际速度较高时继续以20%差速上限循迹，
低于极限速度5%后撤销方向量，规划目标为0且两轮反馈均≤30 count/s才结束速度环。
各题只负责以定距、圈数停车线或里程安全锁决定何时触发；默认斜率100 count/s²。
第二问不调用公共缓刹函数，仍使用普通2000 count/s²启动斜率和原速度PID急停。
Answer列表首项Initialize提供Stepper Zero Cal和Task accel cps2：前者复用中心校准页，
进入即解锁CALIBRATION权限，可点动后KEY1把当前位置设为逻辑零点；后者范围10..2000、
步进30，只作用于Q4/Q5/Q6，不作用于第二问。
Question 5/6不再使用编码器里程目标，改用各自独立的Laps菜单，范围1..99、默认1；
目标圈数停车线触发后冻结计时，并按Task accel cps2斜率把速度目标软降到零。Question 5
默认速度26%。Question 6默认速度26%，并新增Position cm目标位置，范围-10.0..+10.0 cm、
步进0.1 cm。Q6第一次KEY1
调用app_balance_start_ball_target()维持菜单目标位置，第二次KEY1启动定距循迹与计时；
停车后继续维持该目标，KEY4执行与Q4/Q5相同的停止、释放owner和受保护回零。
距离采样与定距运行期间不执行停车线圈数判定；记录当前不上Flash，重启后需重新采样
KEY1：发车
KEY4：停车并返回当前赛题的上级页面
UART @RUNING\r\n：发车
UART @STOP\r\n：停车
运行期间关闭 IPS 硬件刷新
丢线 1.5 s：沿最后有效方向提升到 100% onto，一侧停车、一侧转动
丢线 9 s：停车并由主循环返回 Tracking 页面
```

左右轮速度数据采用反馈区与命令区分离：

```text
轮速反馈区（count/s）：
  只有 app_encoder_update() 通过 app_blackboard_encoder_publish_wheel_speed() 写入。
  其他任务只能通过 app_blackboard_get_wheel_speed_feedback() 读取。

电机速度命令区（-100..100 percent）：
  PID、菜单、速度规划等任务通过 app_blackboard_set_wheel_speed_command() 写入。
  10 ms / 100 Hz 中断末端读取该区，并统一调用 app_motor_set_wheel_speed_percent()。
  10 ms 中断无论控制模式为 0/1/2 都必须下发该命令区，不得在模式 0 分支持续覆盖为零。
  模式 0 表示没有 PID 控制器生产命令，Motor Control 等显式手动任务仍可临时写入命令区。
  每个任务进入时应从零命令取得使用权，退出时必须把左右命令恢复为零。
  普通业务不得绕过黑板直接调用电机速度设置接口。
  同一调度周期存在多个写入者时，最后一次写入生效；新增规划器时必须明确写入优先级或所有权。
```

速度目标百分比映射：

```text
app_tracking 初始化时必须传入编码器极限速度 max_speed_cps。
当前使用 APP_ENCODER_LIMIT_SPEED_CPS = 2100 count/s。
-100..100% 线性映射到 -max_speed_cps..max_speed_cps。
PID 内部目标与反馈统一使用 count/s，PID 输出再限制为 -100..100% 电机命令。
编码器采样周期变化时仍使用 count/s，无需修改百分比映射；只需保证编码器按实际 dt 换算速度。
编码器速度获取阶段按确定的倍频机制统一数量级：右轮 E1 硬件 QEI 为四倍频，
左轮 E2 软件解码为单倍频，因此右轮发布黑板前严格除以 4。
2100 count/s 只是低倍率侧的粗略极限参考值，不得用于反推倍频补偿系数。
方向符号以闭环实测为准：2号小车当前通过交换两路电机各自的正反PWM，使正命令
对应车辆前进；零目标联动测试完成后必须确认左右轮正命令均产生正编码器反馈。
每周期 delta 仍保留硬件原始计数，供诊断倍频和丢脉冲使用。
```

轮胎速度PID使用“业务请求目标”和“规划后目标”两级。`left_target_cps/right_target_cps`
保存业务请求，100 Hz控制周期按`2000 count/s²`斜率生成规划目标，速度PID只读取规划
目标；失能、任务退出和安全停车必须绕过斜坡立即清零。循迹方向半差值在规划后的基础
速度上混合，避免基础速度阶跃造成硬速度环过载。

当前2号小车PID默认参数：

```text
左右速度环：Kp=0.374，Ki=0.31，Kd=0.001。
Angle惯性方向环：Kp=0.075，Ki=0，Kd=0.0012。
Tracking PID Flash记录版本为4；旧版本记录启动时回退到上述速度环默认值。
Angle惯性方向环当前不写Flash，每次启动使用代码默认值。
```

小球LADRC正负5 cm往返的实机最优回退基线记录在
`docs/ball_balance_usage.md`。当前基线为`wc=3.5、wo=10、b0=-0.2、
tracking=0、alpha=0.65、beta=0.12、双向预测0.23 s、预测补偿±2.5 cm、
2500 RPM`。在正端低速稳定后折返的状态机下，约`4.98 s`进入负端稳定并
满足5秒判据，额外连续保持1秒后约`6.07 s`结束。后续调参必须区分“候选
实验值”和“已验证最优值”；候选失败时恢复该基线。

`Ball Zero Hold`用于小车移动过程中的零点保持，必须与正负5 cm最终保持共用
持续在线的LADRC抗扰策略：稳定后不进入idle、不清空ESO；二者共用0.23 s预测、
滑轨位置环Kp=0.30和2500 RPM速度余量。退出Ball Zero或往返任务时恢复普通
滑轨配置并安全锁定。

黑板的定位：

```text
app_blackboard:
  运行时共享状态。
  可以被中断和主循环共同访问。
  不直接负责 Flash 保存。

app_param_store:
  可掉电保存参数缓存。
  初始化时从 Flash 读取。
  用户确认保存时写入 Flash。

菜单页面临时变量:
  只服务当前菜单编辑过程。
  用户未 Apply/Save 前不应影响 Flash。
```

使用规则：

```text
1. 只有确实需要跨模块共享的运行时数据才放入黑板。
2. 单模块内部状态优先使用模块内 static 变量，不要滥用黑板。
3. 中断和主循环都会访问的数据，读写时要考虑原子性；必要时使用 interrupt_global_disable() 保护。
4. 黑板字段应在 app_blackboard.h 中用行尾注释写明单位、方向约定和读写方。
5. 黑板不是持久化存储，掉电后应由模块 init 或 app_param_store 重新初始化。
6. 不要把菜单临时编辑值直接长期塞进黑板；菜单应在确认后调用业务模块接口应用。
```

新增黑板字段前，agent 应先确认：

```text
1. 这个数据是否真的需要多个模块访问？
2. 是否会被中断读写？
3. 数据单位是什么？
4. 数据方向/符号约定是什么？
5. 谁负责写入，谁负责读取？
6. 是否需要掉电保存？若需要，黑板只保存运行时副本，Flash 仍走 app_param_store。
```

## 9. 将模块功能融入菜单流程

当某模块需要加入菜单时，agent 必须先询问或确认：

```text
1. 菜单只显示状态，还是允许修改参数？
2. 参数修改是否立即应用，还是 Apply 后应用？
3. 参数是否需要掉电保存？
4. 是否需要进入中断调度？
5. 如果需要中断，周期和优先级是多少？
6. 如果有解算任务，采样周期、解算周期、显示周期分别是多少？
7. 如果模块存在硬件通讯限制，是否应避免在中断中通讯？
8. 页面内容是否超过一屏，是否需要滚动或折叠？
9. 是否需要运行时关闭屏幕刷新，避免显示通讯扰动？
```

建议分层：

```text
业务模块：
  提供 app_xxx_set/get/apply/save 等接口。

Easy_Menu_User.c：
  只负责页面、输入解释、调用业务接口。

app_param_store.c：
  负责可保存参数缓存和 Flash 写入。

app_interrupts.c：
  负责周期调度或中断事件绑定。
```

不建议：

```text
菜单回调里直接写复杂控制逻辑。
菜单回调里直接写 Flash。
菜单回调里直接操作底层 IPS 屏。
菜单回调里临时绑定中断。
```

## 9.1 步进电机统一控制层

步进电机业务控制统一经过`core/app_stepper_control.c/.h`：

```text
上层任务：
  申请owner锁
  写入POSITION或VELOCITY目标
  不维护自己的滑轨位置PID
        ↓
app_stepper_control_update():
  唯一滑轨位置PID
  反馈有效性检查
  速度限幅
  写入ZDT设备速度命令区
        ↓
app_zdt_stepper_tick():
  唯一Emm协议周期发送点
```

`app_balance`只计算小球外环产生的目标滑轨角，不得重新增加滑轨位置PID。
普通菜单测试和串口诊断只允许通过`app_stepper_control_set_velocity/position()`
写目标。`app_zdt_stepper_move_position/jog/reset_current_position`仅保留给零点校准、
驱动器回零和协议级维护操作，不得被普通业务任务直接调用。释放owner时必须零速
并安全锁定；不同owner不能抢占，切换任务前由原owner明确归还。

主循环调度顺序必须保持：

```text
上层模块更新目标
  -> app_stepper_control_update()
  -> app_zdt_stepper_tick()
```

## 10. Flash 与可保存参数业务流程

Flash 写入只允许发生在用户明确确认的配置变更中，例如 `Apply+Save` 菜单项。不要在周期循环或中断中写 Flash。

当前 Flash 参数区统一保留在 MSPM0G3507 主 Flash 尾部 3 个 1 KiB 页：

```text
程序链接区：0x00000..0x1F3FF
Tracking PID：0x1F400..0x1F7FF（section 18 / page 1）
GS08RA 校准：0x1F800..0x1FBFF（section 19 / page 0）
系统配置：0x1FC00..0x1FFFF（section 19 / page 1）
```

`cmake/mspm0g3507.ld` 必须将程序 FLASH 长度限制为 `0x1F400`。代码增长进入
参数区时必须由链接器报错，禁止为了通过链接而恢复为整片 `0x20000`。
`app_flash_store` 会在运行时拒绝参数保留区以外的读写。新增参数记录时必须先
分配独立擦除页；不得与现有记录共页，也不得直接使用历史地址
`0x18000/0x18800/0x18C00`，这些地址属于当前固件代码区。

统一入口：

```text
project/code/core/app_param_store.c/.h
project/code/core/app_flash_store.c/.h
```

职责分层：

```text
app_flash_store.c/.h
  低层 Flash 记录工具。
  负责 magic、version、checksum、erase、write、read 等通用细节。
  不直接理解业务参数含义。

app_param_store.c/.h
  业务参数持久化中枢。
  负责把系统配置、灰度校准、Tracking PID 等业务参数缓存成结构体。
  负责初始化时从 Flash 读出参数。
  负责用户确认保存时调用 app_flash_store 写入 Flash。

业务模块，例如 app_tracking.c / app_gs08ra.c / app_config.c
  负责运行时参数应用、控制器状态更新和 get/set 接口。
  不应直接 erase/write Flash。

菜单页面，例如 Easy_Menu_User.c
  负责编辑临时副本、响应按键、触发 Apply/Save。
  不应直接操作 Flash 页。
```

判断某个参数是否需要进入 Flash 前，agent 必须先确认：

```text
1. 该参数是否需要掉电保存？
2. 参数属于哪个业务模块？
3. 参数是否已有 app_param_store 区域？
4. 是否可以复用现有 Flash 页和记录格式？
5. 是否需要版本号迁移？
6. 保存动作是否由用户明确触发？
7. 保存失败后菜单需要如何提示？
8. 参数初始化默认值是什么？
9. Flash 读取失败、checksum 错误或版本不匹配时是否回退默认值？
```

标准初始化流程：

```text
系统初始化：
  app_param_store_init() 从 Flash 读取参数
  app_param_store 内部校验 magic/version/checksum
  校验失败则加载默认值
  各业务模块 init 时从 app_param_store 获取参数并应用
  禁止模块 init 时自行读写 Flash
```

标准菜单编辑与保存流程：

菜单编辑：
  1. 进入页面时，从业务模块或 app_param_store 读取当前参数到临时副本。
  2. 用户通过菜单按键修改临时副本。
  3. 修改过程中不写 Flash。
  4. 用户点击 Apply：
       调用业务模块 runtime apply 接口，只更新运行时行为。
  5. 用户点击 Apply+Save：
       调用业务模块 runtime apply 接口。
       调用 app_param_store_set_xxx() 更新参数缓存。
       调用 app_param_store_save_xxx() 写入 Flash。
  6. 保存成功或失败后，通过菜单状态显示结果。
```

业务模块不应直接读写 Flash 页。

如果新增一类可保存参数，实施步骤：

```text
1. 在 app_param_store.h 中定义业务参数结构或 set/get/save 接口。
2. 在 app_param_store.c 中定义默认值。
3. 选择 Flash section/page 或 app_flash_store record 位置。
4. 增加 load 逻辑：读取、校验、版本判断、失败回退默认值。
5. 增加 set/get 逻辑：只操作 RAM 缓存。
6. 增加 save 逻辑：由明确用户动作触发，调用 app_flash_store。
7. 修改业务模块 init：从 app_param_store 获取参数并应用。
8. 修改菜单：编辑临时副本，Apply 时应用，Apply+Save 时保存。
9. 编译并上板验证读取、修改、保存、重启加载。
10. 更新本文件中的 Flash 页/参数归属说明。
```

Flash 禁止事项：

```text
禁止在中断中写 Flash。
禁止在周期任务中自动写 Flash。
禁止每次参数微调都立即写 Flash。
禁止业务模块私自定义 magic/version/checksum 并直接写页。
禁止菜单绕过 app_param_store 直接调用 app_flash_store 或底层 flash 驱动。
```

当前原则：

```text
运行时参数归业务模块。
持久化缓存归 app_param_store。
Flash 格式与校验归 app_flash_store/app_param_store。
保存动作归用户确认。
```

## 11. 当前关键约定

### 灰度通道方向

GS08RA 的数组及屏幕通道按从左到右的下标 `0..7` 使用：

```text
物理最左侧灰度探头 -> 下标 0 -> 屏幕第一个通道
物理最右侧灰度探头 -> 下标 7 -> 屏幕最后一个通道
```

循迹位置误差及方向环必须遵守这一符号约定。不得仅为适配控制符号而私自在显示层倒序通道。

### 舵机方向

当前实车舵机控制关系：

```text
250  -> 前轮最右，车辆右转
700  -> 前轮中位，车辆直行
1150 -> 前轮最左，车辆左转
```

归一化转向接口：

```c
app_servo_set_steer(int8 steer);
```

`steer` 是无量纲转向指令，不是角度：

```text
steer < 0 右转
steer = 0 直行
steer > 0 左转

-100 -> 最右
0    -> 中位
100  -> 最左
```

舵机 API 当前保留供独立测试或未来车型使用。当前三轮差速循迹不得把方向环输出
写入舵机；方向 PD 输出 `-100..100%` 的差速比例，并由 `app_tracking.c`
换算成黑板 `onto_cps`。

### B3/B4 执行器接口释放

舵机和电磁铁当前均不启用，菜单中不注册对应测试页。启动流程保留舵机软件 API
状态初始化，但不初始化 B3 PWM；不再调用 `app_electromagnet_init()`，因此 B4
也不被 PWM 外设占用。后续可把 B3/B4 作为通讯或普通 GPIO 使用，但在启用新复用
功能前仍需确认没有业务代码重新调用电磁铁初始化。

当前已核对的通讯复用关系：

```text
B3：UART3_RX / I2C1_SDA
B4：UART1_TX，无硬件 I2C
B8：UART1_CTS，无 UART TX/RX 或硬件 I2C
B14：无 UART 或硬件 I2C
B26：UART0_RTS，无 UART TX/RX 或硬件 I2C
```

B3+B4、B8+B14+B26 均不能直接组成完整硬件 UART 或硬件 I2C。任意两个空闲 GPIO
可用于软件 I2C；若需要硬件外设，优先使用 B2+B3：UART3 为 B2-TX/B3-RX，
I2C1 为 B2-SCL/B3-SDA。

## 12. 常见禁止事项

- 不要在业务模块中分散 Flash 读写。
- 不要在中断中执行阻塞、Flash 写入、菜单绘制或大量串口输出。
- 不要在 Show Page 周期回调中反复整屏清空。
- 不要绕过 `app_display_flush_dirty()` 直接操作 IPS 屏。
- 不要在 `app_menu.c` 中添加具体页面特判。
- 不要在 Easy Menu 框架源码中硬编码业务页面。
- 不要在模块初始化阶段提前启用应用层中断。
- 新增页面若由菜单导入脚本生成，应同步修改脚本模板，避免生成结果覆盖这些规则。

## 13. 提交与推送规范

提交和推送前，必须使用 Conventional Commits 风格的中文说明，便于追踪云端历史记录：

```text
feat: 增加左轮编码器 x4 解码
fix: 修正姿态反馈方向
docs: 补充硬件资源表
refactor: 拆分菜单与业务层
```

要求：

- 类型使用英文小写，说明使用中文。
- 一次提交只做一类事情，不要把无关改动混在一起。
- 推送到远端前，提交信息必须能直接说明这次改动的核心内容。
- 如涉及多人协作，推送前先确认本地分支基于正确的目标分支。

---
> Source: [colorcard/26NUEDC-H](https://github.com/colorcard/26NUEDC-H) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
