## manim

> 用 Manim 制作数学/物理教学视频。做视频相关的任何事之前,先读

# Manim 教学视频项目

用 Manim 制作数学/物理教学视频。做视频相关的任何事之前,先读
`.claude/skills/manim-video-production/SKILL.md` —— 完整工作流和踩过的坑都在那里。

## 目录结构

```
mvkit/              共享构件:theme(字体配色)、layout(排布/槽位)、verify(坐标核验)
tools/              命令行工具:doctor / verify_layout / render / build
projects/<slug>/    一个视频一个目录
  outline.md          分镜脚本
  clips.txt           场景类名,按出场顺序
  scenes/             每段一个 .py,一个 Scene 类
  media/              渲染产物(不进版本库)
  out/final.mp4       成片(不进版本库)
projects/_template/ 新项目的骨架,直接复制
```

## 命令

**所有 Python 命令都用 `.venv/Scripts/python.exe`**,全局 python 里没有 Manim。

```bash
.venv/Scripts/python.exe tools/doctor.py                       # 环境自检
.venv/Scripts/python.exe tools/verify_layout.py <场景文件>       # 渲染前的坐标核验
.venv/Scripts/python.exe tools/render.py <场景文件> --draft      # 480p15 试跑
.venv/Scripts/python.exe tools/render.py <场景文件> --final      # 1080p60 交付
.venv/Scripts/python.exe tools/render.py <场景文件> --mid        # 720p30,3D 场景用
.venv/Scripts/python.exe tools/build.py projects/<slug>        # 拼接成片
```

## 硬性约定

写场景代码时这几条不要违反:

1. **每段一个 Scene 类、一个文件。** 不要写一个巨大的 Scene —— 改一处就要重渲全片。
2. **每个场景文件都要有 `probes()`。** 声明哪些元素会同时出现,渲染前用数值核验
   布局,不要靠渲完抽帧用眼睛找问题。
3. **说明文字用 `layout.Slot` 替换,不往下堆叠。** 堆叠会越堆越低直到掉出画面。
4. **公式里不能有汉字。** LaTeX 默认引擎会编译失败,用 `theme.math_with_cn()`。
5. **字体字号配色从 `mvkit.theme` 取**,不要在场景里硬编码。
6. **先 `--draft` 跑通再上高清。**

## MiKTeX PATH

MiKTeX 装在用户 PATH 里。如果某个终端会话报"找不到 latex",是这个会话的环境变量
是启动时的快照,新开终端即可;或在当前会话里刷新:

```powershell
$env:Path = [Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [Environment]::GetEnvironmentVariable("Path","User")
```

---
> Source: [cnmzsjbz199328/Manim](https://github.com/cnmzsjbz199328/Manim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
