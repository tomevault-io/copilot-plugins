## cao-s-portfolio-cn

> 本工作流适用于 Project 1、Project 2、Project 3、Project 4、Project 5 及以后所有作品集项目。用户要求修改某一个 Project 时，默认执行以下完整流程；用户当次任务中的明确要求优先。

# 标准作品集项目更新工作流

本工作流适用于 Project 1、Project 2、Project 3、Project 4、Project 5 及以后所有作品集项目。用户要求修改某一个 Project 时，默认执行以下完整流程；用户当次任务中的明确要求优先。

## 一、任务范围

1. 开始前先读取：
   - `AGENTS.md`
   - `PROJECT_CONTEXT.md`
2. 只处理用户当前明确指定的 Project，不修改其他 Project。
3. 除非用户明确要求，否则不要修改：
   - Home
   - About
   - Contact
   - Navigation
   - 全局字体
   - 全局颜色
   - Cloudflare 配置
   - GitHub 配置
   - 构建配置
   - `package.json`
   - `worker`
   - Vite 配置
   - 与本次 Project 无关的代码
4. 优先修改现有文件，不创建重复组件、重复 section、重复数据，避免新版与旧版并存。

## 二、开始前检查

每次收到新的 Project 修改任务时，先定位当前 Project 对应的：

- project id
- `projects.js` 数据
- 图片目录
- gallery 顺序
- 相关组件
- 相关 CSS

只读取完成当前任务真正需要的文件，不扫描整个仓库。

## 三、新 PNG 图片处理

如果用户提供新的高清 PNG，优先从类似 `incoming-projectX/` 的临时目录读取，例如 `incoming-project5/`。

不要先删除网站现有图片。先检查每张新图片的：

- 文件名
- 像素宽度
- 像素高度
- 文件大小
- 宽高比
- 对应的旧图片
- 对应的 gallery 顺序

如果无法确定新旧对应关系，不要猜；停止并报告。

## 四、PNG 网页优化标准

所有 PNG 必须：

- 保持原始宽高比
- 不裁切
- 不改变构图
- 不改变内容
- 不改变展板版式
- 不改变图片顺序
- 不自动放大小图
- 禁止 upscale
- 默认保持 PNG
- 不进行无意义重复压缩
- 不反复 resize

对于明显过大的展板，一般将最长边控制在约 2500–3000 px，具体根据展板内容判断。如果文字、分析图、CAD/线稿或小字号较多，可以接近 3000 px 或适当更高，以保证阅读清晰。

不要为了减少体积造成文字模糊、线稿模糊、锯齿、图纸细节丢失或明显压缩痕迹。

对于尺寸已经合理的图片，不要重新编码。

对于尺寸明显过小的图片，不要放大、不要 AI upscale、不要自动锐化。只报告：

- 文件名
- 当前尺寸
- 文件大小
- 建议最低尺寸
- 建议重新导出的尺寸范围

由用户重新提供高清版本。

## 五、安全替换图片

新图片处理完成并验证可以正常读取后，再覆盖网站中对应的旧图片。

- 新旧文件名相同：直接覆盖同名旧文件。
- 新旧文件名不同：先更新正确引用，再检查旧图片是否仍被使用。

禁止先删除旧图片再处理新图片。必须遵循：

新图处理 → 验证成功 → 覆盖/替换 → 检查引用 → 最后清理废弃文件

## 六、废弃文件和旧版本自动清理

这是每次 Project 修改完成后的必要步骤。必须检查当前 Project 相关目录中是否存在：

- 旧低分辨率图片
- old 文件
- backup 文件
- copy 文件
- temp 文件
- 多余导出文件
- 同一展板的旧版本
- 已不在 gallery 中使用的图片
- 已不被代码引用的图片
- incoming 临时图片
- 重复 PNG
- 已废弃的数据文件
- 本次操作生成的临时文件

对准备删除的文件，必须先确认：

1. 不在 `projects.js` 中引用。
2. 不在任何相关组件中引用。
3. 不在 CSS 中引用。
4. 不在当前 gallery 数组中。
5. 已经存在正确的新版本替代。
6. 删除不会导致页面缺图。

只有以上条件全部满足才允许删除。如果有任何疑问，不要删除，报告给用户。

不要删除其他 Project 的文件，也不要删除项目外部的原始设计资料。

## 七、临时 incoming 目录

如果使用 `incoming-projectX/`，在最终图片成功优化、替换并验证后，清理其中已经不再需要的临时副本。如果目录已经为空，可以删除空目录。

不要在网站代码仓库中保留无意义的重复高清源文件。

## 八、完成后的项目检查

必须检查：

- 图片数量正确
- 图片 01、02、03… 顺序正确
- gallery 顺序正确
- 图片路径正确
- 没有缺图
- 没有重复图片
- 没有旧版本残留
- 没有无引用图片
- 没有临时文件残留
- 没有重复组件
- 没有重复 CSS
- 没有旧代码残留
- 没有修改其他 Project

## 九、本项目不在本地 build

本项目的标准部署流程是：

Codex 修改本地 Git 仓库 → GitHub `main` → Cloudflare 自动 build → Cloudflare 自动 deploy

因此，不要默认运行：

- `npm install`
- `npm run build`

不要因为没有 Node/npm 而安装 Node.js。除非用户明确要求本地 build，否则跳过本地 build，由 Cloudflare 负责正式 build 和部署。

## 十、Git 提交流程

只有当本次 Project 修改、图片替换和废弃文件清理全部完成后，才进入 Git 流程。

### 第一步：确认变化范围

运行：

```text
git status --short
git diff --name-status
```

确认变化只属于当前 Project 和本次任务。如果发现无关文件，停止，不要提交。

### 第二步：精确暂存

只 `git add` 本次明确修改的文件。禁止默认使用：

```text
git add .
```

### 第三步：提交信息

使用描述当前 Project 的 commit message，例如：

```text
Update Drift & Dwell project boards
Update [Project Name] project boards
```

### 第四步：分支和远程安全检查

确认当前正式分支为 `main`，并安全检查远程状态。

- 不得 force push。
- 不得重写 Git 历史。
- 如果远程存在冲突，停止并报告。

### 第五步：推送

如果没有冲突，push 到 `origin/main`。

## 十一、Cloudflare

GitHub push 成功后，不要手动在本地 build。Cloudflare 会自动执行：

GitHub `main` → build → deploy

Codex 不需要修改 Cloudflare 配置。

## 十二、完成后的最终清理

Git push 前再执行一次最终废弃文件检查，重点检查：

- 当前 Project 图片目录
- 临时 incoming 目录
- 本次任务新建的临时文件

确保最终仓库只留下网站实际使用的文件。不要为了安全而保留已经明确废弃的旧版本图片，但删除任何文件之前必须确认没有引用。

## 十三、最终报告

每次完整任务结束后，只向用户报告：

### 【Project】

- 项目名称
- 项目 ID

### 【代码】

- 修改了哪些代码文件
- 为什么修改

### 【图片】

- 哪些图片被替换
- 哪些图片被缩小
- 原尺寸 → 最终尺寸
- 原大小 → 最终大小

### 【清理】

- 删除了哪些旧图片
- 删除了哪些临时文件
- 删除了哪些无引用文件
- 是否仍存在疑似废弃但没有删除的文件

### 【Git】

- 实际提交文件数量
- commit SHA
- 是否成功 push 到 `origin/main`
- 是否存在冲突
- 是否有无关文件进入 commit

### 【部署】

- 不进行本地 build
- 等待 Cloudflare 自动 build/deploy

---
> Source: [ziyuancao2000-cloud/Cao-s-Portfolio-CN](https://github.com/ziyuancao2000-cloud/Cao-s-Portfolio-CN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
