## choon-stock-analysis-report

> AWS Lambda 기반의 **대형주 주식 기술적 분석 리포트 생성기**이다.

# CLAUDE.md

## 프로젝트 개요

AWS Lambda 기반의 **대형주 주식 기술적 분석 리포트 생성기**이다.
SQS 메시지를 트리거로 S3에 저장된 주식 기술적 분석 JSON 데이터를 읽어와, Amazon Bedrock Claude Sonnet 4.5 모델로 분석한 뒤 DOCX/PDF 형식의 투자 리포트를 생성하여 S3에 저장한다.

## 아키텍처

```
SQS → Lambda (이 함수) → S3 (JSON 읽기) → Bedrock Claude Sonnet 4.5 → DOCX 생성 → PDF 변환 → S3 (DOCX/PDF 저장)
```

- **런타임**: Python 3.14 (AWS Lambda, Docker 컨테이너)
- **베이스 이미지**: `public.ecr.aws/shelf/lambda-libreoffice-base:25.8-python3.14-x86_64`
- **리전**: ap-northeast-2 (Lambda), us-east-1 (Bedrock - Cross-region inference)
- **모델**: `us.anthropic.claude-sonnet-4-5-20250929-v1:0`

## 파일 구조

```
choon-stock-analysis-report/
├── lambda_function.py    # Lambda 핸들러 및 전체 로직
├── Dockerfile            # Lambda 컨테이너 이미지 빌드
├── CLAUDE.md             # AI 어시스턴트용 프로젝트 컨텍스트
├── README.md             # 프로젝트 설명
├── .gitignore
└── .dockerignore
```

## 핵심 함수

| 함수 | 설명 |
|---|---|
| `lambda_handler` | SQS 이벤트를 수신하여 전체 파이프라인 실행 |
| `parse_s3_uri` | `s3://bucket/key` 형식의 URI를 (bucket, key) 튜플로 파싱 |
| `read_json_from_s3` | S3에서 JSON 파일 읽기 |
| `invoke_bedrock_sonnet` | Bedrock Claude Sonnet 4.5에 기술적 분석 데이터를 전달하고 구조화된 JSON 결과 수신 |
| `set_cell_shading` | 테이블 셀 배경색 설정 |
| `add_heading` | 문서에 헤딩 추가 |
| `create_table_with_header` | 헤더가 있는 테이블 생성 |
| `add_table_row` | 테이블에 행 추가 |
| `create_docx_report` | 분석 결과 JSON을 DOCX 리포트로 변환 |
| `save_docx_to_s3` | 생성된 DOCX 파일을 S3에 업로드 |
| `convert_docx_to_pdf` | LibreOffice를 사용하여 DOCX를 PDF로 변환 |
| `save_pdf_to_s3` | 생성된 PDF 파일을 S3에 업로드 |

## 의존성

- `boto3` - AWS SDK (S3, Bedrock Runtime)
- `python-docx` - DOCX 문서 생성
- `LibreOffice 25.8` - PDF 변환 (베이스 이미지에 포함)
- `subprocess`, `os` (stdlib) - LibreOffice 프로세스 실행

## 입력 데이터 형식

SQS 메시지 body에 다음 필드가 포함된다:
- `execution_date` - 실행일자
- `s3_location` - 분석 데이터 JSON의 S3 URI (`s3://bucket/key`)
- `analyzed_tickers`, `total_tickers`, `condition_met_count` - 메타 정보

S3 JSON 데이터에는 다음 키가 포함된다:
- `condition_met_tickers` - SMA 골든크로스 종목 (15일선이 50일선 상향 돌파 + 120일선 상회)
- `dead_cross_tickers` - SMA 데드크로스 종목
- `macd_golden_cross_tickers` - MACD 골든크로스 종목
- `macd_dead_cross_tickers` - MACD 데드크로스 종목
- `all_results` - 전체 종목 상세 데이터

## 출력

입력 JSON과 동일한 S3 경로에 확장자만 변경하여 저장된다:
- `.docx` - Word 문서 리포트
- `.pdf` - PDF 리포트 (LibreOffice 변환, 실패 시 DOCX만 저장)

리포트 구성:
1. 분석 개요 (분석일자, 대상 종목 수, 분석 지표)
2. 시장 신호 요약 (골든/데드크로스 카운트, 종합 분석)
3. 매수 추천 종목
   - 3.1 최우선 매수 추천 (SMA + MACD 이중 골든크로스)
   - 3.2 MACD 모멘텀 매수 추천 (히스토그램 상위 10)
4. 매도/주의 종목
   - 4.1 SMA 데드크로스 (단기 추세 전환)
   - 4.2 MACD 매도 압력 강세 (히스토그램 하위 10)
5. 빅테크 종목 동향 (AAPL, MSFT, GOOGL, AMZN, META, NVDA, TSLA)
6. 투자 전략 제안 (단기/스윙/리스크 관리)
7. 면책 조항

## 코드 수정 시 유의사항

- `SYSTEM_PROMPT`에 Bedrock 모델의 출력 JSON 스키마가 정의되어 있다. 스키마를 변경하면 `create_docx_report` 함수도 함께 수정해야 한다.
- `create_docx_report`의 테이블 구조는 `SYSTEM_PROMPT`의 JSON 스키마에 1:1로 대응한다.
- 빅테크 종목 목록은 `SYSTEM_PROMPT` 안에 하드코딩되어 있다.
- DOCX 기본 폰트는 `Malgun Gothic` (맑은 고딕)이다.
- Bedrock 응답에서 마크다운 코드블록(` ```json ``` `)을 제거하는 파싱 로직이 `invoke_bedrock_sonnet`에 포함되어 있다.
- Lambda는 SQS 배치 처리를 지원하며, `event['Records']` 루프로 여러 메시지를 순차 처리한다.
- PDF 변환은 LibreOffice headless 모드로 수행되며, 실패해도 DOCX는 정상 저장된다.
- LibreOffice 실행 시 `/tmp`에 프로필 디렉토리와 잠금 파일이 생성되므로, 매 실행 전 기존 임시 파일을 정리한다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flame-choon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/flame-choon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
