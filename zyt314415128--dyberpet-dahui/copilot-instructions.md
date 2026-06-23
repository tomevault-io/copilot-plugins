## dyberpet-dahui

> 将家里养的猫变成"数字生命"——一只运行在 Windows 桌面上的模拟养成桌面宠物。

# 桌面宠物 — 数字猫打灰

## 项目概述
将家里养的猫变成"数字生命"——一只运行在 Windows 桌面上的模拟养成桌面宠物。

## 核心设计决策

### 产品形态
- **类型**：模拟养成型桌面宠物（非对话伴侣、非纯观赏）
- **互动**：全面养成——饥饿值、心情值、亲密度、喂食、玩耍、铲屎
- **行为**：脚本化随机行为（站立、行走等概率触发）
- **语言交互**：猫不说话，不接入对话/语音功能

### 视觉风格
- **风格**：Disney/皮克斯风格，偏真实感，有猫毛细节，非简洁卡通风
- **形式**：2D 精灵图动画（Sprite Sheet）
- **素材管线**：AI 视频生成 → 色度键抠图（绿幕去背）→ 等比缩放部署

### 技术栈
- **壳/框架**：DyberPet（Python + PySide6），作为桌面渲染和养成系统底座
- **数据存储**：JSON（`dyberpet/data/` 下的 `pet_data.json`、`settings.json`、`act_data.json`、`task_data.json`）
- **目标平台**：Windows（唯一）

### 开发方式
- AI 辅助开发（Claude Code 写代码，用户做决策和测试）
- 参照 DyberPet 成熟项目的架构，在其基础上改造

## 目录结构约定
```
桌面宠物/
├── CLAUDE.md                  # 本文件 — 项目规范
├── .gitignore
├── dyberpet/                  # DyberPet 壳代码（上游 v0.8.5）
│   ├── run_DyberPet.py        #   启动入口 + 信号总线
│   ├── DyberPet/              #   PySide6 主逻辑
│   │   ├── DyberPet.py        #     PetWidget 主窗口 + 鼠标交互
│   │   ├── modules.py         #     三线程 Worker（Animation/Interaction/Scheduler）
│   │   ├── settings.py        #     全局状态 + 配置持久化
│   │   ├── conf.py            #     PetConfig/ActData/PetData/ItemData/TaskData
│   │   ├── utils.py           #     工具函数 + SubPet_Manager
│   │   ├── Accessory.py       #     饰品/掉落物/子宠物系统
│   │   ├── Notification.py    #     通知（Toaster）+ 气泡（BubbleText）
│   │   ├── bubbleManager.py   #     气泡行为逻辑层
│   │   ├── Dashboard/         #     仪表盘（状态/背包/商店/任务）
│   │   └── DyberSettings/     #     设置面板（基本设置/存档/角色卡/物品MOD）
│   ├── res/role/打灰/          #   奶牛猫打灰（→ assets/configs/ 是配置源）
│   └── res/role/花椒/          #   白猫花椒（像素占位，仅 stand 动画，待正式素材替换）
├── assets/                    # 素材与配置（唯一编辑源）
│   └── configs/               #   pet_conf.json + act_conf.json
├── tools/                     # 素材处理工具
│   ├── frame_extractor.py     #   视频帧提取（视频→PNG 序列）
│   ├── deploy_sprites.py      #   精灵图部署（色度键抠图 + resize 到 640×360）
│   ├── copy_configs.py        #   配置文件批量部署
│   ├── adjust_floor.py        #   地面位置可视化调节器
│   ├── cut_pixel_sprites.py   #   像素精灵图表格切割为单独帧
│   ├── map_pixel_frames.py    #   像素帧映射到项目动画命名格式
│   ├── cat-size-preview.html  #   猫尺寸可视化预览
│   └── frame-layout.html      #   1920×1080 参考帧布局
├── data/                      # 运行时数据（gitignore）
└── docs/                      # 设计文档
    ├── design.md              #   完整设计规格
    ├── decisions/             #   架构决策记录
    └── superpowers/           #   AI 工作产物
        ├── specs/             #     设计规格
        └── plans/             #     实现计划
```

## 命名约定
- Python 文件：snake_case
- 类名：PascalCase
- 配置 JSON：camelCase（与 DyberPet 保持一致）
- 精灵图文件：snake_case，动作名_帧数.png（如 walk_8f.png）
- **配置同步**：`assets/configs/` 是角色配置的**唯一编辑源**。修改后运行 `python tools/copy_configs.py` 部署到 `dyberpet/res/role/打灰/`。不要直接改 dyberpet 下的配置

## 开发原则
- 先在 DyberPet 原版跑通，再开始改
- 每次改完必须验证：猫能显示、能互动、数据能存
- 素材替换和代码改造分开做，不要混在一起提交
- **deploy_sprites.py 不覆盖已有 act_conf.json 配置**：只补充新动作 key，已有的 `frame_start`/`frame_end`/`move_phases`/`frame_refresh` 等设置必须保留不动

## 架构概览

### 三线程 + 信号驱动

应用采用三个 `QThread` Worker 并行运行，通过 Qt 信号通信：

| Worker | 职责 | 关键信号 |
|--------|------|---------|
| `Animation_worker` | 随机待机动画、HP/FV 概率系统、半屏方向过滤 | `sig_setimg_anim`, `sig_move_anim`, `sig_start_sleep` |
| `Interaction_worker` | 交互动画（拖拽/落地/睡觉/启动/喂食/摸头） | `sig_setimg_inter`, `sig_bounce_move`, `sig_act_finished` |
| `Scheduler_worker` | HP/FV 衰减、番茄钟/专注计时、物品掉落、自然唤醒 | `sig_change_hp`, `sig_change_fv`, `sig_wake_sche` |

**状态协调**：`PetWidget` 持有三个 Worker 引用。交互发生时暂停 Animation_worker，Interaction_worker 完成后调用 `resume_animation()` 恢复。

### 启动流程
`run_DyberPet.py` → `DyberPetApp` → `PetWidget`（`_init_ui` → `_init_widget` → `init_conf` → `runAnimation` → `runInteraction` → `runScheduler`）→ `DPNote` → `DPAccessory` → `__connectSignalToSlot`

### 数据持久化层（conf.py）

| 类 | 存储文件 | 职责 |
|----|---------|------|
| `PetData` | `data/pet_data.json` | HP、FV、coins、items、陪伴天数 |
| `ActData` | `data/act_data.json` | 动画解锁状态、概率、播放列表 |
| `TaskData` | `data/task_data.json` | 每日任务历史、目标、连续天数 |
| `settings` | `data/settings.json` | 用户偏好（重力/音量/缩放/语言等） |
| `ItemData` | 运行时加载 | 物品配置（扫描 `res/items/` + `res/pet/` + `res/role/{name}/items/`） |
| `PetConfig` | 运行时加载 | 角色配置（从 `pet_conf.json` + `act_conf.json` 构建） |

**懒加载**：`LazyAct` 类延迟加载精灵图，启动时只加载 stand 第一帧 + `fallasleep_wake` 全量，其余动作在后台线程 100ms 后逐个加载。

### 仪表盘系统（Dashboard）
`DashboardMainWindow(FluentWindow)`：状态（HP/FV/Buff）、背包（消耗品/收藏品）、商店（购买/出售，折旧率 75%）、任务（番茄钟/专注计时）。背包是中心枢纽。

### 物品系统
`res/items/Default/` 包含 `items_config.json`（21 个物品）。按 fv_lock 分 Tier 0-5，高层食物更强力。Buff 类型：`HP_stop`（冻结饥饿衰减）、`coin`（周期金币）。掉落概率：基础 8%，每级 FV +2%，上限 25%。

### 喂食动画系统
每个食物有专属喂食动画（`feed_{物品名}`）.
- **动画播放规则**：喂食动画播放期间，继续喂食不打断当前动画，也不补播；只实时产生效果（HP/FV 增加）
- **无专属动画的食物**：播放 stand 作为 fallback
- **素材管线**：绿幕视频 → frame_extractor.py 提帧 → sam2_mask.py 抠图 → deploy_sprites.py 部署
- **配置**：`act_conf.json` 中添加 `feed_{物品名}` 配置，`deploy_sprites.py` 的 `SLEEPActions` 列表中注册

### 金币系统
获取：点击猫（高斯分布）、物品掉落、任务奖励（单任务 200，五任务 1500）。消耗：商店购买。出售：折旧 75%。

### HP/FV 衰减与降级
- **HP 衰减**：每分钟 -1。Tiers：`[0, 20, 70, 100]` → Starving/Hungry/Normal/Energetic
- **FV 衰减**：HP tier 0 时每分钟 -5，其余不衰减
- **FV 降级**：FV 降到 0 后继续扣减，触发 `_level_down` 降至上一级满值（如 lv4→lv3 满值 120）
- **FV 升级**：每分钟 +1，满值后自动升级

### 通知系统
`DyberToaster`：右下角弹窗，合并同类，5 秒消失。`BubbleText`：猫头顶气泡，countdown 类型鼠标靠近淡出。`BubbleManager` 按 HP tier 调度气泡（tier 0 → `fv_drop`/`hp_zero`/`feed_required`；tier 1-2 → `hp_low`/`feed_required`）。隐身模式下所有气泡静默。

## 鼠标交互规则

### 状态机概览
```
normal ──press──→ drag_start ──move──→ grab_loop ──release──→ fall ──帧结束──→ land_bounce ──帧结束──→ land_stay ──帧结束──→ normal
  │                 ↑                                                                                                    ↑
  └──click(no move)─┘                                                                                                    │
       → patpat() (25% 概率触发动画池)                                                                                    │
                                                                                                                          │
  sleep/startup/entertainment: 只允许 X 轴拖动 ───────────────────────────────────────────────────────────────────────────┘
```

### 各状态交互
- **drag_start → drag（grab_loop）**：正常抓起拖拽，drag_start 播一次后切到 grab_loop 循环
- **fall**：松手坠落中，不响应鼠标，播完停在最后一帧
- **land_bounce**：动画播放期间不可打断，禁止拖拽（弹跳有自己移动逻辑），点击可触发金币和爱心
- **land_stay**：动画播放期间不可打断，只能 X 轴拖动，不能抓起，点击可触发金币和爱心
- **land_stay 播完**：恢复正常交互

### 速度追踪
`mouseReleaseEvent` 维护 4 帧滑动窗口（`mouseposx1-4`, `mouseposy1-4`），释放时计算抛体初速度，驱动 `fall` 动画的物理轨迹。

## 自娱自乐动画点击触发
- **触发**：单击猫有概率触发动画池动画（sneeze/groom/flycatch/tailshake/roll/jump 随机）
- **动画池**：6 个动画，根据亲密度解锁：sneeze fv0+；groom/flycatch fv1+；tailshake fv2+；roll/jump fv3+
- **播放期间锁定**：不响应新触发、不可拎起（只允许 X 轴拖拽）、金币和爱心仍正常
- **解锁**：动画播完（`sig_act_finished`）后自动解锁

## 睡觉交互规则
- **触发**：stand 循环中随机触发，或按 S 键手动触发
- **动画**：fallasleep_onset → fallasleep_loop（循环）→ 被唤醒时播放 fallasleep_wake
- **唤醒**：自然唤醒（每分钟 20% 概率，期望 ~5 分钟）或右键菜单"唤醒打灰"
- **鼠标**：左键点击无反应，左键按住只能 X 轴移动，右键只显示"唤醒打灰"菜单

## 启动动画（Startup Wake）
- **动画**：启动时立即播放 fallasleep_wake（169 帧，~6.76 秒）
- **加载策略**：只加载 stand 第一帧用于首帧显示 + fallasleep_wake 全量；stand 保持 `_loaded=False`，Animation_worker 播放时才触发真正的全量加载；其余动作由后台线程 100ms 后逐个加载
- **锁定**：`settings.is_starting_up` 为 True 时，左键只允许 X 轴拖拽，动画不可切换，右键菜单正常显示
- **结束**：动画播完 → `stop_interact()` → `resume_animation()` → Animation_worker 恢复正常随机动画

## 动画解锁系统
- **解锁条件**：只看 fv_level（亲密度），不再看饱食度 tier。`act_type` 在 pet_conf.json 中为单数字（fv_level 门槛）
- **亲密度门槛（fv_level）**：控制动画是否解锁，每分钟 +1 FV
- **权重机制**：所有非站立动画权重相等（act_prob=0.1），站立权重 1.0。概率按 `act_prob / 总权重` 归一化
- **打灰动画门槛**：站立/睡觉/打喷嚏 fv 0+；摇尾巴 fv 1+；舔毛/抓苍蝇 fv 2+；行走/打滚/跳跃 fv 3+
- **数据存储**：`dyberpet/data/act_data.json`，每次亲密度变化时 `_check_fvlock()` 重新判定
- **stand 循环**：stand 播完后攒次数（`STAND_THRESHOLD=3`），每攒够 3 次才触发一次 `random_act()`，70% 继续 stand，30% 换动作
- **nonDefault_prob**：固定 0.35，不再按 tier 分档
- **亲密度升级弹窗**：亲密度升级时弹窗显示"好感度升级啦，已解锁更多动画！"

## 隐身模式（Click-Through）
- **开启**：右键菜单「隐身模式」
- **行为**：鼠标移到猫轮廓上 → 猫消失 + 鼠标穿透到桌面；鼠标离开 → 猫恢复；「恢复」按钮仅在猫消失时出现，点击退出穿透模式
- **技术**：`WS_EX_TRANSPARENT`（Windows API）+ `Qt.WA_TransparentForMouseEvents`，`ctypes` 无额外依赖
- **配置**：`pet_conf.json` 的 `restore_btn_offset`（按钮位置，通过游戏内拖拽调整）
- **恢复按钮**：48×26 圆角按钮，镜像位置以猫窗口中线为对称轴自动计算，超出屏幕时钳制到边界内
- **检测区域**：默认 label + 按钮 + 3px 缓冲带；支持多矩形模式（`active_rects.json`，代码编辑器已实现，菜单已隐藏）
- **像素级检测**：alpha > 30 判定为猫轮廓

## 气泡行为（BubbleText）
- **countdown 类型气泡**（如 feed_required）：鼠标靠近淡出隐藏（200ms），鼠标离开淡入恢复
- **实现**：`_mouse_check_timer`（50ms）驱动 `_check_mouse_position`，检测鼠标是否在气泡区域内
- **鼠标穿透**：隐藏时设置 `Qt.WA_TransparentForMouseEvents`，允许点击穿透到下层
- **配置**：`bubble_conf.json` 的 `countdown` 字段（秒），feed_required 默认 60 秒
- **代码位置**：`Notification.py` 的 `BubbleText` 类

## 便便交互
- **hover 放大**：鼠标移入便便 → 延迟 100ms → 从 80px 逐像素放大到 96px，底部中心点固定向上发散
- **点击消除**：左键点击便便 → 200ms 淡出动画 → 发射 `poop_clicked` 信号 → 关闭窗口
- **实现**：`Poop.py` 的 `PoopWidget`，手动 QTimer 逐像素递增（不用 QPropertyAnimation，避免 round() 抖动）

## 行走动画规则
- **变速移动**：left_walk/right_walk 支持 `move_phases` 分阶段变速（静止→加速→匀速→减速→静止），配置在 `act_conf.json`，代码在 `modules.py` 的 `_get_frame_speed`
- **半屏方向限制**：stand 状态触发行走时，猫窗口中心在左半屏只向右走，在右半屏只向左走。已在走的动作不受影响，会完整走完。配置在 `settings.py`（`pet_center_x`/`screen_width`），过滤逻辑在 `modules.py` 的 `random_act` 和 `DyberPet.py` 的 `keyPressEvent`
- **键盘测试**：A/D 向左/右走（受半屏限制），S 睡觉，空格 跳跃

## 多屏幕支持
- **拖拽跨屏**：释放时检测精灵窗口与各屏幕的重叠面积比，超过 50% 则调用 `switch_screen`
- **floor_y_offset**：每个屏幕独立计算任务栏高度，切换屏幕时自动更新
- **屏幕边界**：`limit_in_screen()` 将窗口坐标限制在当前屏幕可用区域内

## 运行命令速查
```powershell
# 激活环境并启动
cd "D:\Claude专用\桌面宠物\dyberpet"
D:\Python\python.exe run_DyberPet.py

# 素材处理管线（两步）
D:\Python\python.exe tools/frame_extractor.py video.mp4 frames/   # 1. 帧提取
D:\Python\python.exe tools/deploy_sprites.py                       # 2. 色度键抠图 + 部署

# 地面位置调节（可视化脚本）
D:\Python\python.exe tools/adjust_floor.py

# PyInstaller 打包（排除不需要的大型库）
# --contents-directory . 让资源文件直接放在 exe 同级目录，避免 _internal 层级导致图标路径问题
D:\Python\python.exe -m PyInstaller --noconfirm --distpath "D:/桌面宠物打包" --name "我叫打灰" --icon "res/icons/icon.png" --add-data "res;res" --contents-directory . --hidden-import "pynput.mouse._win32" --hidden-import "pynput.keyboard._win32" --hidden-import "PySide6.QtWidgets" --hidden-import "PySide6.QtCore" --hidden-import "PySide6.QtGui" --hidden-import "qfluentwidgets" --hidden-import "tendo" --hidden-import "sqlite3" --noconsole --exclude-module torch --exclude-module torchvision --exclude-module cv2 --exclude-module scipy --exclude-module transformers --exclude-module onnxruntime --exclude-module llvmlite --exclude-module numba --exclude-module pandas --exclude-module imageio --exclude-module imageio_ffmpeg --exclude-module matplotlib --exclude-module PIL --exclude-module PIL.Image --exclude-module PIL.ImageFilter run_DyberPet.py
```

### 地面位置配置（任务栏自适应）
`settings.taskbar_feet_gap`（默认 9px）+ `compute_floor_offset()` 自动适配任务栏高度。影响：DyberPet.py（4 处）+ Accessory.py（1 处）。

## 已知问题
- **Python 3.12**：使用最新版 PySide6-Fluent-Widgets + pyside6（不锁版本）
- **act_conf.json 必需键**：`stand`/`default`/`up`/`down`/`left`/`right`/`drag`/`drag_start`/`drag_loop`/`fall`/`fall_loop`/`on_floor`/`land_bounce`/`land_stay`/`fallasleep_onset`/`fallasleep_loop`/`fallasleep_wake`。`stand` 是默认回退值。新增交互动作必须同时在 pet_conf.json 的 `random_act` 中注册
- **img_from_act 循环**：`act_num=1` 的动画播完后 `playid` 停在终点，需调用方检测 `playid==prev_playid` 并重置为 0
- **SubPet 功能已禁用**：`DPAccessory.setup_accessory()` 中 `pet`/`subpet` 直接 return

## 深入文档
- `docs/design.md` — 完整设计规格（含素材管线、动画架构、视频生成清单）
- `docs/superpowers/specs/` — 各功能设计规格
- `docs/superpowers/plans/` — 各功能实现计划

---
> Source: [zyt314415128/dyberpet-dahui](https://github.com/zyt314415128/dyberpet-dahui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
