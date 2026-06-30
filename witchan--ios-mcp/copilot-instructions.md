## ios-mcp

> 本文件用于约束 Codex 在本仓库中的 Git 操作、提交日志生成、版本发布流程。

# AGENTS.md

本文件用于约束 Codex 在本仓库中的 Git 操作、提交日志生成、版本发布流程。

本项目是一个在 GitHub 上维护的开源项目，日常开发场景主要包括：

- 优化已有功能
- 修复 bug
- 新增功能
- 更新文档
- 发布新版本

Codex 在执行 Git 相关操作时，必须严格遵守本文件中的规范。

---

## 一、AGENTS.md 使用规则

### 1. 文件命名规则

如果希望 Codex 自动读取项目规则，文件名应使用：

```text
AGENTS.md
```

不要将本文件改名为：

```text
GIT.md
SKILL.md
AGENT.md
agents.md
```

如果项目中已经存在 `AGENTS.md`，不要新建第二个同名文件，也不要直接覆盖原文件。应该把本文件中的 Git 规范、提交流程、发布流程合并到已有 `AGENTS.md` 中。

### 2. 多个 AGENTS.md 的处理方式

如果仓库里已经有其他 `AGENTS.md`，优先按以下方式处理：

1. 根目录已有 `AGENTS.md`：把本文件内容合并进去，建议放在 `Git 工作流规范` 章节。
2. 子目录已有 `AGENTS.md`：子目录规则只用于该子目录相关任务，根目录仍应保留全局 Git 规范。
3. 如果已有 `AGENTS.md` 中的规则和本文件冲突，以更具体的规则为准；如果都是全局规则，以用户最新明确要求为准。
4. 不要因为新增 Git 规范而删除已有项目规则、构建规则、测试规则或代码风格规则。

### 3. 推荐组织方式

推荐在根目录 `AGENTS.md` 中使用以下结构：

```text
# AGENTS.md

## 项目说明
## 开发规范
## 测试规范
## Git 工作流规范
## 发布流程规范
```

本文件主要提供：

```text
Git 工作流规范
发布流程规范
```

---

## 二、基础原则

### 1. 所有关键操作必须经过用户确认

以下内容必须先展示给用户，并等待用户明确确认后才能继续执行：

- commit 提交日志
- `git add`
- `git commit`
- `git push`
- release 分支创建或切换
- tag 创建
- GitHub Release 标题
- GitHub Release 内容
- GitHub Release 创建
- 合并 release 分支到 main

用户没有明确确认之前，不允许直接执行提交、推送、打 tag、创建 GitHub Release。

### 2. 禁止危险操作

除非用户明确要求，否则禁止执行以下命令：

```bash
git reset --hard
git push --force
git clean -fd
git branch -D
git push origin --delete
```

尤其注意：发布完成后不要删除 release 分支。

---

## 三、分支命名规范

### 1. 分支命名格式

分支名称统一使用：

```text
类型-简短描述
```

类型和描述之间使用 `-` 分隔，不使用 `/`。

### 2. 常用分支类型

```text
feature-xxx     新功能开发
fix-xxx         普通问题修复
hotfix-xxx      紧急问题修复
release-x.x.x   版本发布分支
docs-xxx        文档修改
refactor-xxx    代码重构
chore-xxx       构建、配置、依赖等调整
perf-xxx        性能优化
```

### 3. 分支命名示例

推荐：

```text
feature-ui-query
feature-screenshot-api
fix-click-position
fix-rootless-install
hotfix-launch-crash
release-1.1.0
docs-update-readme
refactor-element-parser
chore-update-makefile
perf-optimize-screenshot
```

不推荐：

```text
feature/ui-query
release/v1.1.0
release-v1.1.0
fix_click_position
Feature-Login
我的分支
update
```

### 4. main 分支规则

`main` 分支代表当前最新稳定版本。

发布完成后，release 分支必须合并回 `main`，保证 `main` 始终对应最新正式发布版本。

---

## 四、提交日志规范

### 1. 提交日志格式

提交日志统一使用：

```text
类型: 提交说明
```

提交说明使用中文，要求简洁、明确。

### 2. 常用提交类型

```text
feat      新功能
fix       修复问题
docs      文档修改
refactor  代码重构
style     代码格式调整，不影响逻辑
test      测试相关
chore     构建、配置、依赖、脚本等杂项
perf      性能优化
revert    回退提交
release   版本发布
```

### 3. 提交日志示例

```text
feat: 添加 UI 元素查询接口
feat: 添加截图接口
fix: 修复点击坐标偏移问题
fix: 修复 rootless 环境安装失败问题
docs: 更新 README 安装说明
refactor: 重构 AXRuntime 元素解析逻辑
chore: 调整 Makefile 打包配置
perf: 优化截图响应速度
release: 发布 v1.1.0
```

### 4. 不推荐的提交日志

```text
update
fix bug
提交代码
修改了一些东西
优化
修复
```

---

## 五、提交代码到仓库工作流

### 1. 触发方式

当用户输入以下任意内容时，进入“提交代码到仓库”流程：

```text
提交代码
提交代码到仓库
提交并推送
git 提交代码到仓库
生成提交日志
```

### 2. 执行流程

#### 第一步：检查当前状态

必须先执行：

```bash
git status
git diff --stat
git diff
```

如果有未跟踪文件，也需要在总结中说明。

#### 第二步：总结当前改动

根据当前未提交代码，总结本次改动。

输出格式：

```text
本次改动摘要：

1. xxx
2. xxx
3. xxx
```

#### 第三步：生成提交日志

根据代码改动自动生成一条符合规范的提交日志。

例如：

```text
fix: 修复点击坐标偏移问题
```

如果本次改动包含多个方向，优先判断主要目的。

判断规则：

```text
新增功能       -> feat
修复 bug       -> fix
文档修改       -> docs
代码重构       -> refactor
构建配置调整   -> chore
性能优化       -> perf
版本发布       -> release
```

#### 第四步：提交前必须让用户确认

在执行任何提交命令之前，必须展示以下内容：

```text
本次改动摘要：

1. xxx
2. xxx
3. xxx

建议提交日志：

fix: 修复点击坐标偏移问题

准备执行：

git add .
git commit -m "fix: 修复点击坐标偏移问题"
git push

是否确认提交并推送？
```

只有用户明确回复确认后，才能继续执行。

#### 第五步：用户确认后提交并推送

用户确认后执行：

```bash
git add .
git commit -m "用户确认后的提交日志"
git push
```

如果当前分支没有关联远程分支，则执行：

```bash
git push -u origin 当前分支名
```

---

## 六、版本发布工作流

### 1. 触发方式

当用户输入以下内容时，进入“版本发布”流程：

```text
发布 1.1.0
发布 v1.1.0
发布版本 1.1.0
release 1.1.0
创建 release 1.1.0
```

### 2. 版本识别规则

如果用户输入：

```text
发布 1.1.0
```

则识别为：

```text
版本号：1.1.0
release 分支：release-1.1.0
tag：v1.1.0
GitHub Release 标题：项目名 v1.1.0
```

如果项目名称可以从仓库名或 README 中明确识别，则使用项目名生成 Release 标题。

例如：

```text
iOS MCP v1.1.0
```

如果项目名称不明确，则在创建 Release 前让用户确认标题。

### 3. release 分支规则

release 分支统一使用：

```text
release-版本号
```

示例：

```text
release-1.0.0
release-1.0.1
release-1.1.0
```

不要使用：

```text
release/v1.1.0
release-v1.1.0
release_1.1.0
```

### 4. tag 规则

tag 统一使用：

```text
v版本号
```

示例：

```text
v1.0.0
v1.0.1
v1.1.0
```

### 5. GitHub Release 标题规则

GitHub Release 标题使用：

```text
项目名 v版本号
```

例如：

```text
iOS MCP v1.1.0
```

---

## 七、发布版本执行流程

### 第一步：检查仓库状态

发布前必须执行：

```bash
git status
git branch --show-current
git fetch --all --tags
```

如果存在未提交代码，必须停止发布流程，并提示用户先提交或处理当前改动。

如果 `git fetch --all --tags` 因网络、DNS、GitHub 连接、认证或沙箱权限失败，必须按以下规则处理：

1. 可以重试一次相同命令。
2. 如果是沙箱网络或 `.git` 写入权限导致失败，可以用提权方式重试。
3. 如果重试后仍失败，必须停止发布流程。
4. 停止时要明确说明当前本地状态、失败命令、失败原因，以及恢复发布时应继续执行的下一条命令。
5. 远端同步没有成功前，不允许继续创建 tag、推送 tag、创建 GitHub Release。

### 第二步：确认目标版本信息

根据用户输入生成版本信息，并展示给用户：

```text
准备发布版本：1.1.0

release 分支：
release-1.1.0

tag：
v1.1.0

GitHub Release 标题：
iOS MCP v1.1.0
```

### 第三步：检查 tag 是否已存在

执行：

```bash
git tag --list "v1.1.0"
```

如果 tag 已存在，必须停止发布流程，并提示用户该版本已经存在，不能重复发布。

### 第四步：创建或切换 release 分支

先切换到 main 并拉取最新代码：

```bash
git switch main
git pull
```

检查 release 分支是否存在：

```bash
git branch --list "release-1.1.0"
git branch -r | grep "origin/release-1.1.0"
```

如果本地和远程都不存在，则从 main 创建 release 分支：

```bash
git switch -c release-1.1.0
```

如果远程已经存在，则切换到已有 release 分支：

```bash
git switch release-1.1.0
git pull
```

### 第五步：检查并更新版本号

根据项目实际情况，检查可能包含版本号的文件，例如：

```text
README.md
control
package.json
Info.plist
Makefile
其他项目配置文件
```

如果发现需要更新版本号，必须先展示修改计划，等待用户确认后再修改。

如果没有需要修改的版本号文件，则不要强行修改。

### 第六步：构建正式包并检查 Release assets

创建 GitHub Release 前必须构建并检查三种正式包：

```bash
printf "1\n1\n" | ./build.sh
printf "2\n1\n" | ./build.sh
printf "3\n1\n" | ./build.sh
```

构建完成后必须检查以下文件存在：

```text
rootful：packages/com.witchan.ios-mcp_版本号_iphoneos-arm.deb
rootless：packages/com.witchan.ios-mcp_版本号_iphoneos-arm64.deb
roothide：packages/com.witchan.ios-mcp_版本号_iphoneos-arm64e.deb
```

如果任一文件不存在，必须停止发布流程，提示用户先构建正式包。不要上传旧版本包、debug 包、带错误版本号的包，或只上传部分 assets。

Release assets 内容必须按以下格式写入 GitHub Release：

```markdown
## Release assets

rootful: com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
rootless: com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
roothide: com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb
```

### 第七步：生成 Release 内容

根据上一个 tag 到当前版本之间的提交记录生成版本说明。

先查找最近 tag：

```bash
git tag --sort=-v:refname
```

查看提交记录：

```bash
git log 上一个tag..HEAD --oneline
```

生成 Release 内容时，必须默认使用中英文双语格式：

```markdown
## 更新内容 / What's New

### 中文

- xxx
- xxx

### English

- xxx
- xxx

## 修复问题 / Fixes

### 中文

- xxx
- xxx

### English

- xxx
- xxx

## Release assets

rootful: com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
rootless: com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
roothide: com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb
```

默认不要添加 `说明 / Notes` 部分。只有用户明确要求时，才可以增加说明部分。

如果本次版本主要是 bug 修复，内容仍然使用中英文双语格式，但重点放在：

```markdown
## 修复问题 / Fixes
```

如果本次版本主要是功能优化，内容仍然使用中英文双语格式，但重点放在：

```markdown
## 更新内容 / What's New
```

Release 内容不得只写中文，也不得只写英文。

### 第八步：Release 内容必须让用户确认

创建 GitHub Release 之前，必须展示完整 Release 内容。

输出格式：

```text
准备发布版本：1.1.0

release 分支：
release-1.1.0

tag：
v1.1.0

GitHub Release 标题：
iOS MCP v1.1.0

Release 内容：

## 更新内容 / What's New

### 中文

- xxx
- xxx

### English

- xxx
- xxx

## 修复问题 / Fixes

### 中文

- xxx

### English

- xxx

## Release assets

rootful: com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
rootless: com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
roothide: com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb

Release assets：
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb

准备执行的操作：

1. 创建或切换 release-1.1.0 分支
2. 检查并更新版本号
3. 提交 release 改动
4. 推送 release 分支
5. 合并 release-1.1.0 到 main
6. 在 main 上创建 tag v1.1.0
7. 推送 main
8. 推送 tag
9. 构建并检查三种正式包
10. 通过 GitHub REST API 创建/更新 GitHub Release
11. 上传三个 Release assets

是否确认发布？
```

只有用户明确确认后，才能继续执行。

---

## 八、用户确认后的发布操作

### 1. 在 release 分支提交版本改动

如果有版本号文件或发布相关文件被修改，执行：

```bash
git add .
git commit -m "release: 发布 v1.1.0"
```

如果没有文件修改，则不要强行创建 commit。

### 2. 推送 release 分支

```bash
git push -u origin release-1.1.0
```

如果推送失败，必须按以下规则处理：

1. 可以重试一次相同命令。
2. 如果是网络或沙箱问题，可以用提权方式重试。
3. 如果仍然失败，必须停止后续流程，不允许继续合并 main、创建 tag 或创建 GitHub Release。
4. 停止时要说明当前分支、当前提交、是否已本地提交、远端是否已更新，以及恢复时应该执行的下一条命令。

### 3. 合并 release 分支到 main

发布版本时，release 分支确认完成后，必须合并回 main。

执行前再次展示：

```text
准备将 release-1.1.0 合并到 main：

git switch main
git pull
git merge --no-ff release-1.1.0 -m "release: 合并 v1.1.0 到 main"
git push origin main

是否确认合并到 main？
```

用户确认后执行：

```bash
git switch main
git pull
git merge --no-ff release-1.1.0 -m "release: 合并 v1.1.0 到 main"
git push origin main
```

如果 `git push origin main` 失败，必须停止后续 tag 和 GitHub Release 流程。只有 main 成功推送到远端后，才能继续创建 tag。

### 4. 在 main 上创建 tag

合并到 main 后，在 main 分支上创建 tag：

```bash
git tag -a v1.1.0 -m "release: 发布 v1.1.0"
```

推送 tag：

```bash
git push origin v1.1.0
```

如果 tag 推送失败，必须停止 GitHub Release 创建流程。只有 tag 成功推送到远端后，才能创建 GitHub Release。

### 5. 构建正式包和检查 assets

```bash
printf "1\n1\n" | ./build.sh
printf "2\n1\n" | ./build.sh
printf "3\n1\n" | ./build.sh
ls -l packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
ls -l packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
ls -l packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb
```

### 6. 创建 GitHub Release

本项目创建 GitHub Release 时，优先使用 GitHub REST API，不默认依赖 `gh` 命令。

GitHub REST API 规则：

1. 使用 `git credential fill` 从本地 Git 凭据中读取 GitHub token。
2. 不允许在日志、终端输出或最终回复中打印 token。
3. 如果无法读取 GitHub token，必须停止并提示用户配置 GitHub 凭据。
4. 先通过 `GET /repos/witchan/ios-mcp/releases/tags/v1.1.0` 检查 Release 是否存在。
5. 如果 Release 不存在，使用 `POST /repos/witchan/ios-mcp/releases` 创建。
6. 如果 Release 已存在，使用 `PATCH /repos/witchan/ios-mcp/releases/{release_id}` 更新标题和内容。
7. 上传 assets 前检查同名 asset 是否存在。
8. 如果同名 asset 已存在，必须在用户确认过“覆盖/更新 assets”的前提下，先删除旧 asset，再上传新 asset。
9. 上传以下三个 assets：

```text
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb
```

创建/更新完成后，必须输出 GitHub Release URL 和三个 assets 的下载链接。

不要在没有用户确认 Release 内容的情况下创建 GitHub Release。

---

## 九、release 分支保留规则

发布完成后，必须保留 release 分支。

例如发布 `1.1.0` 后，需要保留：

```text
release-1.1.0
v1.1.0
GitHub Release：iOS MCP v1.1.0
```

禁止自动执行：

```bash
git branch -d release-1.1.0
git branch -D release-1.1.0
git push origin --delete release-1.1.0
```

除非用户明确说“删除 release 分支”，否则不能删除任何 release 分支。

---

## 十、发布完成后的输出格式

发布完成后，输出：

```text
发布完成：

版本：1.1.0
release 分支：release-1.1.0
tag：v1.1.0
GitHub Release：iOS MCP v1.1.0
main 分支：已同步到 v1.1.0
release 分支：已保留
Release assets：
rootful：com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb
rootless：com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb
roothide：com.witchan.ios-mcp_1.1.0_iphoneos-arm64e.deb
```

---

## 十一、常见场景处理规则

本章节用于判断提交日志类型和 Release 内容归类。正式 GitHub Release 仍必须遵守第七步的中英文双语格式，并包含 `Release assets`。

### 1. 优化功能后发版

如果用户说：

```text
优化截图功能，发布 1.1.0
```

提交日志优先使用：

```text
perf: 优化截图功能
```

如果只是体验优化，不涉及性能，也可以使用：

```text
feat: 优化截图功能
```

Release 内容优先放到：

```markdown
## 优化内容

- 优化截图功能
```

### 2. 修复 bug 后发版

如果用户说：

```text
修复点击坐标偏移问题，发布 1.1.1
```

提交日志优先使用：

```text
fix: 修复点击坐标偏移问题
```

Release 内容优先放到：

```markdown
## 修复问题

- 修复点击坐标偏移问题
```

### 3. 新增功能后发版

如果用户说：

```text
添加截图接口，发布 1.2.0
```

提交日志优先使用：

```text
feat: 添加截图接口
```

Release 内容优先放到：

```markdown
## 更新内容

- 添加截图接口
```

### 4. 文档修改

如果只是更新 README、安装说明、使用说明，提交日志使用：

```text
docs: 更新 README 使用说明
```

---

## 十二、BigBoss 上架 / 更新流程

### 1. 触发方式

当用户输入以下任意内容时，进入“BigBoss 上架 / 更新”流程：

```text
发布到bigboss
发布到 BigBoss
上架到bigboss
上架到 BigBoss
提交到bigboss
提交到 BigBoss
更新bigboss
更新 BigBoss
```

### 2. 基础规则

BigBoss 是第三方仓库，提交后会进入 BigBoss 审核流程。该操作属于对外可见提交，必须先让用户确认。

本项目 BigBoss 更新表单脚本固定使用：

```text
脚本：scripts/submit_bigboss_update.py
默认 Package Name：iOS MCP
Your Name：witchan
Email：witchan028@126.com
```

脚本运行时只传入：

```text
版本号
Changes Made（必须中英文双语）
deb 路径
```

rootful 和 rootless 提交到 BigBoss 时，BigBoss 表单 `Package Name` 使用默认值：

```text
iOS MCP
```

BigBoss 更新流程只提交 rootful 和 rootless 包，不提交 roothide 包，也不为 BigBoss 临时生成 `com.witchan.ios-mcp-roothide` 包。

BigBoss 的 `Changes Made` 默认必须使用简短中英文双语，不要写成长篇 Release notes。必须先中文、再英文。推荐格式：

```text
中文：xxx。
EN: xxx.
```

如果用户只提供中文或英文，必须先补齐另一种语言，并在提交前展示给用户确认。

### 3. 用户确认内容

触发 BigBoss 流程后，必须先让用户确认以下内容：

准备提交到 BigBoss：

版本号：
1.1.0

Changes Made：
中文：xxx。
EN: xxx.

rootful deb：
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb

rootless deb：
packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb

准备执行：

提交 BigBoss 表单：

```bash
python3 scripts/submit_bigboss_update.py \
  --version 1.1.0 \
  --changes "中文：xxx。
EN: xxx." \
  --response-out .codex-session-data/bigboss_update_1.1.0_rootful_response.html \
  packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm.deb \
  --submit

python3 scripts/submit_bigboss_update.py \
  --version 1.1.0 \
  --changes "中文：xxx。
EN: xxx." \
  --response-out .codex-session-data/bigboss_update_1.1.0_rootless_response.html \
  packages/com.witchan.ios-mcp_1.1.0_iphoneos-arm64.deb \
  --submit
```

是否确认提交到 BigBoss 审核？

只有用户明确确认后，才能执行提交。

### 4. deb 文件规则

默认提交 BigBoss 支持的两个包：

```text
rootful：packages/com.witchan.ios-mcp_版本号_iphoneos-arm.deb
rootless：packages/com.witchan.ios-mcp_版本号_iphoneos-arm64.deb
```

BigBoss 流程不提交普通 roothide Release 包：

```text
packages/com.witchan.ios-mcp_版本号_iphoneos-arm64e.deb
```

BigBoss 流程也不提交或生成 roothide 专用包：

```text
packages/com.witchan.ios-mcp-roothide_版本号_iphoneos-arm64e.deb
```

### 5. 执行前检查

用户确认后，执行前必须检查：

```bash
git status --short --branch
ls -l packages/com.witchan.ios-mcp_版本号_iphoneos-arm.deb
ls -l packages/com.witchan.ios-mcp_版本号_iphoneos-arm64.deb
git diff -- control
python3 scripts/submit_bigboss_update.py --help
```

如果 rootful 或 rootless deb 不存在，必须停止并提示用户先构建正式包。
不允许提交旧版本或错误路径的包。
如果 `git diff -- control` 显示有未确认的包名或版本改动，必须先让用户确认如何处理后再提交 BigBoss。

### 6. 执行提交

确认无误后，依次提交：

```bash
python3 scripts/submit_bigboss_update.py \
  --version 版本号 \
  --changes "用户确认后的中英文双语 Changes Made" \
  --response-out .codex-session-data/bigboss_update_版本号_rootful_response.html \
  packages/com.witchan.ios-mcp_版本号_iphoneos-arm.deb \
  --submit

python3 scripts/submit_bigboss_update.py \
  --version 版本号 \
  --changes "用户确认后的中英文双语 Changes Made" \
  --response-out .codex-session-data/bigboss_update_版本号_rootless_response.html \
  packages/com.witchan.ios-mcp_版本号_iphoneos-arm64.deb \
  --submit
```

如果前面的提交已成功、后面的提交失败，必须明确告诉用户 BigBoss 可能已经收到哪些包，当前处于部分提交状态。不要自动重复提交成功的包，除非用户明确确认重试。

### 7. 完成后的输出格式

提交完成后输出：

```text
BigBoss 提交完成：

版本：1.1.0
rootful：已提交
rootless：已提交
审核状态：等待 BigBoss 审核
响应记录：
.codex-session-data/bigboss_update_1.1.0_rootful_response.html
.codex-session-data/bigboss_update_1.1.0_rootless_response.html
```

---

## 十三、最终要求

Codex 必须始终遵守以下要求：

1. 如果希望 Codex 自动读取规则，文件名使用 `AGENTS.md`。
2. 如果项目已有 `AGENTS.md`，把本文件内容合并进去，不要覆盖已有规则。
3. 分支名使用 `-` 分隔，不使用 `/`。
4. release 分支使用 `release-版本号`。
5. tag 使用 `v版本号`。
6. commit 日志必须先让用户确认。
7. Release 内容必须先让用户确认。
8. 发布完成后必须合并 release 分支到 main。
9. 发布完成后必须保留 release 分支。
10. 未经用户确认，不允许提交、推送、打 tag、创建 GitHub Release。
11. 未经用户确认，不允许提交 BigBoss 更新表单。
12. GitHub Release 默认使用 GitHub REST API 创建/更新，并上传 rootful、rootless、roothide 三个正式包。
13. GitHub Release 内容默认使用中英文双语格式，并包含 Release assets；默认不添加说明部分。
14. GitHub 网络或沙箱失败时必须停止危险后续步骤，明确说明本地状态和恢复命令。
15. BigBoss 默认只提交 rootful、rootless 两个包；不提交 roothide 包，也不为 BigBoss 生成 roothide 专用包。

---
> Source: [witchan/ios-mcp](https://github.com/witchan/ios-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
