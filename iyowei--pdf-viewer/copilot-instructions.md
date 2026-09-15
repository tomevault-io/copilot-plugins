## pdf-viewer

> 基于 [mozilla/pdf.js](https://github.com/mozilla/pdf.js) v5.3.93 实现的开箱即用 PDF Web 阅读器。

# 开箱即用 PDF Web 阅读器

基于 [mozilla/pdf.js](https://github.com/mozilla/pdf.js) v5.3.93 实现的开箱即用 PDF Web 阅读器。

## 关联信息

查看 @README.md 了解项目。
查看 @Makefile 了解可用的脚手架指令。

## 工具栏、侧边栏

`?b=0` 隐藏，`?b=1` 显示。不能有闪烁，比如 "先展示工具栏或侧边栏，待页面资源加载好之后再隐藏"，如果需要隐藏，那么页面自始至终应该都看不见它。

## 进度条

进度条总是显示，不隐藏。没有底部边框。纯色蓝色进度条，而非虚线。进度条长度随着文档下载进度一点点步进直到 PDF 文件下载好渲染出来。

## 主题

确保 `?t=1` 是亮色，`?t=0` 是暗色，未设置 `?t=` 查询参数，默认为跟随系统，3 种情况都不许有样式闪烁。

## Markdown 文档

Markdown 文件都用英文命名，命名风格微 Screaming Snake Case (下划线 + 全大写)。如：SYSTEM_AUDIT_AND_OPTIMIZATION_REPORT.md。

## "构建时间" 标签

在 `<html>` 标签中增加 `data-time` 属性，将构建时间赋值其中，使用 date-fns 模块格式化为 `yyyy/MM/dd-HH:mm:ss`，时区为 `Asia/Shanghai`。

## 部署

使用 Docker 容器部署。

执行 `make build`、`make deploy` 这俩指令时，在 `<html>` 标签中增加 `data-time` 属性，将构建时间赋值其中，格式化为 `yyyy/MM/dd-HH:mm:ss`，时区为 `Asia/Shanghai`。

## Makefile

查看 @Makefile 了解当前具体既有指令。

@Makefile 使用 tab 缩进（非空格）缩进。

@Makefile 里用到的指令得兼容 macOS 和 linux 系统。

@Makefile 里管理如下指令，

1. 复合任务：重新部署
2. 删除容器
3. 删除镜像
4. 清理 Docker
5. 构建镜像
6. 运行容器

`make pack` 指令不应该修改任何源码，这个指令应该是会先将项目内文件拷贝到一个临时目录，在临时目录里添加构建时间等，打包后，再将这个临时目录移除。

## 其它

用中文回复对话消息等。

生成的代码需要将原本代码里的所有内容考虑进来生成，包括注释，不能发生 "原本有注释，新生成的代码却把注释删除了" 的情况。

修改/删除文件前，先备份文件，文件名后附加精确到秒的日期，且附加上一份文件是被删除还是修改的标识。

每次新增或改完代码，要做类型检测和功能测试。

---
> Source: [iyowei/pdf-viewer](https://github.com/iyowei/pdf-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
