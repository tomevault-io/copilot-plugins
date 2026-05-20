## korean-stock-api-reference

> Handles account-related APIs including trading history, positions, and account status.

# Korean Stock API Reference Guide

## API Module Structure

### Account Module ([src/kiwoom_rest_api/koreanstock/account.py](mdc:src/kiwoom_rest_api/koreanstock/account.py))
Handles account-related APIs including trading history, positions, and account status.

#### Common API IDs
- `ka10072` - 일자별종목별실현손익요청 (Realized profit by date and stock)
- `ka10073` - 일자별종목별실현손익요청_기간 (Realized profit by period and stock)
- `ka10074` - 일자별실현손익요청 (Daily realized profit)
- `ka10075` - 미체결요청 (Unfilled orders)
- `ka10076` - 체결요청 (Filled orders)
- `ka10077` - 당일실현손익상세요청 (Today's realized profit detail)
- `ka10085` - 계좌수익률요청 (Account return rate)
- `ka10088` - 미체결 분할주문 상세 요청 (Unfilled split order detail)
- `ka10170` - 당일매매일지 요청 (Today's trading journal)
- `kt00001` - 예수금상세현황 요청 (Deposit detail status)
- `kt00002` - 일별추정예탁자산현황 요청 (Daily estimated deposit asset status)
- `kt00003` - 추정자산조회요청 (Estimated asset inquiry)
- `kt00004` - 계좌평가현황요청 (Account evaluation status)
- `kt00005` - 체결잔고요청 (Filled position)
- `kt00007` - 계좌별주문체결내역상세요청 (Account order execution detail)
- `kt00008` - 계좌별익일결제예정내역요청 (Next day settlement schedule)
- `kt00009` - 계좌별주문체결현황요청 (Account order execution status)
- `kt00010` - 주문인출가능금액요청 (Withdrawable order amount)
- `kt00011` - 증거금율별주문가능수량조회요청 (Orderable quantity by margin ratio)

### Stock Info Module ([src/kiwoom_rest_api/koreanstock/stockinfo.py](mdc:src/kiwoom_rest_api/koreanstock/stockinfo.py))
Provides basic stock information and market data.

### Chart Module ([src/kiwoom_rest_api/koreanstock/chart.py](mdc:src/kiwoom_rest_api/koreanstock/chart.py))
Handles chart data and technical analysis information.

### Market Condition Module ([src/kiwoom_rest_api/koreanstock/market_condition.py](mdc:src/kiwoom_rest_api/koreanstock/market_condition.py))
Provides market condition indicators and analysis.

## Common Parameter Patterns

### Stock Codes
- Format: 6-12 digit strings
- Examples: "005930" (삼성전자), "A005930" (with prefix)
- Always validate stock code format before API calls

### Date Formats
- Standard format: "YYYYMMDD"
- Examples: "20241128", "20240301"
- Use consistent date formatting across all APIs

### Market Codes
- `KRX`: 한국거래소 (Korea Exchange)
- `NXT`: 넥스트트레이드 (Next Trade)
- `%`: 전체 (All markets)
- `SOR`: 최선주문집행 (Best order execution)

### Trading Types
- `0`: 전체 (All)
- `1`: 매도 (Sell)
- `2`: 매수 (Buy)

### Margin Ratios
- `20%`, `30%`, `40%`, `50%`, `60%`, `100%`
- Used for margin trading calculations

### Order Status
- `0`: 전체 (All)
- `1`: 체결 (Filled)
- `3`: 미체결 (Unfilled)

## Response Field Patterns

### Common Fields
- `return_code` (int): API response status code
- `return_msg` (str): Response message
- `stk_cd` (str): Stock code
- `stk_nm` (str): Stock name
- `cur_prc` (str): Current price
- `cntr_qty` (str): Contract quantity
- `cntr_pric` (str): Contract price

### Account Fields
- `entr` (str): 예수금 (Deposit)
- `repl_amt` (str): 대용금 (Substitute money)
- `uncla` (str): 미수금 (Unpaid amount)
- `ord_alowa` (str): 주문가능현금 (Orderable cash)
- `wthd_alowa` (str): 인출가능금액 (Withdrawable amount)

### Trading Fields
- `ord_no` (str): 주문번호 (Order number)
- `ord_qty` (str): 주문수량 (Order quantity)
- `ord_pric` (str): 주문가격 (Order price)
- `cntr_qty` (str): 체결수량 (Contract quantity)
- `cntr_pric` (str): 체결가격 (Contract price)

## Error Handling Patterns

### Common Error Codes
- `0`: 성공 (Success)
- `-1`: 일반 오류 (General error)
- `-2`: 인증 오류 (Authentication error)
- `-3`: 파라미터 오류 (Parameter error)

### Error Response Structure
```python
{
    "return_code": -1,
    "return_msg": "오류 메시지"
}
```

## Testing Patterns

### Test File Structure
- Location: `tests/koreanstock/`
- Naming: `test_{module_name}.py`
- Import pattern: `from kiwoom_rest_api.koreanstock.{module} import {Class}`

### Test Method Pattern
```python
print_result("api_id_result", module.method_name_request_api_id(
    required_param="value"
), print_result=False)
```

### Environment Setup
- Use `.env` file for API credentials
- Load environment variables with `load_dotenv()`
- Initialize `TokenManager()` for authentication

---
> Source: [bamjun/kiwoom-rest-api](https://github.com/bamjun/kiwoom-rest-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
