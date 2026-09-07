## bootcamp2026-drug

> 이 저장소에서 코드를 만들거나 고치기 전에 **반드시 아래를 지키세요.**

# bootcamp2026-drug — AI 어시스턴트 지침

이 저장소에서 코드를 만들거나 고치기 전에 **반드시 아래를 지키세요.**

## 0. 가장 먼저 할 일

**[docs/GLOSSARY.md](./docs/GLOSSARY.md) 를 읽으세요.** 이 저장소는 용어 사전을 단일 출처로 삼습니다.
변수·함수·타입·API 필드·DB 컬럼 이름은 **전부 그 문서에 있는 표기를 그대로** 씁니다.

사전에 없는 개념이 필요하면:
1. 코드를 쓰지 말고 먼저 `docs/GLOSSARY.md` 표에 용어를 추가하세요.
2. 사용자에게 "새 용어 N개를 사전에 추가했다"고 알리세요.

## 1. 이 프로젝트가 무엇인가

사용자가 먹는 **영양제(supplement)** 와 나이·성별·체중·섭취량을 입력하면
**영양소 조합 / 주의점 / 섭취 시기**를 돌려줍니다.

**복용 중인 약(medication)은 선택 입력**이며, 영양제와 약이 부딪히는지 확인하는 데만 씁니다.

> ⚠️ `supplement`(영양제)와 `medication`(약)은 **다른 개념입니다. 절대 섞지 마세요.**
> 이 둘을 혼동하면 상호작용 로직이 통째로 깨집니다.

## 2. 절대 쓰면 안 되는 이름

CI 가 자동으로 검사하며, 걸리면 **빌드가 실패합니다.**

| ❌ 금지 | ✅ 표준 |
|---|---|
| `gender` | `sex` |
| `weight` | `weight_kg` / `weightKg` |
| `dosage` | `dose_amount` / `doseAmount` |
| `warning` | `caution` / `riskLevel` |
| `drug`, `medicine`, `pill` | `medication` (약) 또는 `supplement` (영양제) |
| `combo`, `combination` | `nutrient_stack` / `nutrientStack` |

전체 목록은 [GLOSSARY.md 7절](./docs/GLOSSARY.md).

## 3. 표기 규칙

| 위치 | 표기 |
|---|---|
| 백엔드 Python, API JSON | `snake_case` (`dose_amount`) |
| 프론트엔드 TypeScript | `camelCase` (`doseAmount`) |
| 클래스 / 타입 / 컴포넌트 | `PascalCase` |

변환은 `frontend/src/api/case.ts` **한 곳에서만** 합니다.
컴포넌트 안에서 `snake_case` 가 보이면 잘못된 코드입니다.

## 4. 파일을 어디에 둘 것인가

```
backend/app/       Python. schemas / services / api routes
frontend/src/      TypeScript. types / api / components / pages
docs/              문서
```

- **저장소 루트에 소스 파일을 만들지 마세요.** CI 가 거부합니다.
- 프론트 타입은 이미 `frontend/src/types/analysis.ts` 에 있습니다.
  **새로 만들지 말고 그 파일을 고치세요.**
- import 는 `@/` 별칭을 씁니다: `import type { Supplement } from "@/types/analysis"`
  (`tsconfig.json` 의 `paths` 와 `vite.config.ts` 의 `resolve.alias` 가 짝입니다. 한쪽만 고치면 빌드가 깨집니다.)

## 5. LLM 호출 코드를 추가하지 마세요

LLM 은 **외부 AWS(다른 담당자)** 에 있습니다. 이 저장소에는 모델 호출 코드가 없습니다.
연결은 `backend/app/services/llm/` 의 provider 구조로만 합니다.
자세한 계약은 [docs/LLM_CONTRACT.md](./docs/LLM_CONTRACT.md).

## 6. 의료 도메인 안전 규칙

1. 응답의 `disclaimer` 를 제거하거나 숨기지 마세요.
2. 진단·처방 문구를 생성하지 마세요.
3. 건강 정보(나이·체중·영양제·약)를 저장하거나 로그에 남기지 마세요.

## 7. 작업을 끝내기 전에

```bash
python scripts/check_glossary.py   # 용어 검사
cd backend && pytest               # 백엔드 테스트
cd frontend && npm run build       # 타입 검사 + 빌드
```

세 개 다 통과해야 CI 가 녹색이 됩니다.

---
> Source: [IwanttobeaHokage/bootcamp2026-drug](https://github.com/IwanttobeaHokage/bootcamp2026-drug) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
