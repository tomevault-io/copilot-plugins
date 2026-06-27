## tech-writer-skills

> IT 기술 문서(개발 가이드·API 레퍼런스·README·테크 블로그·기술 리포트·RFC/설계서·업무 문서)를 "정확하고 명확하고 실행 가능하게" 작성·윤문해주는 5인 파이프라인 하네스. 구조(헤딩·목록·표·코드블록)를 제거하지 않고 권장 자산으로 보강하며, 윤문만이 아니라 노트·요점에서 새 문서를 작성하는 하이브리드다. 한국어·영어 양방향.

# Tech Writer — IT 기술 문서 작성·윤문 하네스

## 프로젝트 개요

IT 기술 문서(개발 가이드·API 레퍼런스·README·테크 블로그·기술 리포트·RFC/설계서·업무 문서)를 "정확하고 명확하고 실행 가능하게" 작성·윤문해주는 5인 파이프라인 하네스. 구조(헤딩·목록·표·코드블록)를 제거하지 않고 권장 자산으로 보강하며, 윤문만이 아니라 노트·요점에서 새 문서를 작성하는 하이브리드다. 한국어·영어 양방향.

## 철칙

1. **정확성 최우선 (Accuracy First)** — 코드·명령·수치·식별자·버전·API 시그니처는 100% 정확. 윤문 시 사실 불변.
2. **실행가능성 (Runnable)** — 명령은 복사-실행 가능, 코드블록엔 언어 태그, 위험 명령엔 경고.
3. **구조는 자산 (Structure is Good)** — 헤딩·불릿·번호 목록·표는 제거 대상이 아니라 보강 대상.
4. **모호성 제로 (No Vagueness)** — "쉽게·간단히·여러·빠르게" 평가절하·막연 표현 제거.
5. **장르·독자 유지 (Tone Match)** — 입력 장르·독자 레벨 이탈 금지.
6. **날조 금지 (No Fabrication)** — 작성 모드에서도 없는 사실을 지어내지 않음. 불명확하면 `<!-- TODO -->`.

## 설계 원칙 (정체성)

| 축 | tech-writer 원칙 |
|---|---|
| 철학 | 구조 보강, 정확·명확·실행가능 |
| 불릿·헤딩·번호목록 | **권장 자산** (제거하지 않음) |
| 동작 | **작성 + 윤문 하이브리드** |
| 언어 | 한·영 양방향 |
| 검출 축 | 전제(G)·코드결함(C)·모호성(B)·용어일관(F) 등 기술 문서 고유 축 |

## 디렉토리 구조

이 프로젝트는 **빌드형 멀티 프로바이더** 구조다. `core/`가 단일 진실 원천(SSOT)이고, `providers/` 어댑터가 도구명·경로·매니페스트 차이를 흡수하며, `scripts/build.mjs`가 `dist/<provider>/`를 생성한다.

```
tech-writer-skill/
├── CLAUDE.md                      # 본 파일 (리포 개발 가이드)
├── README.md                      # 사용 안내
├── core/                          # ⭐ 단일 진실 원천 — 여기만 편집
│   ├── registry.json              # 스킬·오케스트레이터·에이전트 5종 메타 (description·tools·tier)
│   ├── orchestrator.md            # 파이프라인 로직 (토큰화 → SKILL.md/오케스트레이터 prompt)
│   ├── agents/                    # 에이전트 5종 프롬프트 본문 (frontmatter 없는 토큰화 md)
│   │   ├── tech-writer-monolith.md
│   │   ├── doc-quality-detector.md
│   │   ├── tech-doc-writer.md
│   │   ├── tech-fidelity-auditor.md
│   │   └── doc-clarity-reviewer.md
│   └── references/                # 룰북 3종 (프로바이더 무관, verbatim 복사)
│       ├── quick-rules.md         # Fast 전용 슬림 룰북 + 등급/변경률 정본 SSOT
│       ├── tech-doc-taxonomy.md   # Strict 전용 전체 분류 체계
│       └── tech-writing-playbook.md # Strict 전용 장르별 골격 + 치환 레시피
├── providers/
│   ├── claude/                    # provider.json(토큰맵·model:opus) + snippets/
│   └── kiro/                      # provider.json(토큰맵·model:default) + snippets/
├── scripts/build.mjs              # core + provider → dist (토큰 치환·include·미해결 검출)
├── install.sh                     # dist/<provider> → ~/.claude 또는 ~/.kiro
├── dist/                          # 빌드 산출물 (생성물 — 직접 편집 금지)
│   ├── claude/  skill/SKILL.md · skill/references/ · agents/*.md (5종)
│   └── kiro/    skills/tech-writer/ · agents/*.json (오케스트레이터 + 워커 5종)
└── _workspace/                    # 런타임 산출물 (run_id별, gitignored)
    └── {YYYY-MM-DD-NNN}/          # 01_input.txt … final.md
```

> **편집 규칙:** 콘텐츠는 `core/`에서만 고친다. 도구명·경로·매니페스트 포맷 차이는 `providers/<id>/`가 흡수한다. 수정 후 `node scripts/build.mjs`로 `dist/`를 재생성한다. `dist/`를 직접 고치면 다음 빌드에서 덮어쓰인다.
>
> 토큰 규약: `{{TOKEN}}`(예: `{{T_READ}}`·`{{SKILL_DIR}}`·`{{VERSION}}`)은 `provider.json`의 `tokens`로 치환(`VERSION`은 빌더가 `registry.skill.version`에서 주입), `{{include:NAME}}`은 `providers/<id>/snippets/NAME.md`를 삽입. 단일 중괄호(`{run_id}` 등)는 토큰이 아니다.
>
> **dist 커밋 정책:** `dist/`는 **커밋 대상**이다(설치 시 Node 불필요). 따라서 `core/`·`providers/`를 고치면 **반드시 재빌드 후 함께 커밋**한다. `node scripts/build.mjs --check`가 메모리 빌드 결과를 커밋된 `dist/`와 대조해 drift(내용 불일치·누락·stale 파일)를 비0 종료로 잡으며, `.github/workflows/build-check.yml`이 PR·push마다 이를 강제한다.

## 파이프라인

```
입력 (노트 또는 초안)
    ↓
[모드 판별] write | polish | hybrid · KR | EN · 장르
    ↓
[Fast] tech-writer-monolith — 단일 호출 (작성 or 탐지+윤문+자체검증)
   또는
[Strict]
[doc-quality-detector] — 탐지 (span·category·severity·suggested_fix)
    ↓
[tech-doc-writer] — 작성/윤문 (finding 기반 + 장르 골격)
    ↓
[병렬 팀]
    ├─ [tech-fidelity-auditor] — 코드·수치·식별자 정확성 감사 (14항)
    └─ [doc-clarity-reviewer]  — 잔존 결함 + 실행가능성 + 과윤문 판정
    ↓
[오케스트레이터 종합]
    ├─ accept → final.md
    ├─ rewrite_round_2 → 작성가 재호출 (최대 3회)
    ├─ rollback_and_rewrite → 문제 edit 롤백
    └─ hold_and_report → 사람 검토 권고
```

## 5인 핵심 팀

1. **tech-writer-monolith** — Fast 단일 호출. 작성·윤문·자체검증을 한 콜에서.
2. **doc-quality-detector** — 탐지기. 제거 결함(A·D·H·I)과 보강 결함(B·C·E·F·G)을 함께 span으로.
3. **tech-doc-writer** — 작성·윤문가. 노트→문서 작성 또는 finding 기반 수술적 윤문.
4. **tech-fidelity-auditor** — 정확성 감사관. 코드·수치·식별자·API 시그니처 14항 검증 → 롤백 지시.
5. **doc-clarity-reviewer** — 명확성 리뷰어. 탐지기 재실행으로 잔존·실행가능성·과윤문 계측. 등급 판정.

## 심각도 기준

- **S1 결정적**: 한 번만 나와도 신뢰를 깨거나 독자가 못 따라 하게 만드는 결함. 무조건 수정.
- **S2 강함**: 1~2회 허용, 반복 시 수정.
- **S3 약함**: 다른 패턴과 중첩될 때만 문제.

## 품질 등급

등급 정의의 **단일 진실 원천(SSOT)** 은 `core/references/quick-rules.md`의 "등급 기준 (정본 SSOT)"이다. 여기서는 재정의하지 않고 요약만 둔다(충돌 시 룰북 우선). write 모드는 변경률(자체검증 2항)을 제외해 6항으로 채점한다.

- **A**: S1 0건, S2 2건 이하, 자체검증 7항 통과(write 6항)
- **B**: S1 0건, S2 4건 이하, 자체검증 6항 이상(write 5항+)
- **C**: S1 1~2건 또는 자체검증 미달 — 2차 작업
- **D**: S1 3건 이상 또는 변경률 60% 초과(윤문 모드) — 사람 검토

## 사용 방법

1. 새 세션에서 오케스트레이터 스킬 트리거:
   - 윤문: `이 기술 문서 명확하게 다듬어줘:` (텍스트 첨부)
   - 작성: `이 노트로 API 문서 작성해줘:` (요점 첨부)
2. 오케스트레이터가 모드(write/polish)·언어·장르를 판별하고 run_id 생성 후 파이프라인 실행.
3. 결과 `final.md` 반환 (끝에 `<!-- TECHWRITER-SUMMARY -->` 메타 블록).

## 파일 시스템 접근 규칙

전용 도구 우선. `Bash`의 `ls`·`cat`·`echo`는 환경에 따라 동작이 달라져 예측 불가.

| 작업 | 올바른 방법 | 피할 방법 |
|---|---|---|
| 파일 존재 확인 | `Glob` | `Bash ls` |
| run_id 시퀀스 조회 | `Glob(pattern="_workspace/*/01_input.txt")` | `Bash ls` |
| 파일 읽기 | `Read` | `Bash cat` |
| 파일 쓰기·편집 | `Write` / `Edit` | `Bash` 리다이렉션 |

## 주요 금기

- 코드·명령·플래그·환경변수·경로·식별자 변경 금지.
- 수치·버전·날짜·단위·벤치마크 값 변경 금지.
- API 시그니처(파라미터·타입·순서) 변경 금지.
- 코드블록·인라인 코드 내부 변경 금지.
- 큰따옴표 인용·라이선스·법률 조문 변경 금지.
- 구조(헤딩·목록·표) 임의 제거 금지 — 보강만.
- 작성 모드에서 없는 사실 날조 금지.

## 확장 포인트

- **장르 확장**: 현재 6장르(가이드·API·README·블로그·리포트·업무). 학술 논문·튜토리얼 시리즈·체인지로그 추가 가능.
- **언어 확장**: 일본어·중국어 기술 문서로 확장 시 언어별 번역투 패턴 분리.
- **린트 연동**: markdownlint·vale 등 외부 린터와 결합해 I(위생) 자동 검증 가능.

## 참고

- 슬림 룰북: `core/references/quick-rules.md`
- 분류 체계: `core/references/tech-doc-taxonomy.md`
- 작성·윤문 처방: `core/references/tech-writing-playbook.md`

---
> Source: [TeiNam/tech-writer-skills](https://github.com/TeiNam/tech-writer-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
