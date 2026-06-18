## lvgl-preview-skill

> |


# lvgl-preview — LVGL 代码生成 + UI 实时确认

把 LVGL (MicroPython) 界面代码在电脑端渲染成 PNG, Agent 读图确认。闭环:
**写/改 LVGL 代码 → 一行渲染 → Read PNG 看效果 → 迭代**。不碰真机、不开浏览器。

**版本自适配**: 渲染器与原语库 (`lvkit.py`) 自动检测 LVGL 8.x 还是 9.x
(`_V8 = hasattr(lv, 'disp_drv_t')`), 字体 / `line` / `qrcode` / display 绑定按版本分支,
**同一份 `build(scr)` 在两端都跑**。桌面预览二进制是哪个版本就按哪个渲染; 设备端
(如 K230 CanMV = v9) 跑同一份 UI 代码, 只换显示初始化 (见 `docs/device-binding.md`)。

## 前置: 桌面运行时

需要一个本地 `lv_micropython` unix 二进制。检查:

```bash
ls "${LVMP_BIN:-$HOME/lv_micropython/ports/unix/micropython}"
```

没有就构建 (macOS arm64 的坑已内置修复; Linux 需 `apt install libsdl2-dev`):

```bash
bash scripts/build_runtime.sh            # 默认 LVGL v8 (release/v8)
bash scripts/build_runtime.sh ~ v9       # 或 LVGL v9 (master)
export LVMP_BIN=$HOME/lv_micropython/ports/unix/micropython
```

## 用法

UI 写成一个模块, 定义 `build(scr, **state)` —— 在 `scr` 上搭界面 (渲染器已
`lv.init()` + 注册好虚拟 display, `scr` 直接可用)。然后:

```bash
python3 scripts/lvgl_preview.py <ui.py> --out preview.png --size 480x800 --state '{"online": true}'
```

**渲染后必须 Read 那张 PNG 自己看一眼确认** —— 这是本 skill 的核心 (改完即看)。
不满意就改 `<ui.py>` 再跑, 秒级迭代。模板见 `templates/ui_example.py`。

参数: `--size WxH` (默认 480x800) | `--state '<json>'` 传给 build 的 kwargs |
`--fn <name>` 改 build 函数名 | `--out` 输出 PNG。

## LVGL MicroPython 速查 (照着写, 一次就对)

- 颜色: `lv.color_hex(0xRRGGBB)`。选择器参数传 `0` 或 `lv.PART.MAIN` (等价)。
- **没有原生 circle**: 用方 obj + `obj.set_style_radius(360, 0)` 画圆。
- 新建 obj 默认带边框/内边距/滚动条, 干净的图元要清掉:
  `set_style_border_width(0,0)`, `set_style_pad_all(0,0)`, `clear_flag(lv.obj.FLAG.SCROLLABLE)`。
- 填充/透明: `set_style_bg_opa(lv.OPA.COVER, 0)` / `lv.OPA.TRANSP`。
- 文字: `lbl = lv.label(parent); lbl.set_text("x"); lbl.set_style_text_color(c, 0)`。
- 布局: `obj.align(lv.ALIGN.CENTER, dx, dy)`, `a.align_to(b, lv.ALIGN.OUT_BOTTOM_MID, 0, 0)`,
  `obj.set_size(lv.pct(100), 60)`, `obj.set_pos(x, y)`。
- **坑: 不注册 display 时 `lv.scr_act()` 返回 None** → 任何 `scr.set_style_*` 都会
  "no such attribute"。本 skill 的渲染器已先注册 display, build() 里直接用 scr 即可。
- 中文文字: 默认 Montserrat 无 CJK 字形。用 `lvkit.cjk_font(size)` 加载 ttf 渲染真中文。
- **版本差异不用你管**: 用 `lvkit` 的封装 (下) 而非裸 API, 8.x/9.x 自动适配。

## 自带原语库 `lib/lvkit.py` (`import lvkit`)

渲染器已把本 skill 的 `lib/` 放进搜索路径, 任意 UI 模块可直接 `import lvkit as K` 用:
- 图元: `K.rect / K.dot(实心圆) / K.ring(空心环) / K.vline(线) / K.label`
- 控件: `K.person(人形剪影)`, `K.seven_seg_char / K.clock_hhmm(七段时钟)`
- 中文: `K.cjk_font(size)` → FreeType 加载中文 ttf (v8 用 `ft_info_t`, v9 用
  `freetype_font_create`, 自动分支), 返回 lv_font (失败返回 None, 自行 fallback)。
  字体路径: 桌面默认系统 ttf, 设备默认 `/sdcard/res/font/...` (见 `lvkit.CJK_FONT_PATH`)。
  注意大字体 + 全屏帧缓冲吃内存, 渲染器已默认 `-X heapsize=64m`。
- 版本标志: `K._V9` (True=LVGL 9.x), 自定义原语要分支时可用。

## 移植到设备

`ui/` 代码一字不改, 只换入口的 display 初始化:
- **桌面**: 渲染器用 SDL 虚拟 display (已封装在 `scripts/render.py`)。
- **设备 (v9, 如 K230 CanMV)**: `Display.init(Display.ST7701, ...)` + `lv.disp_create` +
  `set_draw_buffers` + `set_flush_cb -> Display.show_image(layer=...)`。
- **设备 (v8)**: `disp_draw_buf_t` + `disp_drv_t` + `flush_cb`。

完整 v8/v9 disp 绑定 + 中文字体 + 截图回传的可复制写法见 `docs/device-binding.md`。

## 范围

- 渲染的是 **LVGL 控件树**。摄像头实时画面 / 硬件叠加层 (如 K230 走 `image` 通道) 不在内 ——
  那部分设备上和 LVGL OSD 分层共存, 预览只负责 UI "外壳" (状态栏 / 按钮 / 状态屏)。
- 颜色: LVGL 32-bit, 落盘是 BGRA, `raw2png.py` 用 PIL `raw='BGRA'` 还原, 无需关心。

## 实战案例

`examples/k230-access-control-7screens.png` —— 用本 skill 做的 K230 门禁全套 7 屏
(待机/检测/识别中/成功/失败/未绑定/休眠), 桌面预览确认后上真机 (LVGL v9) 像素一致。

---
> Source: [zhoushoujianwork/lvgl-preview-skill](https://github.com/zhoushoujianwork/lvgl-preview-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
