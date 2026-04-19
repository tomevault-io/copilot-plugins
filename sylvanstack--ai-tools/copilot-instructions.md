## ai-tools

> max_line_length = 120

# Cursor规则配置文件

# Python相关规则
[*.py]
indent_style = space
indent_size = 4
max_line_length = 120
trim_trailing_whitespace = true
insert_final_newline = true

# JavaScript/TypeScript规则
[*.{js,ts,jsx,tsx}]
indent_style = space
indent_size = 2
max_line_length = 100
trim_trailing_whitespace = true
insert_final_newline = true

# HTML/CSS规则
[*.{html,css}]
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
insert_final_newline = true

# JSON规则
[*.json]
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
insert_final_newline = true

# Markdown规则
[*.md]
trim_trailing_whitespace = false
max_line_length = off

# YAML规则
[*.{yml,yaml}]
indent_style = space
indent_size = 2
trim_trailing_whitespace = true
insert_final_newline = true

# 忽略一些特定的文件
[*.min.*]
indent_style = ignore
trim_trailing_whitespace = false
insert_final_newline = ignore

# 项目特定规则
[sca-api/**]
python_path = /Users/sylvan/code/ai-tools/sca-api/ 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SylvanStack) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
