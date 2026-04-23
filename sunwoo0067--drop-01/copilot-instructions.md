## drop-01

> - **기본 언어**: 한국어 (Korean)

# Project Rules: Dropshipping Integration (Coupang & OwnerClan)

## 1. 🌐 언어 규칙 (Language Policy)
- **기본 언어**: 한국어 (Korean)
- **주석 (Comments)**: 모든 코드의 주석은 한국어로 작성합니다.
- **커밋 메시지 (Commits)**: 한국어로 작성하며, 명확한 동사로 끝맺습니다.
- **변수명**: 영어(CamelCase)를 사용하되, 비즈니스 로직 설명이 필요한 경우 주석으로 한국어 설명을 덧붙입니다.

---

## 2. 🌳 브랜치 전략 (Branching Strategy)
이 프로젝트는 **Gitflow-Lite** 전략을 따릅니다.

### 2.1 브랜치 구조
| 브랜치명 | 역할 | 설명 | 보호 규칙 |
|:---:|:---:|---|:---:|
| **main** | 배포 (Production) | 언제나 배포 가능한 안정적인 상태. `dev`에서만 병합 가능. | 🔒 직접 푸시 불가 |
| **dev** | 개발 (Development) | 다음 배포를 위한 통합 브랜치. 모든 기능 브랜치는 이곳으로 병합. | 🔒 강제 푸시 불가 |
| **feat/name** | 기능 (Feature) | 새로운 기능 개발. `dev`에서 분기하여 `dev`로 병합 (PR). | - |
| **fix/name** | 버그 수정 (Fix) | 버그 수정. `dev`에서 분기하여 `dev`로 병합. | - |
| **hotfix/name**| 긴급 수정 | `main`의 치명적 버그 수정. `main`에서 분기하여 `main` & `dev` 병합. | - |

### 2.2 작업 흐름 (Workflow)
1. `dev` 브랜치에서 최신 내용을 pull 받습니다.
2. 작업 목적에 맞는 브랜치를 생성합니다. (예: `feat/coupang-product-upload`)
3. 작업 완료 후 `dev` 브랜치로 Pull Request(PR)를 생성합니다.
4. 코드 리뷰 및 승인 후 `dev`로 Merge 됩니다.
5. 배포 주기에 맞춰 `dev` 내용을 `main`으로 Merge 합니다.

---

## 3. 📝 커밋 컨벤션 (Commit Convention)
Conventional Commits 형식을 따르며, 내용은 **한국어**로 작성합니다.

**형식:** `[태그]: [작업 내용 요약]`

| 태그 | 설명 | 예시 |
|:---:|---|---|
| `feat` | 새로운 기능 추가 | `feat: 쿠팡 상품 등록 API 연동` |
| `fix` | 버그 수정 | `fix: 오너클랜 주문 수집 오류 해결` |
| `docs` | 문서 수정 | `docs: README.md API 명세 업데이트` |
| `refactor` | 코드 리팩토링 | `refactor: 상품 데이터 매핑 로직 개선` |
| `style` | 코드 포맷팅/세미콜론 등 | `style: 코드 들여쓰기 수정` |
| `chore` | 빌드 업무, 패키지 매니저 등 | `chore: npm 패키지 버전 업데이트` |

---

## 4. 🛍️ 도메인 용어 및 API 규칙 (Domain Context)
제공된 API 문서를 기반으로 아래 용어 규칙을 준수합니다.

### 4.1 쿠팡 (Coupang)
- 판매자 ID: `vendorId` (예: A00012345)
- 등록상품 ID: `sellerProductId`
- 옵션 ID: `vendorItemId`
- 참조 문서: `product_api.md`, `logistics_api.md` 등

### 4.2 오너클랜 (OwnerClan)
- 상품 코드: `item_code` (주의: 쿠팡과 용어 구별 필수)
- 공급가: `supply_price`
- 참조 문서: `ownerclan_api_docs.md`

### 4.3 데이터 매핑 주의사항
- **상품명**: 쿠팡의 `sellerProductName`(등록상품명)과 오너클랜의 `item_name`을 매핑할 때 길이 제한(100자)을 확인하세요.
- **배송비**: 쿠팡의 `deliveryChargeType`과 오너클랜의 배송 정책을 비교하여 로직을 작성해야 합니다.

---

## 5. ⚠️ 코딩 가이드라인 (Coding Guidelines)
- **에러 처리**: API 호출 실패 시 반드시 예외 처리(Try-Catch)를 하고, 로그에는 `HTTP 상태 코드`와 `에러 메시지`를 한국어로 남깁니다.
- **환경 변수**: API Key(`Access Key`, `Secret Key`)는 절대 코드에 하드코딩하지 않고 `.env` 파일로 관리합니다.

- **깃허브 연동**: 이슈/PR/리뷰/알림/워크플로우 등 GitHub 작업은 **MCP를 우선**으로 사용합니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sunwoo0067) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
