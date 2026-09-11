## lofi-room-by-astra

> 本项目所有原创 3D 内容采用「Blender 源文件为唯一事实来源」的管线。

# AGENTS.md — lofi_room 项目协作规则(Blender → GLB → Three.js)

本项目所有原创 3D 内容采用「Blender 源文件为唯一事实来源」的管线。
任何 AI Agent(或人类)修改模型时必须遵守以下规则。

## 目录职责

| 路径 | 职责 |
| --- | --- |
| `/blender` | 原始工程文件(`lofi_room.blend`) |
| `/public/models` | 最终交付给 Web 的 **GLB** 产物(`demo.glb`,由导出脚本生成) |
| `/scripts/export_glb.py` | **唯一**的 Blender → GLB 导出脚本 |
| `/src` | Three.js 前端代码 |

## Blender / MCP 环境(本机已全局配置好,无需再装)

- Blender 5.2.1 LTS（请从 Blender 官方安装位置启动，需先在桌面打开）
- Blender 扩展 BlenderMCP 已启用,打开 Blender 后**自动**桥接 `localhost:9876`(N 面板 → BlenderMCP 可查看)
- Codex 已注册 MCP:`blender_mcp`(工具调用需带 `user_prompt` 参数)
- 任务结束时调用 `release_blender_instance` 释放控制权

## 本项目首次建模的固定动作

Blender 当前打开的场景可能属于其他项目。开始本项目时:

1. 先新建空场景:`bpy.ops.wm.read_homefile(use_empty=True)`
2. 创建标准 Collections:`ROOM_SHELL`(房间壳体)/ `FURNITURE`(家具)/ `PROPS`(小物件)/ `LIGHTS` / `CAMERAS`
3. 建模完成后保存为 `blender/lofi_room.blend`(用 `bpy.ops.wm.save_as_mainfile`)

## 修改 3D 模型的标准流程(必须完整走完)

```text
1. Inspect Blender scene        (list_blender_instances / get_scene_info)
2. Modify using Blender MCP/bpy (execute_blender_code,或结构化节点工具)
3. Inspect viewport             (get_viewport_screenshot,视觉确认后再继续)
4. Save .blend                  (保存到 /blender/lofi_room.blend)
5. Export .glb                  (执行 /scripts/export_glb.py,见下)
6. Verify file                  (脚本自带 SIZE/MAGIC 断言,输出 EXPORT_OK)
7. Run web app                  (npm run dev)
8. Verify browser               (刷新页面 + Console 无错误,必要时截图对比)
```

### 导出 GLB 的正确方式

通过 MCP 在 Blender 内执行(脚本从 `bpy.data.filepath` 自动定位项目根,无硬编码路径):

```python
exec(open(r'<PROJECT_ROOT>\scripts\export_glb.py', encoding='utf-8').read())
```

或无头模式:`blender -b blender\lofi_room.blend -P scripts\export_glb.py`

## 禁止事项

- ❌ 禁止只修改 GLB 而不保留 `.blend` 源(GLB 是产物,不是源)
- ❌ 禁止覆盖/删除场景中已有的其他模型(新增对象用独立命名)
- ❌ 禁止绕过 `export_glb.py` 手工零散导出
- ❌ 禁止把 Blender 桥接端口暴露到 `0.0.0.0` / 公网(只允许 localhost)
- ❌ 禁动 `~/.codex/config.toml` 里的其他 MCP 与配置

## 经实战验证的 bpy 陷阱(必读)

1. **删除对象前先转移 active**:
   `bpy.context.view_layer.objects.active` 指向待删对象时,先切到安全对象并取消选择,
   再 `bpy.data.objects.remove(obj, do_unlink=True)`(否则桥接的编辑记录器会误报)。
2. **不要用 `bpy.context.active_object` 接新建对象**:
   先 `pre = set(bpy.data.objects)`,ops 之后用集合差集找新对象
   (MCP 桥接的执行上下文里 active_object 可能返回已删除的旧引用)。
3. **`execute_blender_code` 删除 datablock 后的变更报告**:
   扩展 v1.17.0 在报告 `changes.deleted` 时访问已释放引用会误报
   `StructRNA of type Object has been removed`(本机扩展已打补丁修复;
   若升级后复现:先确认对象是否实际已删除,再考虑报告 bug)。
4. **glTF 材质**:统一用 Principled BSDF(Metal/Rough),发光用
   Emission Color + Strength;透明物体设 Alpha 且 blend 方式设为 BLEND。
5. **面向实时渲染**:lofi 房间整体建议 < 80k 三角面;硬表面细节用 Bevel Modifier
   (导出时 `export_apply=True` 自动烘焙);不要用负缩放;法线保持向外。
6. **灯光只放 `LIGHTS` Collection、相机只放 `CAMERAS`**:导出脚本会自动排除它们,
   网页端的灯光由 three.js 场景自己提供。

## 场景组织与命名约定

- Collection:`ROOM_SHELL` / `FURNITURE` / `PROPS` / `LIGHTS` / `CAMERAS`
- 对象命名:语义化 PascalCase(如 `Desk_Main`、`Bed_Frame`、`Window_Frame`、`Lamp_Shade`)
- 材质以 `MAT_` 开头(如 `MAT_Wood_Oak`、`MAT_Fabric_Warm`)
- 尺寸按真实世界比例(米):房间高度 2.7~3m,桌面高 0.75m,以此类推

---
> Source: [ACKEYTLC/lofi-room_by_Astra](https://github.com/ACKEYTLC/lofi-room_by_Astra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
