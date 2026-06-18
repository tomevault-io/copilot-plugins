## openclaw-paper-search

> |


# paper-search - 学术论文搜索技能

一个功能完整的学术论文搜索和文献管理工具，专为科研工作者、学生和开发者设计。

## 🚀 快速开始

### 安装
```bash
# 1. 复制技能目录到OpenClaw的skills目录
cp -r paper-search /path/to/openclaw/skills/

# 2. 安装Python依赖
pip install -r requirements.txt
```

### 基本使用
```bash
# 搜索arXiv论文
python3 scripts/search_arxiv.py --query "machine learning" --max-results 10

# 获取BibTeX引用
python3 scripts/get_bibtex.py --arxiv "2402.04557v1"

# 整理搜索结果
python3 scripts/paper_organizer.py --input results.json --output organized.md
```

## 📖 详细功能

### 1. arXiv论文搜索
支持高级搜索语法和多种筛选选项：

```bash
# 基本搜索
python3 scripts/search_arxiv.py --query "deep learning"

# 按分类搜索
python3 scripts/search_arxiv.py --query "cat:cs.LG AND transformer"

# 按日期排序
python3 scripts/search_arxiv.py --query "AI" --sort-by submittedDate --sort-order descending

# 多种输出格式
python3 scripts/search_arxiv.py --query "quantum" --format json > results.json
python3 scripts/search_arxiv.py --query "neural networks" --format markdown > papers.md
```

### 2. 引用管理
自动生成标准BibTeX引用格式：

```bash
# 通过arXiv ID获取
python3 scripts/get_bibtex.py --arxiv "2402.04557v1"

# 通过DOI获取
python3 scripts/get_bibtex.py --doi "10.1038/s41586-021-03819-2"

# 批量处理
python3 scripts/get_bibtex.py --input ids.txt --output references.bib
```

### 3. 论文整理
智能整理和分类搜索结果：

```bash
# 整理为Markdown报告
python3 scripts/paper_organizer.py --input search_results.json --output report.md

# 生成阅读清单
python3 scripts/paper_organizer.py --input papers.json --format reading-list

# 按主题分类
python3 scripts/paper_organizer.py --input results.json --group-by category
```

## 🎯 适用场景

### 科研工作者
- **文献调研**：快速查找相关研究
- **参考文献管理**：自动生成引用格式
- **研究跟踪**：定期搜索最新论文
- **合作发现**：查找相关作者和研究机构

### 学生
- **课程论文**：收集参考文献
- **毕业论文**：系统文献综述
- **学术入门**：了解领域基础知识
- **项目研究**：查找技术方案和算法

### 开发者
- **技术调研**：查找算法实现和优化
- **开源项目**：参考相关研究和实现
- **技术选型**：比较不同技术方案
- **趋势分析**：了解技术发展方向

## 🔧 脚本说明

### `scripts/search_arxiv.py`
**功能**：arXiv论文搜索  
**参数**：
- `--query`：搜索查询（支持arXiv高级语法）
- `--max-results`：最大结果数（1-1000）
- `--start`：起始位置（用于分页）
- `--sort-by`：排序方式（relevance/lastUpdatedDate/submittedDate）
- `--sort-order`：排序顺序（ascending/descending）
- `--format`：输出格式（text/json/markdown）

### `scripts/get_bibtex.py`
**功能**：BibTeX引用生成  
**参数**：
- `--arxiv`：arXiv ID（如2402.04557v1）
- `--doi`：DOI（如10.1038/s41586-021-03819-2）
- `--identifier`：自动检测标识符类型
- `--input`：输入文件（包含多个ID）
- `--output`：输出文件路径

### `scripts/paper_organizer.py`
**功能**：论文整理和分类  
**参数**：
- `--input`：输入文件（JSON格式）
- `--output`：输出文件路径
- `--format`：输出格式（markdown/reading-list/bibtex）
- `--group-by`：分组方式（category/author/year）

## 📁 项目结构

```
paper-search/
├── SKILL.md              # OpenClaw技能描述（本文件）
├── README.md             # 详细项目文档
├── LICENSE               # MIT许可证
├── requirements.txt      # Python依赖
├── .gitignore           # Git忽略文件
├── scripts/              # 主要脚本
│   ├── search_arxiv.py   # arXiv搜索
│   ├── get_bibtex.py     # BibTeX生成
│   └── paper_organizer.py # 论文整理
├── examples/             # 使用示例
│   ├── basic_search.py   # 基础搜索示例
│   └── batch_process.py  # 批量处理示例
├── tests/                # 单元测试
│   └── test_search.py    # 搜索功能测试
└── references/           # 参考文档
    └── api_reference.md  # API参考
```

## 💡 实用技巧

### 搜索技巧
```bash
# 1. 使用分类代码限制领域
python3 scripts/search_arxiv.py --query "cat:physics.chem-ph AND catalyst"

# 2. 组合搜索条件
python3 scripts/search_arxiv.py --query "machine learning AND (neural OR deep)"

# 3. 排除特定词汇
python3 scripts/search_arxiv.py --query "quantum NOT computing"

# 4. 短语搜索
python3 scripts/search_arxiv.py --query '"graph neural network"'
```

### 批量处理
```bash
# 1. 批量搜索多个主题
topics=("AI" "ML" "DL")
for topic in "${topics[@]}"; do
  python3 scripts/search_arxiv.py --query "$topic" --max-results 5 --format json > "${topic}.json"
done

# 2. 批量生成BibTeX
cat paper_ids.txt | while read id; do
  python3 scripts/get_bibtex.py --arxiv "$id"
done > references.bib
```

### 自动化工作流
```bash
#!/bin/bash
# 自动化文献调研脚本

# 1. 搜索最新论文
python3 scripts/search_arxiv.py \
  --query "cat:cs.LG" \
  --max-results 20 \
  --sort-by submittedDate \
  --sort-order descending \
  --format json > latest_papers.json

# 2. 提取arXiv ID
jq -r '.[].id' latest_papers.json | sed 's|.*/||' > arxiv_ids.txt

# 3. 生成BibTeX
python3 scripts/get_bibtex.py --input arxiv_ids.txt --output references.bib

# 4. 生成报告
python3 scripts/paper_organizer.py --input latest_papers.json --output report.md

echo "文献调研完成！"
```

## ⚠️ 注意事项

### API使用限制
- arXiv API：每小时最多请求120次
- Crossref API：免费用户有限制
- 遵守各数据库的使用条款

### 学术诚信
- 仅用于文献调研和学习
- 尊重知识产权和版权
- 正确引用参考文献

### 技术限制
- 需要Python 3.7+环境
- 需要网络连接访问API
- 部分数据库需要API密钥

## 🔄 更新和维护

### 版本历史
- **v1.0.0** (2026-03-16): 初始版本发布
  - arXiv搜索功能
  - BibTeX引用生成
  - 论文整理工具

### 问题反馈
- GitHub Issues: https://github.com/yourusername/paper-search/issues
- OpenClaw社区: https://discord.gg/clawd

### 贡献指南
欢迎提交Pull Request或报告问题！

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

---

**开始你的学术探索之旅吧！** 🎓🔍

---
> Source: [Lloyd-Jahn/openclaw-paper-search](https://github.com/Lloyd-Jahn/openclaw-paper-search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
