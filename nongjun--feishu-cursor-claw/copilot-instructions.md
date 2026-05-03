## python

> Python 安全，扩展通用规则

# Python 安全

> 本文件以 Python 特定内容扩展通用安全规则。

## 密钥管理

```python
import os
from dotenv import load_dotenv

load_dotenv()

api_key = os.environ["OPENAI_API_KEY"]  # 缺失时抛出 KeyError
```

## 安全扫描

- 使用 **bandit** 进行静态安全分析：
  ```bash
  bandit -r src/
  ```

## 参考

Django 项目可参考 Django 官方安全文档（如适用）。

---
> Source: [nongjun/feishu-cursor-claw](https://github.com/nongjun/feishu-cursor-claw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
