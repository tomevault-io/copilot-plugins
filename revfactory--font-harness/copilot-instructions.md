## font-harness

> **목표:** 세상에 없는 독창적인 한글 폰트를 샘플 → 피드백 → 개선 루프로 제작한다.

# 한글 폰트 스튜디오

## 하네스: 한글 폰트 제작

**목표:** 세상에 없는 독창적인 한글 폰트를 샘플 → 피드백 → 개선 루프로 제작한다.

**트리거:** 폰트·글꼴·글리프·시안·자모·타이포그래피 관련 작업(제작·수정·피드백 반영·재실행·빌드) 요청 시 `hangul-font-studio` 스킬을 사용하라. 단순 질문은 직접 응답 가능.

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-07-07 | 초기 구성 (에이전트 12개, 스킬 6개, 빌드 스크립트) | 전체 | - |
| 2026-07-07 | 라운드 1 시안 제작 (콘셉트 3종 × 6음절) | _workspace/ | 샘플 우선 피드백 루프 시작 |
| 2026-07-07 | QA: 이음 '리' 오독(러) 수정, R7 규칙 일반화 | concepts/이음 | 시각 검증에서 브리지 오독 발견 |
| 2026-07-07 | 시안 조립기·QA 렌더러 번들링 | skills/font-specimen, skills/glyph-design | 라운드 1 반복 패턴 스크립트화 |
| 2026-07-07 | 라운드 2: 천지인 채택, 신규 14자(총 20자), 유형3·6 규칙 확정 | _workspace/, concepts/천지인 | 사용자 피드백 "천지인으로 한글 폰트 제작" |
| 2026-07-07 | 빌드 스크립트 cu2qu 변환 추가 (3차→2차 베지어) | skills/font-build/scripts | TTF glyf가 2차만 허용 — 첫 실빌드에서 발견 |
| 2026-07-07 | TTF v0.2 빌드 (Cheonjiin-v0.2.ttf, 20자) + 실조판 검증 | _workspace/build/ | 설치 가능한 폰트 파일 요구 |
| 2026-07-07 | 라운드 3: 화·관 받침 조정 (유형3 보 바닥 정렬, 유형6 받침 간격 120), TTF v0.3 | concepts/천지인, _workspace/build/ | 사용자 피드백 "화·관 받침 조정 필요" |
| 2026-07-07 | 라운드 4: 자모 조합 시스템 구축 (정의 67개 + compose.py 엔진), 11,172자 전체 생성, TTF v0.4 | _workspace/glyphs/천지인/system/, build/ | "다음 라운드 진행" — 조합 시스템 우선 권고안 채택 |
| 2026-07-07 | fonts/ 배포 폴더 신설, README 작성, GitHub 저장소 등록 | fonts/, README.md, docs/ | 사용자 요청 "fonts 폴더 저장 + 깃헙 업로드" |
| 2026-07-07 | v0.4.1: macOS name 테이블 수정 (ASCII psName + ko 로컬라이즈, 레거시 Mac 레코드 제외) | skills/font-build/scripts, fonts/ | Font Book "서체 이름 없음" 설치 오류 |
| 2026-07-07 | 하네스 동기화: 자모 조합 시스템·빌드 규칙·TTF 임베드 시안·릴리스 절차 반영 | skills 4종, agents 5종 (오케스트레이터 Phase 3·4 재작성) | 라운드 2~4 진화분 드리프트 해소 (/harness 유지보수) |

---
> Source: [revfactory/font-harness](https://github.com/revfactory/font-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
