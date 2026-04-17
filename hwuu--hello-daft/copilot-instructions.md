## hello-daft

> 1. 如果我要求先讨论方案时不要着急修改代码，直到方案确定才可以修改代码。

1. 如果我要求先讨论方案时不要着急修改代码，直到方案确定才可以修改代码。

2. 方案讨论需要在我们双方都没疑问的情况下才可以输出具体方案文档。

3. 方案评估请主动思考需求边界，合理质疑当下方案的完善性，方案需包含：重要逻辑的实现思路、需求按技术实现的依赖关系拆解并排序，便于后续渐进式开发、输出修改或新增文件的路径、输出测试要点利于需求完成后的自动化测试。

4. 方案讨论或代码编写时，如果遇到了争议或不确定性请主动告知我，请牢记让我决策而不是默认采用一种方案实现，重点强调。

5. 开发项目必须严格按步骤执行，每次只专注当前讨论的步骤，要求：不允许跨步骤实现功能或"顺便"完成其他步骤任务、实现前必须先确认技术方案和实现细节、每个步骤完成后必须明确汇报，等待 Review 确认后才能进入下一步。

6. 进行代码提交时，请先给我梳理提交的内容，等待 Review 确认后才能进行提交。

7. 与第五,六点类似，任何代码修改请始终遵守最小改动原则，除非我主动要求优化或者重构。

8. 代码实现请先思考哪些业务可以参考或复用，尽可能参考现有业务的实现风格，如果你不明确可让我为你提供，避免重复造轮子。

9. 不要在源码中插入mock的硬编码数据。

10. 同步更新相关文档。

11. 使用TDD开发模式开发。

12. 小步快跑，对一步都进行测试，并保证不影响现有用例。

13. 使用中文回答

14. 记得每次测试完后，清理下测试文件。

15. 在 bug 修复时如果超过 2 次修复失败，请主动添加关键日志后再进行尝试修复，在我反馈修复后主动清除之前的日志信息。

16. 项目中的重试过2次以上环境配置问题或其他重复犯错的问题，请在项目的CLAUDE.md中做记录。常用的命令，请记录在项目的CLAUDE.md中。

17. 及时在 tests/unit 中添加单元测试

18. 文档中如果要画流程框图，那么框图中的文字用英文，框线要对齐；其余内容保持中文

19. commit message 使用中文

20. 各 Demo 的 README.md 只做入口导航（简介、快速开始、notebook 列表、前置要求），所有教学内容（概念讲解、代码示例、练习题、FAQ、扩展阅读）放在 notebook 中。

21. Notebook 中所有 Python 函数（包括 Ray remote 函数、Actor 方法）必须添加类型注解。

22. Daft UDF 统一使用 `@daft.func` + 类型注解，禁止使用已废弃的 `@daft.udf(return_dtype=...)`。

23. Notebook 作为教学文档，每个代码 cell 前必须有 markdown cell 说明代码目的和关键概念。

---

## 常用命令

```bash
# 生成 demo1 测试数据（默认 10 万条，可用 --size 调整）
python demo1_daft/data/generate_data.py --size 1000 --output demo1_daft/data

# 清理缓存
rm -rf .pytest_cache **/__pycache__

# nbstripout（已安装，commit 时自动清除 notebook output）
nbstripout --status
```

## 环境备忘

- Python 环境：`hello_daft` conda 环境
- 初始环境只有 `daft 0.7.2` 和 `pyarrow`，需手动安装 `pandas`, `faker`, `pytest` 等依赖

## Daft 0.7.2 API 备忘

- 无 `rename()` 方法，用 `select(col("old").alias("new"))` 替代
- `@daft.udf` 已废弃，使用 `@daft.func`（逐元素）或 `@daft.cls`（批处理）
- `explain()` 返回 `None`（直接打印到 stdout），需用 `io.StringIO` 捕获
- `fill_null()` 直接在 Expression 上调用
- `with_columns({})` 支持字典形式批量添加/修改列
- 无 `is_not_null()` 方法，使用 `not_null()` 替代
- 无 `Expression.str` 命名空间，字符串方法直接在 Expression 上调用：`col("x").length()` 而非 `col("x").str.length()`
- 字符串拼接用 `+` 运算符：`col("a") + lit(" ") + col("b")`，不支持 `str.concat()`
- `distinct()` 参数用位置参数：`df.distinct("col")` 而非 `df.distinct(on="col")`
- 聚合表达式（如 `.sum()`）不能在 `select()` 中使用，必须用 `agg()`
- `is_null()` 返回 Bool 类型，不能直接 `.sum()`，需先 `.cast(daft.DataType.int64())` 再 `.sum()`
- `column_names` 是属性不是方法，不要加括号
- `over()` 需要传入 `Window` 对象，不能直接传 Expression，用法：`col("x").mean().over(Window().partition_by("y"))`
- Ray 后端切换用 `daft.set_runner_ray()`，不是 `daft.context.set_runner_ray()`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hwuu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
