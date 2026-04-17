## seven-eleven

> > **[경고]** 본 프로젝트는 이기종 그래프 신경망(HIN) 기반의 신제품 예측 프레임워크입니다. AI 에이전트는 아래의 시스템적 경계와 규칙을 반드시 준수해야 합니다.

# 7-Eleven NPD AI Framework - Global Harness Constraints

> **[경고]** 본 프로젝트는 이기종 그래프 신경망(HIN) 기반의 신제품 예측 프레임워크입니다. AI 에이전트는 아래의 시스템적 경계와 규칙을 반드시 준수해야 합니다.

## 1. 컨텍스트 라우팅 (Context Routing)
AI 에이전트는 작업 중인 디렉토리에 따라 다음 국지적 규칙을 최우선으로 적용해야 합니다.
* 탐색적 데이터 분석(EDA) 및 시각화 작업: `eda/.gemini-rules.md`
* 데이터 전처리 및 그래프 생성 작업: `src/data_builder/.gemini-rules.md`
* GNN 모델 아키텍처 설계 및 수정: `src/models/.gemini-rules.md`
* 학습 루프 및 최적화 작업: `src/train/.gemini-rules.md`

## 2. 물리적 경계 및 데이터 불변성 (Physical Boundaries)
* `data/raw/` 및 `data/processed/` 디렉토리 내의 원천 데이터(CSV, JSON, Pickle 등)는 절대 스크립트로 직접 수정하거나 덮어쓰지 마십시오. (Read-Only)
* 모든 데이터 변환 결과는 명시적인 함수의 반환값이나 새로운 파일로만 저장되어야 합니다.

## 3. 도구 및 라이브러리 제약
* 프레임워크는 `PyTorch` 및 협의된 GNN 라이브러리를 메인으로 사용합니다. 사전 승인되지 않은 외부 패키지(`pip install`) 도입을 금지합니다.
* 작성된 코드는 `flake8`, `black` 등의 린터 검증을 통과해야 합니다.

## 4. 자가 수정 루프 (Self-Correction Loop)
* 에러(예: RuntimeError: size mismatch) 발생 시 인간에게 즉시 묻지 말고, 텐서 차원 및 로직을 스스로 점검하고 수정하여 유닛 테스트를 재실행하십시오.

## 5. 하드웨어 가속 및 효율성 (Hardware Acceleration & Efficiency)
* 대규모 데이터 연산, 그래프 임베딩 생성 및 모델 학습 시 GPU(CUDA)를 사용하는 것이 더 효율적인 경우, 반드시 GPU 가속을 활용하도록 코드를 작성하십시오. (예: `device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')`)
* CPU와 GPU 간의 불필요한 데이터 전송(Data Transfer)을 최소화하여 오버헤드를 방지하십시오.

## 6. 전처리 데이터 명세 (Data Specification)
모든 원천 데이터는 전처리되어 `data/processed/*.parquet`에 저장되어 있으며, 다음 컬럼명을 사용합니다.

* **POS 판매 데이터 (`B2_POS_SALE.parquet`)**
  * 주요 컬럼: `판매일자`, `판매시간`, `점포코드`, `POS번호`, `거래번호`, `상품코드`, `판매수량`, `판매금액`
  * 용도: [영수증]-[상품] 구매 엣지 및 타겟 레이블링(KPI)
* **온라인 주문 데이터 (`B3_OLN_ODR_DLVR.parquet`)**
  * 주요 컬럼: `영업일자`, `주문일자`, `주문시간`, `점포코드`, `거래번호`, `판매금액`
  * 용도: 온라인 채널별 상품 구매 패턴 분석
* **상품 마스터 정보 (`B4_ITEM_DV_INFO.parquet`)**
  * 주요 컬럼: `상품코드`, `상품명`, `대분류명`, `중분류명`, `소분류명`
  * 용도: [상품]-[속성] 관계 형성 및 노드 초기 피처
* **행사 데이터 (`B5_MNM_DATA.parquet`)**
  * 용도: 프로모션 기간별 [상품]-[이벤트] 관계 및 외부 요인 분석

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Song-exp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
