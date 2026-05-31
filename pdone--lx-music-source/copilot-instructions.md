## lx-music-source

> 本项目是洛雪音乐源仓库，包含多个音乐源的 JS 文件。

# AGENTS.md

本项目是洛雪音乐源仓库，包含多个音乐源的 JS 文件。

## 项目结构

- `sixyin/` - 六音源
- `huibq/` - Huibq 源
- `flower/` - 野花源
- `lx/` - 洛雪官方源
- `ikun/` - ikun 源
- `grass/` - Grass 源
- `juhe/` - 聚合 API 源
- `CreateLatest.py` - 更新 latest.js 的脚本

## 命令

### 更新 latest.js

```bash
python CreateLatest.py
```

将各源的版本文件复制为 latest.js。

### 添加新版本

1. 将新版本 JS 文件放入对应源目录（如 `sixyin/1.2.2.js`）
2. 编辑 `CreateLatest.py`，更新 `VERSIONS` 字典中的版本号
3. 运行 `python CreateLatest.py`

## 代码风格

### JS 文件规范

- 文件头部必须包含注释块，包含：
  - `@name` - 音源名称
  - `@description` - 描述和版本信息
  - `@version` - 版本号
  - `@author` - 作者
  - `@homepage` - 主页（可选）
  - `@repository` - 仓库地址（可选）
  - `@netease` / `@tencent` - 平台标识（可选）

示例：
```javascript
/*!
 * @name 六音音源
 * @description v1.2.1
 * @version v1.2.1
 * @author 六音
 * @homepage www.sixyin.com
 * @netease MUSIC_U=;
 * @tencent ts_last=y.qq.com/n/ryqq/album;
 * @preserve
 */
```

- `@preserve` 标记表示保留注释，防止被压缩工具移除

### Python 脚本规范

- 使用 Python 3
- 遵循 PEP 8 风格指南
- 使用类型提示（PEP 484）
- 注释使用中文

### 版本命名

- 使用语义化版本号（如 `1.2.0`, `4`）
- 版本号对应文件名（如 `1.2.0.js`）
- latest.js 始终指向当前最新版本

### Git 提交

- 使用中文提交信息
- 格式：`[源名] 更新到 v版本号`
- 示例：`[sixyin] 更新到 v1.2.1`

## 注意事项

- JS 文件通常是混淆或压缩后的代码，无需手动编辑
- 如需更新源，请从上游项目获取最新版本
- 保持各源目录结构一致

---
> Source: [pdone/lx-music-source](https://github.com/pdone/lx-music-source) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
