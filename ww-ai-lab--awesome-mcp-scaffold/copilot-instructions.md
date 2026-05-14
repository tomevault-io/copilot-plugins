## mcp-outputschema-enforcement

> 1. **返回类型注解**：必须使用 Pydantic 模型作为返回类型

# 🎯 MCP outputSchema 强制开发规范

## 🚨 CRITICAL REQUIREMENTS

### ✅ 每个工具必须满足的条件
1. **返回类型注解**：必须使用 Pydantic 模型作为返回类型
2. **导入响应模型**：从 `server.models.responses` 导入标准模型
3. **结构化返回**：返回实例化的 Pydantic 模型对象
4. **错误处理**：使用 `ErrorResult` 处理异常情况

## 📋 标准开发模板

### 工具实现模板
```python
# ✅ 正确的工具实现
from server.models.responses import CalculationResult, ErrorResult
from typing import Union

@mcp.tool(title="工具标题", description="详细描述")
def tool_name(param1: type, param2: type) -> Union[CalculationResult, ErrorResult]:
    """
    工具文档字符串
    
    Args:
        param1: 参数1描述
        param2: 参数2描述
        
    Returns:
        CalculationResult: 成功时返回计算结果
        ErrorResult: 失败时返回错误信息
    """
    try:
        # 业务逻辑处理
        result = perform_calculation(param1, param2)
        
        return CalculationResult(
            success=True,
            result=result,
            operation="tool_operation",
            expression=f"{param1} op {param2}"
        )
    except Exception as e:
        return ErrorResult(
            success=False,
            error_type=type(e).__name__,
            error_message=str(e),
            error_code="TOOL_ERROR"
        )
```

### 测试模板
```python
# ✅ 正确的测试实现
def test_tool_name():
    """测试工具的 outputSchema 和功能"""
    # 测试成功情况
    result = tool_name("valid_param1", "valid_param2")
    
    # 验证返回类型
    assert isinstance(result, (CalculationResult, ErrorResult))
    assert hasattr(result, 'success')
    assert hasattr(result, 'timestamp')
    
    # 验证 Pydantic 模型验证
    validated = result.model_validate(result.model_dump())
    assert validated == result
    
    # 测试错误情况
    error_result = tool_name("invalid_param1", "invalid_param2")
    if isinstance(error_result, ErrorResult):
        assert error_result.success is False
        assert error_result.error_type is not None
```

## 🔧 可用的响应模型

### 基础模型
- `BaseToolResponse`: 所有响应的基类
- `SimpleResult`: 简单操作结果
- `ErrorResult`: 错误结果

### 计算类模型
- `CalculationResult`: 数学计算结果
- `BMIResult`: BMI 计算专用
- `StatisticsResult`: 统计计算结果

### 文本处理模型
- `TextAnalysisResult`: 文本分析结果
- `CaseConversionResult`: 大小写转换结果
- `ExtractionResult`: 文本提取结果

### 文件操作模型
- `FileOperationResult`: 文件操作结果
- `FileInfoResult`: 文件信息结果
- `DirectoryResult`: 目录操作结果

### 系统信息模型
- `SystemInfoResult`: 系统信息结果

## ❌ 禁止的实现方式

### 错误示例 1：返回字符串
```python
# ❌ 错误：不要返回字符串
@mcp.tool(title="错误工具")
def bad_tool() -> str:
    return "这样不对"
```

### 错误示例 2：返回字典
```python
# ❌ 错误：不要返回字典
@mcp.tool(title="错误工具")
def bad_tool() -> Dict[str, Any]:
    return {"result": "value"}
```

### 错误示例 3：无返回类型注解
```python
# ❌ 错误：必须有类型注解
@mcp.tool(title="错误工具")
def bad_tool():
    return some_value
```

### 错误示例 4：不使用标准模型
```python
# ❌ 错误：不要自定义简单返回类型
@mcp.tool(title="错误工具")
def bad_tool() -> tuple:
    return (True, "result")
```

## 🚀 快速检查清单

在提交代码前，确保你的工具满足以下条件：

- [ ] 导入了正确的响应模型
- [ ] 工具函数有明确的返回类型注解
- [ ] 返回类型是 Pydantic 模型
- [ ] 实现了错误处理并返回 ErrorResult
- [ ] 包含完整的工具文档字符串
- [ ] 编写了相应的测试用例
- [ ] 测试验证了 outputSchema 生成

## 🔍 常见问题解决

### 问题 1：如何选择合适的响应模型？
**解决方案**：根据工具的功能选择：
- 数学计算 → `CalculationResult`
- 文本处理 → `TextAnalysisResult` 或其他文本模型
- 文件操作 → `FileOperationResult` 或 `DirectoryResult`
- 简单操作 → `SimpleResult`

### 问题 2：需要自定义响应字段怎么办？
**解决方案**：在 `server/models/responses.py` 中创建新模型：
```python
class CustomResult(BaseToolResponse):
    """自定义结果模型"""
    custom_field: str = Field(description="自定义字段")
    special_data: Optional[Dict[str, Any]] = Field(None, description="特殊数据")
```

### 问题 3：如何处理复杂的错误情况？
**解决方案**：使用 Union 类型和详细的 ErrorResult：
```python
def complex_tool() -> Union[SuccessResult, ErrorResult]:
    try:
        # 复杂逻辑
        pass
    except SpecificError as e:
        return ErrorResult(
            success=False,
            error_type="SpecificError",
            error_message=str(e),
            error_code="SPECIFIC_001"
        )
    except Exception as e:
        return ErrorResult(
            success=False,
            error_type="UnknownError",
            error_message=str(e),
            error_code="UNKNOWN_001"
        )
```

## 📚 参考资源

- [响应模型库](mdc:server/models/responses.py)
- [计算器工具示例](mdc:server/tools/calculator.py)
- [文本处理工具示例](mdc:server/tools/text_processing.py)
- [测试示例](mdc:tests/test_output_schema.py)
- [使用指南](mdc:docs/OUTPUTSCHEMA_GUIDE.md)

---

**🎯 记住：每个工具都必须返回 Pydantic 模型，这是 MCP v1.11.0 的核心要求！**

---
> Source: [WW-AI-Lab/Awesome-MCP-Scaffold](https://github.com/WW-AI-Lab/Awesome-MCP-Scaffold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
