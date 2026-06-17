## openclaw-flomo-skill

> 轻量 flomo 风格笔记。快速记录想法、搜索回顾。触发词：记一下、记笔记、搜笔记、回顾笔记、随机一条笔记。


# Memo Skill

本地轻量笔记，数据文件：`~/Documents/hub/areas/personal/memos.md`

## 命令路由

| 触发词 | 动作 |
|--------|------|
| 「记一下 xxx」「记笔记 xxx」 | 新增笔记 |
| 「搜笔记 xxx」 | 关键词搜索 |
| 「#标签 的笔记」 | 按标签筛选 |
| 「随机回顾」「随机一条笔记」 | 随机展示旧笔记 + AI 追问 |

---

## 新增笔记（Capture）

1. 提取用户输入的文本内容（去掉触发词前缀）
2. 识别文本中手动写的 `#标签`（允许多个）
3. AI 自动判断 1-3 个补充标签（归一化，用中文短词，不重复手动标签）
4. 获取当前时间（Asia/Shanghai）：`date '+%Y-%m-%d %H:%M'`
5. 追加到文件末尾：

```
## 2026-04-05 10:32
用户原文 #手动标签
AI标签：#自动标签1 #自动标签2

---
```

Bash 追加命令：
```bash
MEMOS_FILE="$HOME/Documents/hub/areas/personal/memos.md"
TIMESTAMP=$(date '+%Y-%m-%d %H:%M')
printf '\n## %s\n%s %s\nAI标签：%s\n\n---\n' \
  "$TIMESTAMP" "$CONTENT" "$MANUAL_TAGS" "$AI_TAGS" >> "$MEMOS_FILE"
```

6. 回复用户：「已记录 ✓」+ 显示刚写入的条目

---

## 搜索（Search）

**关键词搜索：**
```bash
grep -n "关键词" "$HOME/Documents/hub/areas/personal/memos.md"
```

提取完整条目（从上方最近的 `## ` 到下方最近的 `---`）。超过 5 条时先展示前 5 条并询问是否继续。

**按标签筛选：**
```bash
grep -n "#标签名" "$HOME/Documents/hub/areas/personal/memos.md"
```

---

## 随机回顾（Resurface）

1. 读取全部文件
2. 解析所有条目（`## ` 开头，`---` 结束）
3. 过滤最近 7 天，排除新笔记，优先选旧条目
4. 随机选 1 条
5. AI 生成 1 个开放性追问（引导思考，不问事实）
6. 输出：

```
📌 本周回顾

[笔记原文]

💬 深入一问：[AI 生成的追问]
```

---

## 注意事项

- 数据文件固定路径：`~/Documents/hub/areas/personal/memos.md`
- 只追加，不修改已有内容
- Resurface 无满足条件旧笔记时，从全部条目中随机选

---
> Source: [chester-zhou/openclaw-flomo-skill](https://github.com/chester-zhou/openclaw-flomo-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
