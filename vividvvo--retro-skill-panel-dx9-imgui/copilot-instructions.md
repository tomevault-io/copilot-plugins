## retro-skill-panel-dx9-imgui

> [文件名] hook_game_text_数据查询索引.txt

[文件名] hook_game_text_数据查询索引.txt
[用途] 冒险岛 / Maple 客户端文字接管项目 - 全量查询索引 / 交接总表
[语言] 中文
[状态] 结合本轮对话、上传日志、已确认反编译/断点结果整理；尽量把“已确认 / 已证伪 / 待实现”分开。

============================================================
0. 使用说明 / 查询方法
============================================================
本文件目标：以后需要任何相关数据时，优先在这里全文搜索关键词。

建议搜索关键词：
- 地址：50009C1C / 50009B80 / 5000E640 / 5000E520 / 5000B960 / 50018890 / 004382EA
- 对象：textCtx / drawState / target / FontCache / skillTarget / taskTarget / fullScreenTarget
- 现象：崩溃 / 延迟崩 / 裁剪 / 错位 / 一坨 / 任务框 / 技能列表 / fallback
- 状态：已确认 / 已证伪 / 待实现 / 当前正确方向
- 文件：hook_game_text_takeover_callsite_real_jmp / swaptarget / validated_v2

状态标签说明：
[已确认] 断点、伪代码、现场寄存器/栈或有效 dump 已验证
[已证伪] 已经证明错误，后续不要再走
[候选] 当前看起来可能对，但还没彻底跑通
[历史备份] 有参考价值，但不是当前主路线

============================================================
1. 项目目标与路线历史
============================================================
1.1 当前真正目标 [已确认]
- 在你自己的 D3D9 + ImGui hook 窗口/界面中，借用 Maple 客户端现成的文字链，画出和游戏一致的文字。
- 不是 atlas 最终方案，不是 glyph replay 最终方案，不是 INT3/VEH 最终方案。
- 当前最终目标文件名围绕：hook_game_text_takeover_callsite_real_jmp_swaptarget。

1.2 历史 atlas 路线 [历史备份]
- 旧思路：离线/半离线导出字体 atlas + glyph metrics，再在 ImGui 里 AddImage() 自绘。
- 这条路线有一批已经确认过的数据，仍有参考价值（尤其 5000E520 / 5000E640 / 5000B960 的职责分工）。
- 但用户当前明确目标是“游戏文字接管”，不是 atlas 导出成品。

1.3 明确不要的东西 [已确认]
- 不再用 glyph replay / atlas 作为最终方案
- 不再用 INT3/VEH 作为最终方案
- 不再 patch maplestory.004382EA
- 不再把 callsite 当函数入口 detour 去做 E9 jmp trampolines

============================================================
2. 稳定工程基座
============================================================
2.1 当前稳定基础
- 你原始 hook.cpp 的 Present + ImGui 覆盖层是稳定的。
- 文字原始链路：DrawOutlinedText -> ImDrawList::AddText
- 默认字体初始化：AddFontFromFileTTF(... msyh.ttc ...)

2.2 当前真正需要接管的，只是文字层
- D3D9 Present hook / WndProc / ImGui 总体框架不要乱改
- 问题核心不是 D3D9，而是文字链调用点、对象生命周期、target 裁剪

============================================================
3. 关键地址总表
============================================================
3.1 文字链核心地址 [已确认]
- 50009B80  : 高层 wrapper，最终会调用 50018890 和 5000E640
- 50009C1C  : 真正的 callsite，原始指令是 call 5000E640
- 5000E640  : 整串文本绘制/排版主函数（当前主接管点）
- 5000E520  : 单字 glyph/advance 核心函数（只做理解，不再最终 hook）
- 5000B960  : glyph 提交到目标的更底层绘制函数
- 50018890  : VARIANT/alpha/辅助对象转换函数
- 004382EA  : 上层 wrapper 内 call eax（EAX=50009B80），错误 patch 点

3.2 关键对象头 / vtbl 头 [已确认]
- textCtx 头：
  - [0x00] = 0x50025B14
  - [0x04] = 0x50025B10
- target 头：
  - [0x00] = 0x50025888
  - [0x04] = 0x50025880
- 其他相关：
  - 50025A4C / 50025AA8 / 50025414 等经常出现在 textCtx/target 内部字段

3.3 历史 atlas 路线地址 [历史备份]
- 5000BBC0 : glyph miss 时生成单字 glyph
- 5000BD20 : 16bpp atlas 写入
- 5000E9D8 / 5000E9C9 : 旧时确认过能抓整串 UTF-16 技能名的点

============================================================
4. 函数伪代码 / 调用约定（当前最重要）
============================================================
4.1 sub_50009B80 [已确认]
原型（伪代码）：
int __stdcall sub_50009B80(int a1, int a2, int a3, int a4, int a5, VARIANTARG pvarSrc, VARIANTARG a7, _DWORD *a8)

关键逻辑：
- a5 为空时直接失败
- v10 = sub_50018890(&pvarSrc, 255)，并 clamp 到 <=255
- v11 = sub_50018890(&a7, 0)
- sub_5000E640(a5, a2, a3, a4, v10, v11)
- a5 就是 textCtx

意义：
- 50009B80 是高层 wrapper，不是最终 patch 点
- 004382EA 调的其实是它，不是 5000E640

4.2 sub_5000E640 [已确认]
伪代码核心：
void __thiscall sub_5000E640(int this, int a2, char *a3, int a4, _WORD *a5, int a6, _DWORD *a7)

已确认职责：
- this = textCtx
- a2 = drawState / target state
- a3 = x（局部坐标 / pen 相关）
- a4 = y
- a5 = UTF-16 字符串
- a6 = alpha / 颜色缩放相关
- a7 = 另一个与 pen/base / Variant 相关的对象

最关键一句：
- v15 = *(_DWORD *)(this + 56)   // 即 textCtx + 0x38
- sub_5000B960(v8, a4, v15, v23, 0, 0, ...)

关键结论：
- 决定最终绘制目标 / 裁剪目标的，是 textCtx + 0x38
- 不是单纯 x/y，也不是外部长期保存的 drawState 变量

4.3 sub_5000E520 [已确认]
可理解为：
int __thiscall GetGlyph(FontCache* this, wchar_t ch, Rect* outRectOrNull)

已确认：
- 返回 EAX = advance
- 非空 out buffer 时可写 glyph rect
- 最终方案不要再 hook 它；它只是理解链路的底层依据

4.4 真正 callsite：50009C1C [已确认]
反汇编：
50009C1C  E8 1F4A0000  call canvas_m.5000E640

这才是当前真正应该 patch 的点。

============================================================
5. 真正 callsite 的现场参数（非常重要）
============================================================
5.1 有效现场 1：技能文字 [已确认]
地址：50009C1C

寄存器/栈：
- ECX = 32727DEC   ; textCtx
- [ESP+00] = 3DE9E29C ; drawState
- [ESP+04] = 0x32     ; x
- [ESP+08] = 0x60     ; y
- [ESP+0C] = L"魔法抗性"
- [ESP+10] = 0xFF     ; alpha
- [ESP+14] = 0x00     ; a6

5.2 有效现场 2：另一条文本 [已确认]
地址：50009C1C

寄存器/栈：
- ECX = 316D1D74
- [ESP+00] = 1E3E1F24 ; drawState
- [ESP+04] = 0x06     ; x
- [ESP+08] = 0x07     ; y
- [ESP+0C] = L"你的旅游怎么样"
- [ESP+10] = 0xFF
- [ESP+14] = 0x00

关键结论：
- 5000E640 是正常 thiscall 风格：ECX=textCtx，剩余参数走栈
- 之前把 EDX 当作正式调用约定的一部分，是误判

============================================================
6. 正确 / 错误 patch 方式
============================================================
6.1 错误 patch 点 [已证伪]
- 004382EA 不是最终 callsite，只是 wrapper
- 任何基于 004382EA + [ebp+..] 的取参方式都作废

6.2 错误 detour 方式 [已证伪]
- 不能用 SmartSafeHook 那种“函数入口 E9 detour + trampoline”去处理 50009C1C 这种 callsite
- 不能把 call 指令改成 jmp 指令

6.3 正确的 callsite patch 方式 [已确认]
- 原始：50009C1C = call 5000E640
- 正确：把它改成 call stub
- 进入 stub 时，栈必须保持原始形状：
  - [esp+00] = 50009C21 （原始返回地址）
  - [esp+04] = drawState
  - [esp+08] = x
  - [esp+0C] = y
  - [esp+10] = text
  - [esp+14] = alpha
  - [esp+18] = a6
  - ecx      = textCtx

6.4 stub 的正确写法 [已确认]
- stub 里只允许裸采样（mov 到全局 POD）
- 然后直接 jmp 5000E640
- 绝不能：
  - call 5000E640 + ret
  - pushad/popad
  - call 任何 C++ helper
  - copy string
  - __try/__except
  - 读对象深层字段

============================================================
7. textCtx / target 结构与样本
============================================================
7.1 textCtx 样本 A：技能列表字体链 [已确认]
textCtx = 4DFF3BC4

关键字段：
- 0x00 = 50025B14
- 0x04 = 50025B10
- 0x10 左右看到字体名 -> L"Nsimsun"
- 0x14 / 0x18 附近包含字号、颜色/样式等
- 0x38 = 4E04CF54  （skillTarget）

原始 dump 摘要：
4DFF3BC4  50025B14
4DFF3BC8  50025B10
4DFF3BD4  -> L"Nsimsun"
4DFF3BD8  0000000C
4DFF3BFC  4E04CF54   ; [textCtx+0x38]

7.2 skillTarget 样本 A [已确认]
skillTarget = 4E04CF54

头：
- 0x00 = 50025888
- 0x04 = 50025880

关键字段：
- +0x28 = 0x100
- +0x2C = 0x18
- +0x30 = 0x100
- +0x34 = 0x0C
- +0x38 = 0x01
- +0x3C = 0x02

意义：
- 这是技能列表自己的 target
- 用它作画，文字会被裁剪在技能列表区域内

7.3 另一组有效 textCtx/target 样本 [已确认]
textCtx = 0896482C
target  = [textCtx+0x38] = 2FFCA4DC

textCtx 头正确：
0896482C  50025B14
08964830  50025B10
0896483C  -> L"Nsimsun"
08964840  0000000C
08964864  2FFCA4DC

目标头正确：
2FFCA4DC  50025888
2FFCA4E0  50025880

关键字段：
- +0x28 = 0x100
- +0x2C = 0x30
- +0x30 = 0x100
- +0x34 = 0x0C
- +0x38 = 0x01
- +0x3C = 0x04

实测现象：
- 把 target 换成这组后，文字会跑到任务/对话框区域，并挤成一坨
- 说明 target 交换本身有效
- 说明换 target 后必须按新 target 重算 localX/localY

7.4 有效白字/系统类候选 target 样本 [已确认]
textCtx = 4E1877A4
[textCtx+0x38] = 4A5B6AAC

目标头：
4A5B6AAC  50025888
4A5B6AB0  50025880

字段摘要：
- +0x28 = 0x100
- +0x2C = 0x78
- +0x30 = 0x100
- +0x34 = 0x0C
- +0x38 = 0x01
- +0x3C = 0x0A

说明：
- 这是一个有效 target 候选
- 不能保证它就是最终最佳“全屏 target”，但它是合法同类对象

7.5 误抓/错位样本 [已证伪]
- 任何从 4DFF3BC3 / 04CF5400 / 4DFF3BFB 这类偏 1 字节开始的 dump 都是错位的
- 这类数据不能用于写代码

============================================================
8. 现象 -> 根因 对照表
============================================================
8.1 注入即崩 [已定位过多次]
可能根因：
- 用 INT3/VEH 作为最终方案
- callsite 被错误改成 jmp detour
- stub 里 call 5000E640 + ret，额外压栈
- stub 里调用了 C++ helper / CopyLastText / 读对象 / __try

8.2 打开技能栏崩 [已定位]
根因：
- patch 了错误点 004382EA
- 或 stub 内部破坏了 50009C1C 原始调用帧

8.3 过一段时间才崩 [已定位]
根因：
- 长期持有会过期的 UI 对象指针（textCtx / target）
- 技能列表关闭、UI 重建后，仍拿旧对象重放

8.4 文字只在技能列表右侧露一小半 [已定位]
根因：
- 字体链已经走通，但当前 target 还是 skillTarget
- 即 textCtx+0x38 仍然指向技能列表自己的裁剪区

8.5 文字跑到任务/对话框里并挤成一坨 [已定位]
根因：
- target 交换成功了
- 但 localX/localY 仍按旧 target 的基准在算
- 换 target 后没重做坐标基准，导致投影到另一套 UI 区域里挤成一坨

8.6 设置白字/系统白字抢刷新 [历史问题，已分析]
根因：
- 以前错误地让所有 E640 命中都覆盖当前 owner/textCtx/drawState
- 需要对候选 target / 来源链做分类，而不是谁最新就用谁

============================================================
9. 当前正确的重新开发方向（不再额外取数也能做）
============================================================
9.1 最终方案核心
- 只 patch 50009C1C
- stub 裸采样 -> jmp 5000E640
- 捕获 skillTextCtx / skillTarget / 其他候选 target
- draw 时以 skillTextCtx 作为字体来源
- 可选：临时替换 textCtx+0x38 为 activeTarget
- 换 target 时，必须按 activeTarget 的基准重算 localX/localY

9.2 一定要加的保护
- lastHitTick：超过 100~200ms 没再命中技能列表文字链，就停用游戏文字，回退 fallback
- ValidateTextCtxAndTarget():
  - textCtx[0]==50025B14 && textCtx[4]==50025B10
  - target[0]==50025888 && target[4]==50025880
- 不长期缓存 target 单独使用；以 textCtx 为主，每次 draw 前现读/现验

9.3 local 坐标正确计算规则 [当前最重要]
不能再这样：
- 先按旧 target 算 localX/localY
- 再把 textCtx+0x38 换成新 target 去画

必须这样：
1) 先决定 activeTarget
2) 读 activeTarget 的相关基准字段
3) 用 activeTarget 计算 localX/localY
4) 再把 textCtx+0x38 = activeTarget
5) 再 draw
6) 再恢复原 target

可直接使用的表达式（需要接手人最终再按代码核对具体字段含义）：
- targetBaseX ≈ *(int*)((BYTE*)activeTarget + 0x28)
- targetBaseY ≈ *(int*)((BYTE*)activeTarget + 0x2C)
- localX = absX - targetBaseX
- localY = absY - targetBaseY

注意：
- 这里 +0x28/+0x2C/+0x30/+0x34 已经和裁剪/区域强相关
- 但在最终代码里仍应保留 debug 打印，不要绝对写死语义为 left/top/right/bottom

============================================================
10. 已证伪路线清单（不要再走）
============================================================
- patch 004382EA
- 把 50009C1C 当函数入口，用 E9 detour/trampoline
- INT3/VEH 做最终成品
- glyph replay / atlas / B960 replay 作为最终方案
- 在热路径 stub 里调 C++ helper
- 在热路径 stub 里 call 5000E640 + ret
- 在错误 callsite 用 [ebp+0C]/[ebp+10]/[ebp+14] 猜 x/y/text
- 用错位 dump（偏 1 字节）写代码

============================================================
11. 历史 atlas / glyph 路线数据（保留查询价值）
============================================================
11.1 atlas 路线关键链路 [历史备份]
- 5000E640 : 逐字符排版
- 5000E520 : glyph 查/建、返回 advance
- 5000B960 : glyph 提交给布局/绘制目标
- miss -> 5000BBC0 / 5000BD20

11.2 历史 FontCache / bucket 数据 [历史备份]
- 某次有效 FontCache = 0x2461B344
- bucketTable = 0x2B385334
- bucketCountRaw = 0xFF
- bucketSlots = 256
- pages = 6
- glyphs = 115
- runtimeGlyphs = 115

11.3 历史 page pixelBase [历史备份]
- 0x1FF00000  lineHeight=12
- 0x0C600000  lineHeight=14
- 0x26BF0000  lineHeight=12
- 0x26DB0000  lineHeight=12
- 0x28AA0000  lineHeight=12
- 0x26BA0000  lineHeight=12

11.4 历史结构体 [历史备份]
RemoteGlyphNode:
- +0x00 unk0
- +0x04 next
- +0x08 ch
- +0x0C x
- +0x0E width
- +0x10 advance
- +0x12 row
- +0x14 pageNode

RemoteRealPageDesc:
- +0x1C width
- +0x20 lineHeight
- +0x24 pitch
- +0x28 pixelBase

意义：
- 这批数据将来如果要回头做 atlas 或对照 glyph cache，仍可用
- 但当前“游戏文字接管”主路线不依赖它们做最终实现

============================================================
12. 文件版本状态总表
============================================================
以下是本轮生成过的版本及当前建议状态：

[废弃 / 仅历史参考]
- hook_game_text_complete.cpp
- hook_game_text_complete_fixed.cpp
- hook_game_text_complete_fixed_probe.cpp
- hook_game_text_runtime_direct*.cpp
- hook_game_text_takeover_fixed*.cpp
- hook_game_text_takeover_callsite.cpp
- hook_game_text_takeover_callsite_fixed*.cpp
- hook_game_text_takeover_callsite_minhotpath.cpp
- hook_game_text_b960_replay.cpp
- hook_game_text_auto_capture.cpp

[当前主线相关]
- hook_game_text_takeover_callsite_real.cpp
- hook_game_text_takeover_callsite_real_jmp.cpp
- hook_game_text_takeover_callsite_real_jmp_validated_v2.cpp
- hook_game_text_takeover_callsite_real_jmp_swaptarget.cpp

当前建议：
- 以 callsite_real_jmp / validated_v2 / swaptarget 三者思想合并重做
- 不要直接依赖任何一个旧文件“继续修到死”
- 以本索引中的已确认事实重写更稳

============================================================
13. 交接给下一个开发者的任务清单
============================================================
任务 1：重建最小安全版 callsite_real_jmp
- 只 patch 50009C1C
- stub 裸采样后 jmp 5000E640
- 打开技能栏不崩

任务 2：加入 capture 时效和对象头校验
- 防止延迟崩
- 超时自动 fallback

任务 3：重建 skillTextCtx + skillTarget 字体链
- 先接受它只能画在技能列表内
- 只为证明字体链走通

任务 4：做 target 候选选择器
- 运行时收集多个 target 候选
- 不再人工抓新 dump 才能继续开发
- debug UI 可切换 target 候选

任务 5：实现 swaptarget + activeTarget 重基准 localX/localY
- 解决跑到任务框里挤成一坨
- 解决仍被技能列表裁剪

任务 6：最后收尾 UI 效果
- 让技能名、等级、标题、按钮文字都走游戏字体
- 减少或去掉 fallback
- 保证不崩、不拖影、不乱跑、不被错误裁剪

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VividVVO) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
