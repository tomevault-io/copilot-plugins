## kml-commontools

> KML-CommonTools 是一个生物信息学工具集合，基于 Python 3.12+ 开发，使用 Poetry 管理依赖。项目提供多个命令行工具，用于处理测序数据、样本信息转换等任务。

# AGENTS.md - KML-CommonTools 项目指南

## 项目概述

KML-CommonTools 是一个生物信息学工具集合，基于 Python 3.12+ 开发，使用 Poetry 管理依赖。项目提供多个命令行工具，用于处理测序数据、样本信息转换等任务。

## 构建与运行

### 依赖管理

```bash
# 安装依赖
poetry install

# 添加新依赖
poetry add <package>
```

### 运行工具

```bash
# 运行已注册的命令行工具
poetry run <command-name>

# 示例
poetry run haobo-info-rec-to-samplesheet input.xlsx output.csv
poetry run prep-input-from-bcl2fastq --input-dir /path/to/fastq --output-file input.tsv

# 运行模块（未注册为命令的工具）
poetry run python -m src.kml_commontools.<module_name>

# 导入并使用 gene_id_lookup 函数（通过 conda develop 安装）
import gene_id_lookup
gene_mapping = gene_id_lookup.get_gene_ids(
    ["TP53", "BRCA1", "EGFR"],
    "/path/to/Homo_sapiens.gene_info"
)
```

## 代码风格指南

### 格式规范

- **缩进**: 4 个空格（非 Tab）
- **行尾**: LF（Unix 风格）
- **字符集**: UTF-8
- **行长度**: 无严格限制，但建议不超过 120 字符

### 导入规范

```python
# 标准库导入
import logging
import subprocess
from pathlib import Path
from typing import Union, Optional

# 第三方库导入
import click
import pandas as pd
from Bio import SeqIO

# 项目内部导入
from src.config.software import DWGSIM, SAMTOOLS
```

### 命名约定

- **函数/变量**: snake\_case（如 `simulate_fastq_by_vcf`）
- **类名**: PascalCase（如 `SimulateFastqByVcf`）
- **常量**: UPPER\_SNAKE\_CASE（如 `DWGSIM`, `BWA`）
- **模块文件**: snake\_case（如 `prep_input_from_bcl2fastq.py`）

### 类型注解

```python
# 使用 typing 模块的类型注解
def process_file(input_path: Union[str, Path], output_file: str) -> None:
    pass

# 使用 Optional 表示可选参数
def analyze(data: pd.DataFrame, threshold: Optional[float] = None) -> dict:
    pass
```

### CLI 工具规范

使用 `click` 库创建命令行接口：

```python
@click.command()
@click.option("--input-file", type=click.Path(exists=True), required=True, help="输入文件")
@click.option("--output-file", default="output.tsv", show_default=True, help="输出文件")
@click.help_option("--help", help="显示帮助信息并退出")
def my_tool(input_file: str, output_file: str):
    """工具描述（会显示在帮助信息中）"""
    pass
```

### 日志记录

```python
import logging

# 配置日志格式
logging.basicConfig(level=logging.INFO, format="%(asctime)s - %(levelname)s - %(message)s")

# 或使用模块级 logger
logger = logging.getLogger(__name__)
logger.info("处理完成")
```

### 错误处理

```python
# 使用 click.ClickException 抛出用户友好的错误
if not valid:
    raise click.ClickException(f"输入文件 {input_file} 格式不正确")

# subprocess 调用使用 check=True
subprocess.run(cmd, shell=True, check=True, capture_output=True, text=True)
```

### 数据类使用

```python
from dataclasses import dataclass
from typing import Union, Optional

@dataclass
class SimulateConfig:
    input_ref: Union[str, Path]
    output_prefix: Union[str, Path]
    variant_allele_freq: float
    input_vcf: Optional[Union[str, Path]] = None

    def __post_init__(self):
        # 初始化后处理
        self.tmpdir = Path(self.output_prefix).parent / ".tmp"
```

## 项目结构

```text
src/
├── assets/           # 静态资源文件（模板等）
├── config/           # 配置文件（软件路径等）
├── functions/        # 通用功能函数模块
├── kml_commontools/  # 主要工具模块
│   ├── __init__.py
│   ├── haobo_info_rec_to_samplesheet.py
│   ├── prep_input_from_bcl2fastq.py
│   ├── simu_fq_by_vcf.py
│   └── ...
└── util/             # 辅助脚本（Perl 等）
```

## 基因ID查询函数

`src/functions/gene_id_lookup.py` 提供了 `get_gene_ids` 函数，用于根据NCBI gene\_info文件查询基因名称对应的GeneID。

### 函数说明

```python
def get_gene_ids(
    gene_symbols: list[str],
    gene_info_file: Union[str, Path],
) -> dict[str, str]:
    """根据NCBI gene_info文件查询基因名称对应的GeneID。

    Args:
        gene_symbols: 基因名称列表，可以是Symbol或Synonyms。
        gene_info_file: NCBI gene_info文件路径。

    Returns:
        字典，键为输入的基因名称，值为对应的GeneID（字符串）。
        如果某个基因名称未找到，则不包含在字典中。
    """
```

### 使用示例

```python
import gene_id_lookup

# 查询基因ID
gene_symbols = ["TP53", "BRCA1", "EGFR", "P53"]
gene_info_file = "/data/mengxf/Database/NCBI/gene/Homo_sapiens.gene_info"

result = gene_id_lookup.get_gene_ids(gene_symbols, gene_info_file)
# 输出: {"TP53": "7157", "BRCA1": "672", "EGFR": "1956", "P53": "7157"}

for gene, gene_id in result.items():
    print(f"{gene}: {gene_id}")
```

### 安装方式

该模块已通过 `conda develop` 安装在 Python 3.12 环境下，可直接导入使用：

```bash
conda develop /data/mengxf/Agents/KML-CommonTools/src/functions
```

### 注意事项

- 支持基因Symbol和Synonyms查询
- 未找到的基因名称不会出现在结果字典中
- 需要有效的NCBI gene\_info文件（可从NCBI下载）

## 测试

当前项目测试目录为 `tests/`，暂无自动化测试配置。如需添加测试：

```bash
# 安装 pytest
poetry add --group dev pytest

# 运行测试
poetry run pytest

# 运行单个测试文件
poetry run pytest tests/test_<name>.py
```

## 注意事项

- 外部工具路径配置在 `src/config/software.py` 中
- 所有 CLI 工具使用中文帮助信息
- 处理 Excel 文件时注意忽略 openpyxl 的 UserWarning
- 大文件操作建议使用流式处理，避免内存溢出
- 基因ID查询函数位于 `src/functions/gene_id_lookup.py`，通过 `conda develop` 安装后可直接导入
- 使用 `get_gene_ids` 函数时需要确保gene\_info文件路径正确

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kr1stoff)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kr1stoff)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
