## clawhub-skill-submission-guide

> ClawHub Skill 提交指南 - 从零开始创建、测试、发布 Skill 到 ClawHub 官方市场，包含完整的自动化流程、手动步骤、常见坑点解决方案，适用于无经验用户和经验开发者


# ClawHub Skill 提交完全指南

**版本**：v1.0.0
**更新日期**：2026-03-25
**目标用户**：无经验小白 → 成功发布 Skill 的完整路径

---

## 📋 目录

1. [流程概览](#流程概览)
2. [准备工作](#准备工作)
3. [自动化步骤（AI 完成）](#自动化步骤ai-完成)
4. [手动步骤（用户完成）](#手动步骤用户完成))
5. [常见坑点与解决方案](#常见坑点与解决方案)
6. [完整操作流程](#完整操作流程)
7. [检查清单](#检查清单)

---

## 流程概览

### 整体流程图

```
准备阶段（自动化）
    ↓
GitHub 仓库创建（自动化 + 手动确认）
    ↓
文档准备（自动化）
    ↓
敏感信息脱敏（自动化）
    ↓
推送到 GitHub（自动化）
    ↓
创建 GitHub Release（手动）
    ↓
提交到 ClawHub（手动）
    ↓
等待审核（被动）
    ↓
审核通过（完成）
```

### 角色分工

| 阶段 | AI 助手 | 用户作者 |
|------|---------|----------|
| 准备工作 | ✅ 检查 Skill 文件完整性 | - |
| 文档准备 | ✅ 生成中英文文档 | - |
| 敏感信息脱敏 | ✅ 自动检测和脱敏 | - |
| GitHub 仓库 | ⚠️ 建议创建信息，需用户确认 | ✅ 创建仓库 |
| 推送代码 | ✅ 自动推送 | - |
| GitHub Release | - | ✅ 创建 Release 和上传截图 |
| ClawHub 提交 | ⚠️ 生成提交表单内容 | ✅ 填写并提交 |
| 等待审核 | - | ✅ 跟进审核状态 |

---

## 准备工作

### 1. 检查 Skill 文件完整性

#### AI 自动检查清单

**必需文件**：
- ✅ `SKILL.md` - Skill 核心配置文件
- ✅ 至少一个实现文件（脚本/代码）

**推荐文件**：
- ✅ `README.md` - 中文使用文档
- ✅ `README_EN.md` - 英文使用文档
- ✅ `LICENSE` - 开源协议（推荐 MIT）
- ✅ `CHANGELOG.md` - 版本更新日志
- ✅ `.gitignore` - Git 忽略配置

#### 检查命令（AI 执行）

```bash
# 列出 Skill 目录下的所有文件
ls -la ~/.workbuddy/skills/your-skill-name/

# 检查必需文件是否存在
test -f ~/.workbuddy/skills/your-skill-name/SKILL.md && echo "✅ SKILL.md 存在" || echo "❌ SKILL.md 缺失"
```

### 2. 检查 SKILL.md 配置

#### 必需字段

```yaml
---
description: 你的 Skill 描述文字，100 字以内
---
```

**检查要点**：
- ✅ `description` 字段必须存在
- ✅ 描述简洁明了，100 字以内
- ✅ 避免特殊字符和表情符号
- ✅ 使用中性、专业的语言

#### 示例

```yaml
---
description: 3D 词云可视化工具 - 将对话历史或其他文本数据自动转换为炫酷的 3D 地球词云，支持多格式文件导入（JSON/MD/TXT），自动中文分词和词频统计，生成 TOP30 高频词的 3D 可视化效果
---
```

### 3. 检查 Skill 实现

#### 实现方式

ClawHub 支持多种 Skill 实现方式：

1. **Python 脚本**（最常见）
   - 单文件或多文件
   - 必须有 `__main__` 入口
   - 依赖通过 `requirements.txt` 或 `pyproject.toml`

2. **Shell 脚本**
   - Bash 脚本
   - 必须有执行权限

3. **Node.js 脚本**
   - JavaScript/TypeScript
   - 依赖通过 `package.json`

4. **纯配置型 Skill**
   - 主要是 SKILL.md 和文档
   - 实现由 WorkBuddy 核心 API 完成

---

## 自动化步骤（AI 完成）

### 步骤 1：生成完整文档

#### AI 操作

```python
# 生成 README.md（中文）
def generate_readme_cn(skill_info):
    template = """# {name}

{description}

## 功能特性

{features}

## 使用方法

{usage}

## 技术栈

{tech_stack}

## 许可证

MIT License

## English Version

[English Documentation](README_EN.md)
"""
    return template.format(**skill_info)

# 生成 README_EN.md（英文）
def generate_readme_en(skill_info):
    template = """# {name_en}

{description_en}

## Features

{features_en}

## Usage

{usage_en}

## Tech Stack

{tech_stack_en}

## License

MIT License

## 中文文档

[中文文档](README.md)
"""
    return template.format(**skill_info)
```

#### 输出内容

- ✅ `README.md` - 完整的中文使用文档
- ✅ `README_EN.md` - 完整的英文使用文档

### 步骤 2：生成 LICENSE

#### AI 操作

```python
# 生成 MIT License
def generate_mit_license(author, year):
    template = f"""MIT License

Copyright (c) {year} {author}

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
"""
    return template
```

#### 输出内容

- ✅ `LICENSE` - MIT 开源协议文件

### 步骤 3：生成 CHANGELOG.md

#### AI 操作

```python
# 生成版本更新日志
def generate_changelog(version, date):
    template = f"""# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [{version}] - {date}

### Added
- Initial release
- {feature_list}

### Features
- {detailed_features}
"""
    return template
```

#### 输出内容

- ✅ `CHANGELOG.md` - 版本更新日志

### 步骤 4：敏感信息检测与脱敏

#### AI 自动检测

**检测目标**：

1. **硬编码路径**
   - `.workbuddy/memory/`
   - `MEMORY.md`
   - `C:\Users\Administrator\`
   - `/home/username/`

2. **敏感信息**
   - API Keys
   - 密码
   - 邮箱地址
   - 个人信息

3. **工具名称硬编码**
   - 具体的工具名称（如 `python`、`npm`）
   - 绝对路径的脚本路径

#### 脱敏规则

```python
# 脱敏映射表
DESENSITIZE_MAP = {
    '.workbuddy/memory/': 'WorkBuddy 工作记忆目录',
    'MEMORY.md': '长期记忆文件',
    'C:\\Users\\Administrator\\': '~/',
    '/home/username/': '~/',
    'sk-proj-': 'sk-proj-***',
}
```

#### AI 操作

```python
def desensitize_content(content):
    for sensitive, replacement in DESENSITIZE_MAP.items():
        content = content.replace(sensitive, replacement)
    return content
```

### 步骤 5：生成 GitHub 仓库信息

#### AI 操作

```python
def generate_github_info(skill_name, github_username):
    return {
        'repository_name': skill_name.lower().replace(' ', '-'),
        'repository_url': f'https://github.com/{github_username}/{skill_name.lower().replace(" ", "-")}',
        'repository_description': 'Your Skill description from SKILL.md',
        'visibility': 'public',
        'default_branch': 'main',
    }
```

#### 输出内容

- ✅ GitHub 仓库名称
- ✅ GitHub 仓库 URL
- ✅ 仓库描述
- ✅ 推荐分支名称

### 步骤 6：准备 Git 推送命令

#### AI 操作

```python
def generate_git_commands(github_info):
    commands = f"""
# 步骤 1：初始化 Git 仓库
cd ~/.workbuddy/skills/{skill_name}
git init

# 步骤 2：添加所有文件
git add .

# 步骤 3：提交初始代码
git commit -m "Initial commit - {skill_name} v1.0.0"

# 步骤 4：添加远程仓库
git remote add origin {github_info['repository_url']}.git

# 步骤 5：重命名分支为 main
git branch -M main

# 步骤 6：推送到 GitHub
git push -u origin main
"""
    return commands
```

---

## 手动步骤（用户完成）

### 步骤 1：创建 GitHub 仓库

#### 操作前确认

**前提条件**：
- ✅ 拥有 GitHub 账号
- ✅ 已登录 GitHub
- ✅ 已配置 Git 用户信息

**检查 Git 配置**：

```bash
# 检查 Git 用户名
git config user.name

# 检查 Git 邮箱
git config user.email

# 如果没有配置，执行以下命令
git config --global user.name "Your GitHub Username"
git config --global user.email "your-github-email@example.com"
```

#### 详细操作步骤

**方法一：通过 GitHub Web 界面创建**

1. **访问 GitHub**
   - 打开浏览器，访问 https://github.com

2. **创建新仓库**
   - 点击右上角 "+" 图标
   - 选择 "New repository"

3. **填写仓库信息**
   ```
   Repository name: {AI 提供的仓库名称}
   Description: {AI 提供的仓库描述}
   Public ☑️ (必须公开，ClawHub 无法访问私有仓库)
   ❌ Add a README file (不要勾选，AI 已准备)
   ❌ Add .gitignore (不要勾选，AI 已准备)
   ❌ Choose a license (不要勾选，AI 已准备)
   ```

4. **点击 "Create repository"**

5. **记录仓库 URL**
   ```
   https://github.com/{your-username}/{repository-name}.git
   ```

**方法二：通过 GitHub CLI 创建**

```bash
# 安装 GitHub CLI（如果未安装）
# Windows: choco install gh
# macOS: brew install gh
# Linux: 根据发行版安装

# 登录 GitHub
gh auth login

# 创建仓库
gh repo create {repository-name} --public --description "{description}"
```

#### 坑点提醒

⚠️ **坑点 1：仓库名称大小写**
- **问题**：GitHub URL 不区分大小写，但文件系统区分
- **解决**：全部使用小写，用连字符连接

⚠️ **坑点 2：仓库可见性**
- **问题**：私有仓库 ClawHub 无法访问
- **解决**：必须选择 Public

⚠️ **坑点 3：邮箱隐私设置**
- **问题**：GitHub 邮箱隐私设置会阻止推送
- **解决**：设置 Git 邮箱为 `noreply@github.com` 或在 GitHub 设置中公开邮箱

### 步骤 2：推送代码到 GitHub

#### AI 准备的命令

AI 会为你准备好完整的推送命令，复制执行即可。

#### 执行推送

**场景 1：空仓库**

```bash
cd ~/.workbuddy/skills/your-skill-name
git init
git add .
git commit -m "Initial commit - Your Skill v1.0.0"
git remote add origin https://github.com/your-username/your-repository.git
git branch -M main
git push -u origin main
```

**场景 2：非空仓库（远程已有内容）**

```bash
cd ~/.workbuddy/skills/your-skill-name
git init
git add .
git commit -m "Initial commit - Your Skill v1.0.0"
git remote add origin https://github.com/your-username/your-repository.git
git branch -M main
git pull origin main --allow-unrelated-histories
git push -u origin main
```

#### 坑点提醒

⚠️ **坑点 4：远程仓库非空**
- **问题**：GitHub 创建时勾选了 README，远程仓库不空
- **解决**：使用 `git pull --allow-unrelated-histories` 合并

⚠️ **坑点 5：分支名称冲突**
- **问题**：默认分支是 `master`，现代标准是 `main`
- **解决**：使用 `git branch -M main` 重命名

⚠️ **坑点 6：认证失败**
- **问题**：Git 推送时要求认证
- **解决**：
  - 使用 GitHub CLI：`gh auth login`
  - 使用 SSH 密钥：`git remote set-url origin git@github.com:username/repo.git`

### 步骤 3：创建 GitHub Release

#### 操作前准备

**需要准备的文件**：
- ✅ Skill 截图（PNG/JPG 格式，建议尺寸 1200x800 或更高）
- ✅ 截图内容：展示核心功能、界面清晰、专业美观

#### 详细操作步骤

1. **访问 GitHub 仓库**
   - 打开浏览器，访问仓库 URL
   - 例如：https://github.com/0xf4vul/3d-wordcloud-visualizer

2. **进入 Releases 页面**
   - 点击右侧 "Releases"
   - 点击 "Create a new release"

3. **填写 Release 信息**
   ```
   Tag: v1.0.0
   Target: main
   Title: v1.0.0 - Initial Release
   Description: {AI 提供的 CHANGELOG.md 内容}
   ```

4. **上传附件**
   - 点击 "Attach binaries by dropping them here or selecting them"
   - 选择准备好的截图文件
   - 建议文件名：`screenshot-1-feature-demo.png`

5. **点击 "Publish release"**

#### AI 准备的内容

AI 会为你准备好：
- ✅ Release 标签：`v1.0.0`
- ✅ Release 标题：`v1.0.0 - Initial Release`
- ✅ Release 描述：CHANGELOG.md 内容

#### 坑点提醒

⚠️ **坑点 7：Release 标签格式**
- **问题**：标签必须以 `v` 开头，否则 ClawHub 无法识别
- **解决**：使用 `v1.0.0`，不是 `1.0.0`

⚠️ **坑点 8：截图尺寸**
- **问题**：截图过大或过小，影响展示效果
- **解决**：推荐 1200x800，文件大小 < 5MB

⚠️ **坑点 9：Release 描述格式**
- **问题**：纯文本格式不友好，Markdown 格式更好
- **解决**：使用 Markdown 格式，突出关键信息

### 步骤 4：提交到 ClawHub

#### 操作前准备

**需要的账号信息**：
- ✅ ClawHub 账号（可能需要注册）
- ✅ GitHub 账号（用于授权登录）

**需要的信息**：
- ✅ Skill 名称
- ✅ Skill 描述（SKILL.md 中的 description）
- ✅ GitHub 仓库 URL
- ✅ 版本号：`v1.0.0`

#### 详细操作步骤

1. **访问 ClawHub 官网**
   - 打开浏览器，访问 ClawHub 官网
   - 使用 GitHub 账号登录

2. **进入提交页面**
   - 点击 "Submit a Skill" 或 "发布 Skill"

3. **填写 Skill 信息**
   ```
   Skill 名称: {AI 提供的 Skill 名称}
   Skill 描述: {AI 提供的 Skill 描述}
   GitHub 仓库: {AI 提供的仓库 URL}
   版本号: v1.0.0
   截图: {上传准备好的截图}
   分类: {选择合适的分类}
   ```

4. **提交审核**
   - 检查所有信息
   - 点击 "Submit" 或 "提交审核"

#### AI 准备的内容

AI 会为你准备好：
- ✅ Skill 名称
- ✅ Skill 描述（从 SKILL.md 提取）
- ✅ GitHub 仓库 URL
- ✅ 版本号：`v1.0.0`
- ✅ 推荐分类（根据 Skill 功能自动判断）

#### 坑点提醒

⚠️ **坑点 10：Skill 描述过长**
- **问题**：超过字符限制，提交失败
- **解决**：使用 SKILL.md 中的 description（通常 < 100 字）

⚠️ **坑点 11：仓库访问权限**
- **问题**：私有仓库 ClawHub 无法访问
- **解决**：确认仓库为 Public

⚠️ **坑点 12：分类选择错误**
- **问题**：分类错误影响 Skill 曝光
- **解决**：根据 Skill 功能选择最相关的分类

### 步骤 5：等待审核

#### 审核流程

**审核周期**：1-3 个工作日

**审核内容**：
- ✅ 代码质量
- ✅ 文档完整性
- ✅ 功能可用性
- ✅ 安全性检查
- ✅ 敏感信息检查

#### 等待期间

**可以做的事**：
- ✅ 检查 GitHub 仓库是否正常访问
- ✅ 检查文档是否完整
- ✅ 检查截图是否清晰
- ✅ 准备后续版本更新

**不要做的事**：
- ❌ 不要修改已提交的代码（可能影响审核）
- ❌ 不要删除或移动仓库
- ❌ 不要频繁联系审核人员

#### 审核结果

**审核通过**：
- ✅ Skill 在 ClawHub 官方市场展示
- ✅ 用户可以通过 WorkBuddy 直接安装使用
- ✅ 会收到邮件通知

**审核失败**：
- ❌ 会收到拒绝原因和修改建议
- ✅ 根据建议修改后可重新提交
- ✅ 最多可以重新提交 3 次

#### 坑点提醒

⚠️ **坑点 13：审核期间修改代码**
- **问题**：修改代码可能导致审核失败
- **解决**：审核期间保持仓库不变

⚠️ **坑点 14：审核失败后未修复**
- **问题**：直接重新提交，未修复问题
- **解决**：根据拒绝原因修改后再提交

---

## 常见坑点与解决方案

### Git 相关

| 坑点 | 问题表现 | 解决方案 |
|------|----------|----------|
| 坑点 1 | 仓库名称大小写问题 | 全部小写，用连字符连接 |
| 坑点 2 | 私有仓库 ClawHub 无法访问 | 必须选择 Public |
| 坑点 3 | 邮箱隐私设置阻止推送 | 设置邮箱为 `noreply@github.com` 或公开邮箱 |
| 坑点 4 | 远程仓库非空导致推送失败 | 使用 `git pull --allow-unrelated-histories` |
| 坑点 5 | 分支名称冲突 | 使用 `git branch -M main` 重命名 |
| 坑点 6 | Git 认证失败 | 使用 GitHub CLI 或 SSH 密钥 |

### GitHub Release 相关

| 坑点 | 问题表现 | 解决方案 |
|------|----------|----------|
| 坑点 7 | Release 标签格式错误 | 使用 `v1.0.0`，不是 `1.0.0` |
| 坑点 8 | 截图尺寸不合适 | 推荐 1200x800，文件大小 < 5MB |
| 坑点 9 | Release 描述格式不友好 | 使用 Markdown 格式 |

### ClawHub 提交相关

| 坑点 | 问题表现 | 解决方案 |
|------|----------|----------|
| 坑点 10 | Skill 描述过长 | 使用 SKILL.md 中的 description（< 100 字） |
| 坑点 11 | 仓库访问权限问题 | 确认仓库为 Public |
| 坑点 12 | 分类选择错误 | 根据 Skill 功能选择最相关的分类 |

### 审核相关

| 坑点 | 问题表现 | 解决方案 |
|------|----------|----------|
| 坑点 13 | 审核期间修改代码 | 审核期间保持仓库不变 |
| 坑点 14 | 审核失败后未修复 | 根据拒绝原因修改后再提交 |

---

## 完整操作流程

### 阶段 1：准备工作（AI 自动化）

**AI 动作**：
1. ✅ 检查 Skill 文件完整性
2. ✅ 检查 SKILL.md 配置
3. ✅ 检查 Skill 实现
4. ✅ 生成中英文文档（README.md + README_EN.md）
5. ✅ 生成 LICENSE（MIT License）
6. ✅ 生成 CHANGELOG.md
7. ✅ 敏感信息检测与脱敏
8. ✅ 生成 GitHub 仓库信息
9. ✅ 准备 Git 推送命令

**用户动作**：
- 确认 AI 生成的文档内容
- 确认 AI 脱敏后的代码

**预计时间**：5-10 分钟

### 阶段 2：GitHub 仓库创建（用户手动）

**用户动作**：
1. 访问 GitHub
2. 创建新仓库（使用 AI 提供的信息）
3. 记录仓库 URL

**AI 动作**：
- 提供详细的创建步骤说明
- 提供仓库配置建议

**预计时间**：3-5 分钟

### 阶段 3：推送代码（AI 自动化）

**AI 动作**：
1. 初始化 Git 仓库
2. 添加所有文件
3. 提交初始代码
4. 添加远程仓库
5. 推送到 GitHub

**用户动作**：
- 复制并执行 AI 提供的 Git 命令

**预计时间**：1-2 分钟

### 阶段 4：创建 GitHub Release（用户手动）

**用户动作**：
1. 访问 GitHub 仓库
2. 进入 Releases 页面
3. 创建新 Release（使用 AI 提供的信息）
4. 上传截图

**AI 动作**：
- 提供 Release 标签、标题、描述
- 提供截图建议

**预计时间**：5-10 分钟

### 阶段 5：提交到 ClawHub（用户手动）

**用户动作**：
1. 访问 ClawHub 官网
2. 填写 Skill 信息（使用 AI 提供的内容）
3. 提交审核

**AI 动作**：
- 提供 Skill 名称、描述、仓库 URL
- 提供分类建议

**预计时间**：5-10 分钟

### 阶段 6：等待审核（被动）

**用户动作**：
- 等待审核结果
- 检查邮件通知

**AI 动作**：
- 提供审核期间的建议
- 提供审核失败的处理方案

**预计时间**：1-3 个工作日

---

## 检查清单

### 提交前检查（AI 自动化）

- [ ] SKILL.md 包含 description 字段
- [ ] description 长度 < 100 字
- [ ] README.md（中文）已生成
- [ ] README_EN.md（英文）已生成
- [ ] LICENSE（MIT）已生成
- [ ] CHANGELOG.md 已生成
- [ ] 敏感信息已脱敏
- [ ] 所有文件已准备好

### GitHub 仓库检查（用户手动）

- [ ] GitHub 账号已登录
- [ ] Git 用户信息已配置
- [ ] 仓库已创建
- [ ] 仓库为 Public
- [ ] 仓库 URL 正确

### 代码推送检查（AI 自动化）

- [ ] Git 仓库已初始化
- [ ] 所有文件已添加
- [ ] 初始提交已创建
- [ ] 远程仓库已关联
- [ ] 分支已重命名为 main
- [ ] 代码已推送到 GitHub

### Release 检查（用户手动）

- [ ] Release 标签格式正确（v1.0.0）
- [ ] Release 标题已填写
- [ ] Release 描述已填写（Markdown 格式）
- [ ] 截图已上传
- [ ] Release 已发布

### ClawHub 提交检查（用户手动）

- [ ] ClawHub 账号已登录
- [ ] Skill 名称已填写
- [ ] Skill 描述已填写（< 100 字）
- [ ] GitHub 仓库 URL 已填写
- [ ] 版本号已填写（v1.0.0）
- [ ] 截图已上传
- [ ] 分类已选择
- [ ] 表单已提交

### 审核期间检查（用户手动）

- [ ] GitHub 仓库可正常访问
- [ ] 文档完整且可读
- [ ] 截图清晰可见
- [ ] 仓库保持不变

---

## 快速参考

### 常用命令

**Git 配置**：
```bash
git config --global user.name "Your GitHub Username"
git config --global user.email "your-github-email@example.com"
```

**GitHub CLI 登录**：
```bash
gh auth login
```

**推送代码**：
```bash
cd ~/.workbuddy/skills/your-skill-name
git init
git add .
git commit -m "Initial commit - v1.0.0"
git remote add origin https://github.com/username/repo.git
git branch -M main
git push -u origin main
```

### 常见问题

**Q: 如何查看 Git 配置？**
```bash
git config --list
```

**Q: 如何撤销最后一次提交？**
```bash
git reset --soft HEAD~1
```

**Q: 如何修改远程仓库 URL？**
```bash
git remote set-url origin https://github.com/username/new-repo.git
```

**Q: 如何查看 Git 状态？**
```bash
git status
```

**Q: 如何查看提交历史？**
```bash
git log
```

---

## 总结

### AI 助手角色

**自动化任务**：
- ✅ 检查 Skill 完整性
- ✅ 生成中英文文档
- ✅ 生成 LICENSE 和 CHANGELOG
- ✅ 敏感信息脱敏
- ✅ 生成 GitHub 仓库信息
- ✅ 推送代码到 GitHub
- ✅ 准备 Release 内容
- ✅ 准备 ClawHub 提交内容

**辅助任务**：
- ⚠️ 提供详细的操作步骤说明
- ⚠️ 提供坑点提醒和解决方案
- ⚠️ 提供检查清单
- ⚠️ 提供常见问题解答

### 用户作者角色

**必需操作**：
- ✅ 创建 GitHub 仓库
- ✅ 执行 Git 推送命令
- ✅ 创建 GitHub Release
- ✅ 上传截图
- ✅ 提交到 ClawHub
- ✅ 等待审核

**预计总时间**：
- 准备工作（AI 自动化）：5-10 分钟
- GitHub 仓库创建：3-5 分钟
- 推送代码（AI 自动化）：1-2 分钟
- 创建 Release：5-10 分钟
- 提交到 ClawHub：5-10 分钟
- **总计：约 30 分钟（用户操作时间）**

### 无痛体验关键

1. **AI 自动化 80% 的工作**
   - 文档生成
   - 代码推送
   - 内容准备

2. **用户只需做 20% 的关键操作**
   - 创建仓库
   - 创建 Release
   - 提交到 ClawHub

3. **详细的步骤指导**
   - 每一步都有详细说明
   - 坑点提前提醒
   - 解决方案清晰

4. **完整的检查清单**
   - 提交前检查
   - 过程中检查
   - 提交后检查

5. **常见问题解答**
   - 常用命令
   - 常见坑点
   - 解决方案

---

**祝你成功发布第一个 Skill！** 🎉

如有任何问题，随时咨询 AI 助手。

---
> Source: [0xf4vul/clawhub-skill-submission-guide](https://github.com/0xf4vul/clawhub-skill-submission-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
