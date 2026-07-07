## friendlinks

> ├── *.yml                    # 友链配置文件（每个 yml 对应一个站点）

# AGENTS - 友链管理指南

## 📁 目录结构

```
links/
├── *.yml                    # 友链配置文件（每个 yml 对应一个站点）
├── example.yml              # yml 格式示例（可参考）
└── README.md                # 本文档
```

**核心文件**: `links/*.yml` - 每个文件代表一个站点的友链配置

---

## 📝 YML 格式规范

### 完整示例

```yaml
site:
  name: 我的博客                      # 必填：站点名称
  description: 分享编程和技术相关文章  # 必填：站点描述
  url: https://example.com            # 必填：站点 URL（也是文件名）
  color: "#ff6600"                    # 可选：自定义节点颜色（6位16进制）
  links: /links                       # 可选：友链页面路由（默认 /links）
  friends:                            # 必填：友链数组
    - name: 编程小站                   # 友链名称
      url: https://codehub.example.com # 友链 URL
      description: 技术分享            # 可选：友链描述
      avatar: https://example.com/avatar.png # 可选：头像 URL
    - name: 技术前沿
      url: https://techfrontier.example.com
```

### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `site.name` | string | ✅ | 站点名称（显示在节点上） |
| `site.description` | string | ✅ | 站点描述（鼠标悬停显示） |
| `site.url` | string | ✅ | 站点 URL（也是文件名） |
| `site.color` | string | ❌ | 节点颜色，格式 `#RRGGBB` |
| `site.links` | string | ❌ | 友链页面路由，默认 `/links` |
| `site.friends[]` | array | ✅ | 友链数组 |
| `site.friends[].name` | string | ✅ | 友链名称 |
| `site.friends[].url` | string | ✅ | 友链 URL |
| `site.friends[].description` | string | ❌ | 友链描述 |
| `site.friends[].avatar` | string | ❌ | 友链头像 URL |

---

## 🚀 快速操作指南

### 添加新站点

**方法 1: 手动创建**
```bash
# 1. 创建 yml 文件（文件名 = URL 中的域名）
touch links/example.com.yml

# 2. 填写配置
vim links/example.com.yml
```

**方法 2: 使用脚本生成**
```bash
# 提取友链后自动创建
python3 scripts/extract-friends.py https://example.com/links
```

### 编辑现有站点

```bash
# 直接编辑 yml 文件
vim links/example.com.yml

# 格式化检查
bun run fmt links/example.com.yml

# 提交更改
git add links/example.com.yml
git commit -m "feat: 更新 example.com 友链"
```

### 删除站点

```bash
# 删除 yml 文件
rm links/example.com.yml

# 提交更改
git add links/
git commit -m "refactor: 移除 example.com 友链"
```

### 查看所有站点

```bash
# 列出所有 yml 文件
ls -1 links/*.yml | wc -l

# 查看特定站点
cat links/example.com.yml

# 搜索包含特定关键词的站点
grep -l "关键词" links/*.yml
```

---

## 📋 友链准入标准

### ✅ 允许收录

- **个人博客**：技术博客、生活博客、个人作品集
- **博客聚合站**：十年之约、萌国等独立博客社区

### ❌ 严格排除

- 博客框架/主题/工具（Hexo、Hugo、WordPress 等）
- 商业网站、产品页、公司官网
- CDN、图床、短链服务
- 托管平台、域名服务商
- 论坛、社区、社交媒体主页
- 纯导航站、聚合站（无原创内容）

---

## 🎨 自定义节点颜色

在 `site` 层级添加 `color` 字段：

```yaml
site:
  name: 我的博客
  description: 分享编程相关文章
  url: https://example.com
  color: "#ff6600"  # 完整 6 位 16 进制色
```

**不指定**：从默认 12 色调色板按域名哈希分配

**常用色值**：
- `#ff6600` - 橙色
- `#00ccff` - 蓝色
- `#33cc33` - 绿色
- `#ff3366` - 粉红
- `#cc33ff` - 紫色
- `#ffcc00` - 黄色

---

## 🔧 开发规范

### 包管理器

**本项目强制使用 Bun**

```bash
bun install          # 安装依赖
bun run <script>     # 运行脚本
bun run lint         # 代码检查
bun run fmt          # 代码格式化
```

**禁止**：npm、yarn、pnpm

### 代码风格

```bash
bun run lint         # oxlint 检查
bun run fmt          # oxfmt 格式化
bun run lint && bun run fmt  # 提交前检查
```

### 清理脚本

`scripts/prune-irrelevant.ts` - 剔除不符合 AGENTS.md 标准的友链

```bash
bun run prune
```

**过滤规则**（`scripts/filter/` 目录）：

| 文件 | 用途 |
|------|------|
| `names.ts` | 名称关键词过滤（如"博客"、"blog"等） |
| `urls.ts` | URL 模式匹配 |
| `domains.ts` | 非博客域名列表 |
| `sensitive.ts` | 敏感域名（SHA-256 哈希） |
| `subdomains.ts` | 服务子域名前缀（如 `cdn.`、`img.`） |
| `platforms.ts` | 托管平台列表（GitHub Pages、Vercel 等） |

空文件会自动删除。

---

## 📊 数据端点

| 端点 | 格式 | 用途 |
|------|------|------|
| `/graph.bin` | msgpack 二进制 | 3D 图数据（客户端加载） |
| `/all.json` | JSON | 完整站点数据（外部使用） |

---

## 🛠️ 实用工具

### 提取友链脚本

`scripts/extract-friends.ts` - 从网站提取友链

```bash
# 手动提取
bun run extract https://example.com/links

# 自动创建 yml 文件
bun run extract --create-yml https://example.com/links
```

### 清理脚本

```bash
bun run prune              # 剔除不符合标准的友链
bun run prune --dry-run    # 预览将要删除的内容
```

### 统计信息

```bash
# 统计站点数量
ls -1 links/*.yml | wc -l

# 统计友链总数
grep -h "^  - name:" links/*.yml | wc -l

# 统计每个站点的友链数
for f in links/*.yml; do
  echo "$(basename $f): $(grep -c "^  - name:" $f) 友链"
done
```

---

## 📖 常见问题

### Q: 文件名和 URL 不一致怎么办？

**A**: 确保文件名与 URL 中的域名一致。例如：
- URL: `https://example.com` → 文件名: `example.com.yml`
- URL: `https://blog.example.com` → 文件名: `blog.example.com.yml`

### Q: 如何批量更新友链？

**A**: 使用提取脚本：
```bash
# 提取并自动创建 yml
bun run extract --create-yml https://example.com/links

# 手动提取后编辑
bun run extract https://example.com/links
vim links/example.com.yml
```

### Q: 如何确认友链是否符合标准？

**A**: 检查以下特征：
1. ✅ 有原创内容（文章、博客）
2. ✅ 个人或小团队运营
3. ❌ 不是框架/主题/工具
4. ❌ 不是商业网站
5. ❌ 不是纯导航站

### Q: 友链被移除后如何恢复？

**A**:
```bash
# 检查 git 历史
git log --oneline --all -- links/example.com.yml

# 恢复特定版本
git checkout <commit-hash> -- links/example.com.yml
```

---

## 📝 提交规范

```bash
# 添加新友链
git add links/example.com.yml
git commit -m "feat: 添加 example.com 友链 (X个友链)"

# 更新现有友链
git add links/example.com.yml
git commit -m "fix: 更新 example.com 友链描述"

# 移除友链
git add links/
git commit -m "refactor: 移除 example.com 友链"

# 修改格式
git add links/
git commit -m "style: 格式化 example.com.yml"
```

**注意**：不要自动 push，需要手动推送：
```bash
git push origin main
```

---
> Source: [xingwangzhe/FriendLinks](https://github.com/xingwangzhe/FriendLinks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
