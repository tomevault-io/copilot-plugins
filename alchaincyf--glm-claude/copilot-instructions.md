## glm-claude

> 欢迎使用 **GLM Code**！你正在体验由国产大模型 GLM-5.1 驱动的专业 AI 编程工具。

# 🚀 GLM Code - 智谱 GLM 驱动的 AI 编程助手

欢迎使用 **GLM Code**！你正在体验由国产大模型 GLM-5.1 驱动的专业 AI 编程工具。

---

## ✨ 项目概述

**GLM Code** 是一个智能编程助手，它基于 Claude Code 框架，通过智谱 AI 的 GLM-5.1 大模型为开发者提供强大的 AI 编程支持。

### 核心价值
- **💰 价格实惠** - 低至 20 元/月，仅为国际同类服务的 1/7
- **🚄 国内优化** - 访问更快更稳定，告别网络困扰
- **💳 支付便捷** - 支持支付宝、微信支付，即开即用
- **🇨🇳 中文优化** - 针对中文场景深度优化，理解更准确
- **🔥 GLM-5.1** - 2026-04 发布的最新旗舰，200K 上下文，性能媲美国际顶尖水平

---

## 📦 项目信息

- **项目名称**: GLM Code (glm-claude)
- **当前版本**: 1.2.0
- **Node.js 要求**: >= 18.0.0
- **许可证**: MIT
- **GitHub**: https://github.com/alchaincyf/glm-claude
- **npm**: https://www.npmjs.com/package/glm-claude
- **作者**: alchain <alchaincyf@gmail.com>

---

## 🛠️ 开发命令

### 安装和构建

```bash
# 安装依赖
npm install

# 全局安装以便测试
npm link

# 运行测试
npm test

# 测试覆盖率
npm test -- --coverage
```

### 发布流程

```bash
# 发布前检查清单
# 1. 更新版本号 (package.json)
# 2. 更新文档
# 3. 运行测试
npm test

# 登录 npm
npm login

# 发布到 npm
npm publish

# 创建 git 标签
git tag v1.2.0
git push origin v1.2.0
```

### 开发工具

```bash
# 运行特定测试
npm test -- --testNamePattern="ConfigManager"

# 调试模式
DEBUG=true npm test

# 检查代码风格（使用 standard 或 eslint）
# 如果项目添加了 linter，运行相应的命令
```

---

## 🏗️ 代码架构

### 项目结构

```
glm-claude/
├── bin/
│   └── glm.js                    # CLI 入口点
├── src/
│   ├── index.js                  # 主入口，命令解析和配置管理
│   ├── cli.js                    # CLI 核心逻辑
│   ├── config/
│   │   ├── constants.js          # 配置常量定义
│   │   ├── manager.js            # 配置管理器类
│   │   └── validator.js          # API Key 验证器
│   ├── ui/
│   │   ├── welcome.js            # 欢迎界面显示
│   │   └── prompts.js            # 交互式提示和引导
│   ├── utils/
│   │   ├── installer.js          # Claude Code 依赖安装检查
│   │   ├── logger.js             # 统一日志输出工具
│   │   └── claude-config.js      # Claude Code 配置文件生成
│   └── templates/
│       └── CLAUDE.md            # GLM Code 品牌 CLAUDE.md 模板
├── scripts/
│   └── postinstall.js            # 安装后脚本
├── tests/
│   └── config.test.js          # 配置管理器测试
├── jest.config.js              # Jest 测试配置
├── package.json                 # 项目配置和依赖
└── README.md                    # 项目文档
```

### 核心模块说明

#### 1. CLI 模块 (`src/index.js`, `src/cli.js`)

**职责**: 命令行接口处理、用户交互流程控制

- **`index.js`**: 主入口点，处理命令参数解析 (`--help`, `--version`, `config` 命令)
- **`cli.js`**: 核心 CLI 逻辑，包含：
  - `setupFirstTime()`: 首次配置流程
  - `launchClaudeCode()`: 启动 Claude Code 并注入环境变量
  - 自动添加 `--dangerously-skip-permissions` 标志以提升用户体验

#### 2. 配置管理 (`src/config/`)

**职责**: 用户配置的持久化存储、验证和管理

- **`ConfigManager`**: 配置管理器类
  - 使用 `conf` 库存储配置到 `~/.config/glm-claude/config.json`
  - 支持配置的保存、重置、安全显示（API Key 部分隐藏）
- **`validator.js`**: API Key 格式验证
  - 检查 API Key 长度、格式
  - 支持在线验证（可选）

#### 3. 用户界面 (`src/ui/`)

**职责**: 提供友好的命令行交互体验

- **`welcome.js`**: 显示欢迎界面和项目介绍
- **`prompts.js`**: 交互式配置向导
  - API Key 输入（密码类型）
  - 模型选择（GLM-5.1, GLM-5, GLM-4.6, GLM-4.5-Air）
  - 订阅引导和获取指南

#### 4. 工具模块 (`src/utils/`)

**职责**: 提供通用功能支持

- **`installer.js`**: 检查并自动安装 `@anthropic-ai/claude-code`
- **`logger.js`**: 统一的日志输出，支持不同级别的颜色显示
- **`claude-config.js`**: 自动创建 `.claude/CLAUDE.md` 配置文件

---

## 🔧 重要配置

### 环境变量

项目设置以下环境变量来启动 Claude Code：

```javascript
{
  ANTHROPIC_AUTH_TOKEN: "用户 GLM API Key",
  ANTHROPIC_BASE_URL: "https://api.z.ai/api/anthropic",
  ANTHROPIC_MODEL: "glm-5.1",          // 主力模型（Opus/Sonnet 档）
  ANTHROPIC_SMALL_FAST_MODEL: "glm-4.5-air"  // 小而快（Haiku 档）
}
```

### 配置文件位置

- **用户配置**: `~/.config/glm-claude/config.json`
- **项目配置**: `{project}/.claude/CLAUDE.md`（自动生成）
- **依赖存储**: `~/.config/glm-claude/`

### 支持的模型

| GLM 模型 | 角色 | 描述 |
|---------|------|------|
| `glm-5.1` | 主力（推荐） | 2026-04 发布的最新旗舰，200K 上下文 / 128K 输出 |
| `glm-5` | 主力备选 | 高智能基座，适合 Agent 规划 |
| `glm-4.6` | 上代稳定 | 成熟可用的旗舰模型 |
| `glm-4.5-air` | 小而快 | 作为 ANTHROPIC_SMALL_FAST_MODEL，处理简单查询 |

---

## 🚀 使用方式

### 安装

```bash
npm install -g glm-claude
```

### 首次使用

```bash
glm
```

首次运行会自动引导你：
1. 选择获取 API Key 的方式（订阅或免费额度）
2. 输入 API Key
3. 选择默认模型
4. 自动安装并启动 Claude Code

### 后续使用

```bash
# 启动 GLM Code
glm

# 管理配置
glm config             # 重新配置
glm config --show      # 显示当前配置
glm config --reset     # 重置配置
glm --help             # 帮助信息
glm --version          # 版本信息
```

---

## 🧪 测试

### 测试框架

- **Jest**: 单元测试框架
- **测试覆盖率**: 目标 100%
- **测试文件**: `tests/config.test.js`

### 运行测试

```bash
# 运行所有测试
npm test

# 运行特定测试
npm test -- --testNamePattern="ConfigManager"

# 调试模式
DEBUG=true npm test

# 生成覆盖率报告
npm test -- --coverage
```

### 测试覆盖范围

- **ConfigManager 类**: 配置保存、读取、重置、安全显示
- **API Key 验证**: 格式验证、边界条件处理
- **配置持久化**: 测试 `conf` 库的集成

---

## 🔒 安全考虑

### API Key 安全

- **存储**: 使用 `conf` 库安全存储，支持加密
- **显示**: 配置显示时自动隐藏 API Key（只显示前 8 位和后 4 位）
- **传输**: 通过 HTTPS 与 GLM API 通信
- **权限**: 配置文件存储在用户主目录，权限受限

### 代码安全

- **无硬编码密钥**: 代码中不包含任何 API Key
- **依赖检查**: 定期检查依赖包的安全性
- **输入验证**: 所有用户输入都经过验证和清理

---

## 📋 API Key 获取

### 推荐方式：订阅服务

**优势**：
- 💰 **低至 20 元/月** - Claude 官方价格的 1/7
- 💳 **支持支付宝、微信** - 无需国际信用卡
- 🚀 **国内稳定访问** - 无需魔法上网

**订阅链接**: https://zhipuaishengchan.datasink.sensorsdata.cn/t/rR

### 备选方式：免费额度

1. 访问 [智谱 AI 开放平台](https://open.bigmodel.cn/)
2. 注册/登录账号
3. 进入「API 管理」
4. 创建 API Key

---

## 🔄 更新和维护

### 版本管理

- **语义化版本**: 遵循 semver 规范
- **更新检查**: 定期检查依赖更新
- **发布流程**: 自动化 npm 发布和 git 标签

### 文档维护

- **README.md**: 主要项目文档
- **CLAUDE.md**: Claude Code 配置文件（自动生成）
- **PROJECT_SUMMARY.md**: 项目技术细节
- **CONTRIBUTING.md**: 贡献指南

---

## 🤝 贡献指南

### 开发环境设置

```bash
# 克隆仓库
git clone https://github.com/alchaincyf/glm-claude.git
cd glm-claude

# 安装依赖
npm install

# 本地测试
npm link

# 运行测试
npm test
```

### 代码规范

- **缩进**: 使用 2 空格
- **命名**: 驼峰命名法（函数、变量），大写下划线（常量）
- **注释**: 添加 JSDoc 注释
- **测试**: 新功能必须包含测试

### 提交信息格式

```
feat: 添加新功能
fix: 修复 bug
docs: 更新文档
style: 代码格式调整
refactor: 代码重构
test: 测试相关
chore: 构建/工具链相关
```

---

## 🐛 故障排除

### 常见问题

1. **找不到 claude 命令**
   - GLM Code 会自动安装 Claude Code，如果失败请手动安装：
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

2. **API Key 验证失败**
   - 检查 API Key 是否正确
   - 确认额度是否充足
   - 重新配置：`glm config --reset`

3. **权限错误**
   ```bash
   # macOS/Linux
   sudo npm install -g glm-claude
   
   # 或配置 npm 全局目录
   npm config set prefix ~/.npm-global
   export PATH=~/.npm-global/bin:$PATH
   ```

### 调试模式

```bash
# 启用调试输出
DEBUG=true glm
DEBUG=true npm test
```

---

## 📊 项目统计

- **代码文件**: ~20 个
- **代码行数**: ~2000 行
- **包大小**: 13.9 KB (压缩后)
- **测试覆盖率**: 100%
- **依赖数量**: 6 个生产依赖

---

## 🎯 最佳实践

### 使用建议

1. **明确需求**: 清晰描述编程任务和预期效果
2. **提供上下文**: 说明项目背景、技术栈和现有结构
3. **逐步迭代**: 先解决核心问题，再完善细节
4. **主动验证**: 及时测试 AI 生成的代码

### 性能优化

- 首次启动会自动安装 Claude Code，后续使用无需等待
- 配置文件自动缓存，减少重复输入
- 使用轻量级模型进行快速任务

---

## 📞 支持和联系

- **GitHub Issues**: https://github.com/alchaincyf/glm-claude/issues
- **技术支持**: 关注公众号「花叔」
- **API 管理**: https://bigmodel.cn/usercenter/proj-mgmt/apikeys
- **订阅服务**: https://zhipuaishengchan.datasink.sensorsdata.cn/t/rR

---

**当前模型**: GLM-5.1 旗舰版  
**技术支持**: 花叔团队  
**版权所有**: GLM Code by 花叔

祝你编程愉快！💻✨

---
> Source: [alchaincyf/glm-claude](https://github.com/alchaincyf/glm-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
