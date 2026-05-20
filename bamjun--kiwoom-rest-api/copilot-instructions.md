## api-development-patterns

> All API methods in the Kiwoom REST API follow this consistent pattern:

# API Development Patterns for Kiwoom REST API

## Method Implementation Pattern

### Standard Method Structure
All API methods in the Kiwoom REST API follow this consistent pattern:

```python
def method_name_request_api_id(
    self,
    required_param1: str,
    required_param2: str,
    optional_param1: str = "",
    optional_param2: str = "",
    cont_yn: str = "N",
    next_key: str = ""
) -> dict:
    """
    Korean method description (API ID)

    Args:
        required_param1 (str): Parameter description with possible values
        required_param2 (str): Parameter description with possible values
        optional_param1 (str, optional): Optional parameter description. Defaults to "".
        optional_param2 (str, optional): Optional parameter description. Defaults to "".
        cont_yn (str, optional): 연속조회여부. Defaults to "N".
        next_key (str, optional): 연속조회키. Defaults to "".

    Returns:
        dict: Response data structure with detailed field descriptions
            {
                "field1": str,  # Korean field description
                "field2": str,  # Korean field description
                "array_field": [  # Array field description
                    {
                        "sub_field1": str,  # Sub-field description
                        "sub_field2": str,  # Sub-field description
                    },
                    ...
                ],
                "return_code": int,  # 응답코드
                "return_msg": str,  # 응답메시지
            }

    Example:
        >>> from kiwoom_rest_api import KiwoomRestAPI
        >>> api = KiwoomRestAPI()
        >>> result = api.module.method_name_request_api_id(
        ...     required_param1="value1",
        ...     required_param2="value2"
        ... )
        >>> print(result)
    """
    headers = {
        "cont-yn": cont_yn,
        "next-key": next_key,
        "api-id": "api_id",
    }
    data = {
        "required_param1": required_param1,
        "required_param2": required_param2,
    }
    
    if optional_param1:
        data["optional_param1"] = optional_param1
    if optional_param2:
        data["optional_param2"] = optional_param2
        
    return self._execute_request(
        "POST",
        json=data,
        headers=headers,
    )
```

### Key Patterns to Follow

1. **Method Naming**: Use descriptive names ending with `_request_api_id`
2. **Parameter Types**: All parameters should be typed as `str`
3. **Optional Parameters**: Use empty string defaults for optional parameters
4. **Pagination Support**: Always include `cont_yn` and `next_key` parameters
5. **Comprehensive Documentation**: Include Korean descriptions for all fields
6. **Example Usage**: Provide clear usage examples in docstrings
7. **Conditional Data**: Only add optional fields to request data if they have values

### Common Parameter Patterns

#### Pagination Parameters
- `cont_yn` (str): 연속조회여부, defaults to "N"
- `next_key` (str): 연속조회키, defaults to ""

#### Date Parameters
- Use format "YYYYMMDD" for date parameters
- Example: `start_date="20241128"`

#### Code Parameters
- Stock codes: 6-12 digit strings
- Order numbers: 7-20 digit strings
- Market codes: specific string values (KRX, NXT, etc.)

### Response Structure
All API responses include:
- `return_code` (int): Response status code
- `return_msg` (str): Response message
- Data fields specific to each API

### Error Handling
- Use the base class `_execute_request` method for consistent error handling
- Return raw API responses to allow caller to handle errors appropriately

## Testing Pattern

### Test Method Structure
```python
print_result("api_id_result", module.method_name_request_api_id(
    required_param1="value1",
    required_param2="value2"
), print_result=False)
```

### Test File Organization
- Test files should be in `tests/koreanstock/` directory
- Use descriptive test names that match the API ID
- Include both success and error case testing
- Use `print_result=False` for clean test output

---
> Source: [bamjun/kiwoom-rest-api](https://github.com/bamjun/kiwoom-rest-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
