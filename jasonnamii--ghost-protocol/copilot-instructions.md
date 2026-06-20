## ghost-protocol

> |


<!-- Triggers
P1: 고스트프로토콜, ghost-protocol, ghost protocol, 기획핸드오프, 핸드오프패키지, PRD와기능정의, 기획패키지, 피그마기획, 기획Git패키지.
P2: 고스트프로토콜 실행, PRD랑 기능정의 리스트 만들어줘, 이 피그마로 PRD와 기능 리스트 뽑아줘, 컨셉을 개발자가 볼 PRD로, 이 PDF를 PRD와 기능 리스트로 바꿔줘, Git에 올릴 구조로, 고스트프로토콜 업데이트, 고스트프로토콜 갱신, {기능} 업데이트, 결정 반영해줘(기존 패키지 갱신 → Update Mode), QA 만들어줘, QA 기준 뽑아줘, 디자인·UX QA, 미결 점검, 미결 리마인드, 까먹은 결정 있어?(QA·미결 추적).
P3: planning handoff package, PRD + feature list, concept to PRD, figma to PRD, build planning package.
P5: Git 구조로, /docs/planning 으로, md+csv로, 엑셀로도, 7파일로(+qa).
NOT: 화면→기능 후보만(→feature-inventory), 정책·예외 설계 검증만(→planning-review), 범용기획 발산(→planning-skill), 정책기획(→policy-planning), UI비주얼(→ui-designer), 사업전략(→biz-skill).
GATE: "컨셉/화면/문서가 있고 → 개발자·디자이너가 바로 받아 착수할 기획 패키지(PRD + 기능정의)가 필요하다"는 상황에서 발동. 입력이 컨셉 한 줄이라도 있으면 실행한다(부족해도 멈추지 않음).
-->

# 고스트프로토콜 — ghost-protocol (입력 → PRD·기능정의 동시 산출 → Git 패키지)

**핵심 공식**: 고스트프로토콜은 컨셉·화면·기존 문서를 **개발 착수 가능한 PRD와 기능정의 리스트로 바꾸는 기획 핸드오프 스킬**이다. 핵심은 많이 쓰는 것이 아니라 **빠짐없이 닫는 것**이다. PRD는 제품 의도와 구현 범위를 정리하고, 기능정의 리스트는 개발자가 작업 단위를 볼 수 있게 만든다. 두 문서는 F-ID와 FR-ID로 연결한다. 가치는 "문서를 많이 만드는 것"이 아니라 **근거(화면에서 본 것)·의도(컨셉에서 온 것)·제안(AI가 보강한 것)·미결정(사람이 정할 것)을 절대 섞지 않는 것**에 있다.

> **이 스킬은 PRD 자동작성기도 기능정의 자동화기도 아니다.** 두 산출물을 ID로 연결하고 Git 패키지로 닫는 **단일 핸드오프 엔진**이다. PRD만 만들면 실패, 기능 리스트만 만들면 반쪽이다. 그리고 PRD는 **구현 지시서가 아니라 개발 착수 기준 문서**다 — 긴 문서가 목표가 아니라 개발 착수에 필요한 최소 완결성이 목표다.

## 내부 6엔진 (한 흐름으로 잇는다)

두 참고 스킬(feature-inventory·planning-review)을 **병렬 호출하지 않는다. 내부 엔진으로 흡수**해 한 흐름으로 잇는다.

```
ghost-protocol
├─ 1. 입력 해석 엔진     — 컨셉/이미지/피그마URL/PDF/기존문서/혼합 판별·정규화 → references/input-normalization.md
├─ 2. 컨텍스트 로드 엔진 — (요청 시) GitHub 원격 최신 코퍼스 fetch·읽기·레지스트리 → references/platform-context.md · scripts/corpus_tools.py
├─ 3. 기능 추출 엔진     — 화면 있으면 화면 기반 역추출, 컨셉만이면 컨셉 기반 초안. 15컬럼 → references/extraction-rules.md · feature-column-spec.md
│                          (feature-inventory 흡수 — 화면 전제는 상위 레이어에서 완화)
├─ 4. PRD 검증·작성 엔진 — 정책·예외·상태 내부 점검 + 레드플래그 + 컴플라이언스 → 기본 12섹션 PRD(+ 선택 섹션)
│                          → references/policy-exception-checklist.md · prd-template.md (planning-review 흡수, 웨이트 게이트 제거)
├─ 5. 결정 엔진         — 못감-정보부족 / 못감-결정부족 / 기획 확인 필요 + (요청 시) 플랫폼 정합성 → references/decision-log-template.md
└─ 6. Git 패키징·사후검토 — 7파일 동시 산출(+qa.md) + F↔FR + _registry + 미결 추적(track_open) + (요청 시) Git 대조 사후검토 + 커밋 안내 → references/git-output-rules.md · platform-context.md
```

핵심은 **feature-inventory → planning-review를 따로 부르지 않고**, 고스트프로토콜 안에서 입력 정규화 → (요청 시) 코퍼스 로드 → 기능 추출 → PRD 검증 → 결정 분리 → Git 패키징을 **한 번에·멈추지 않고** 잇는 것이다.

## Skill Boundaries

- **하는 것** — 어떤 입력이든(컨셉/이미지/피그마/PDF/기존문서/혼합) 받아 → 제품·기능 모델 정리 → 화면·컨셉에서 기능 후보(15컬럼) 역추출 → 정책·예외·상태 내부 점검 + 레드플래그·컴플라이언스 스캔 → **기본 12섹션 PRD와 기능정의 리스트를 동시 생성** → FR↔F ID 연결 → 미결정 분리 → README·PRD·features.md·features.csv·decisions.md·qa.md·changelog.md **7파일**을 Git 패키지로 산출(엑셀은 옵션). qa.md는 PRD·features에서 도출한 합격 기준(디자인 완성도·사용자 경험/모션 포함)을 담는 별도 검증 문서이고, 열린 미결은 닫힐 때까지 추적한다.
- **안 하는 것** — 화면만 보고 기능 후보만 뽑는 단일 작업(→`feature-inventory`), 이미 있는 기획의 정책·예외 설계 검증만(→`planning-review`), 범용 기획 발산·아이디어 단계(→`planning-skill`), 정책기획·공약(→`policy-planning`), 화면 UI 비주얼 디자인(→`ui-designer`·`apple-canvas`), 사업전략(→`biz-skill`), 약관·개인정보 등 **법무 문서 생성**(감지·라우팅만, 생성은 →`app-and-jang`·`ip-skill`). 그리고 **개발 구현 방식 지시**(DB 테이블 구조·API 엔드포인트·컴포넌트 세부 설계·픽셀 UI)는 쓰지 않는다.

---

## When to Use

- 컨셉 문장·와이어프레임·피그마 URL·PDF·기존 기획서가 있고, **개발자·디자이너가 바로 받아 착수할 기획 패키지(PRD + 기능정의)**가 필요할 때
- "고스트프로토콜 실행", "PRD랑 기능정의 리스트 만들어줘", "이 피그마로 PRD와 기능 리스트 뽑아줘"로 발동
- 화면은 있는데 PRD가 없거나, 컨셉만 있는데 개발 가능한 기준 문서로 닫아야 할 때
- **안 쓸 때** — 화면 보고 기능 후보 목록 1개만 빠르게(→`feature-inventory`), 이미 있는 기획의 구멍만 메우기(→`planning-review`), 아직 아이디어 발산 단계(→`planning-skill`)
- **이미 만든 패키지를 갱신할 때** → 아래 **Update Mode**(첫 생성과 같은 스킬, 다른 진입)

## Update Mode — 기존 패키지 갱신 (첫 생성 이후)

첫 생성으로 끝이 아니다. 컨셉 초안은 미결정이 많고, 사람이 **결정을 내리고 기능을 더하면서** 여러 번 갱신된다. 그 입구가 **`decisions.md` 답안지**다(별도 입력 파일 없음 — 결정 답안지가 곧 갱신 입력칸). → 상세 `references/update-cycle.md`·`decision-log-template.md`.

- **발동**: "고스트프로토콜 업데이트 {feature}", "고스트프로토콜 갱신", "{feature} 업데이트", "결정 반영해줘". 대상 패키지가 `mnt/outputs/planning/{feature}/`(또는 repo)에 이미 있으면 Update Mode.
- **사용자 흐름**: decisions.md에서 결정은 객관식(`▶ 내 결정: A/B`), 추가·수정·뺄것·새입력은 ➕ 빈칸을 채운다 → "고스트프로토콜 업데이트 {feature}".
- **스킬 처리(한 흐름)**:
  0. **묵힌 미결 환기**: `python3 scripts/track_open.py --root mnt/outputs/planning --today {오늘}` 1회 → 3일+ 안 닫힌 미결을 응답 맨 위에 먼저 띄운다(이 패키지뿐 아니라 형제 패키지 것도). "3일 전에 정하기로 해놓고 잊은 것"을 지금 환기(`update-cycle.md §6`).
  1. 7파일 읽고 decisions.md에서 **답이 채워진 항목만** 추림(없으면 무엇을 갱신할지 1줄 확인).
  2. 종류별 반영. **결정(D-n→안)은 PRD §8/§9·features 정책 칸을 미정→확정, decisions의 그 항목은 ✅ 결정됨 표로 이동. 기능 추가는 새 F/FR ID(기존 최대+1, 재사용 금지). 수정/범위변경/새입력**은 해당 위치만(`update-cycle.md §3`).
  3. **근거·의도·제안·미결정 분리(절대규칙 4)·F↔FR 연결(절대규칙 6) 유지.** PRD·features를 직접 손대지 말고 decisions.md 답안을 통해서만 반영(직접 편집은 ID·분리가 깨짐).
  4. changelog.md 새 버전 항목(날짜+변경+영향 ID), `version`·`updated` 올림, `status` 재판정(미결정 닫히면 draft→review).
  5. decisions.md ✅ 결정됨 정리(열린 목록에서 제거, 닫힌 항목의 📅 줄은 날짜로 흡수), ➕ 처리분 비움, **새로 연 미결엔 📅 opened={오늘}**. 영향 큰 결정·범위변경은 반영 전 한 줄로 영향 짚고 확인.
  6. **qa.md 재생성**(닫힌 결정·확정 화면이면 묶인 보류 QA를 미검증으로 풀고 합격선 확정, 새 FR엔 QA 추가 — `update-cycle.md §6`) + 재검증(validate) + (코퍼스 활성 시 정합성 재대조) + (요청 시 커밋/푸시 명령).

## QA 기준 · 미결 추적 (별도 레이어 — 무엇·왜)

핸드오프는 "개발 착수"에서 끝나지 않는다. 두 가지를 더 둔다(실행 절차는 위 Phase 0·6·7·Update Mode, 깊이는 `qa-template.md`·`update-cycle.md §6`).

- **QA(`qa.md`)** — 무엇을 만드나는 PRD·features에, **무엇으로 합격을 판정하나**는 qa.md에. 최종 PRD(§7 수용 기준·§8·§9)와 features에서 기능별 합격 기준을 펼치고, **디자인 완성도와 사용자 경험(모션·전환·로딩·빈/실패 화면·반응성·동작 줄이기 등)을 별도 축**으로 본다. 화면 미확정 항목은 보류로 두고 decisions의 I-n과 묶어 추적한다. 코퍼스 활성 시 다른 패키지의 공통 기준(차단·세션·삭제·모션 언어)과 어긋나지 않게 맞춘다.
- **미결 추적** — 올렸다고 끝이 아니라 닫혀야 끝이다(절대규칙 9). decisions.md 열린 항목에 **📅 올린 날짜**를 달고, 세션 시작·갱신 때 `track_open.py`가 "N일째 안 닫힌 미결 M건"을 자동으로 다시 띄운다 — 3일 전에 정하기로 해놓고 묻힌 결정을 다음 작업 때 환기한다. 능동 리마인드가 필요하면 `/schedule`·셸 alias로 거는 절차를 안내한다(`update-cycle.md §6`).
- **GNB 기준선(`_nav.md`)** — 모든 패키지의 대전제는 앱 GNB·유틸 메뉴 트리다. 실행 시 맨 먼저 현재 GNB를 선제시·확인하고(Phase 0 step 1b), 각 패키지 `메뉴` 컬럼이 GNB 항목에 매핑된다. `nav_check.py`가 **월초(달 바뀜)·메뉴 변경**을 세션 시작 시 환기한다(코퍼스 비의존·항상 존재 — `nav-anchor.md`).
- **발동**: "QA 만들어줘 / QA 기준 / 디자인·UX QA"(qa.md), "미결 점검 / 까먹은 결정 있어?"(추적), "GNB 보여줘 / 메뉴 구성 / 내비 점검"(IA 앵커). 첫 생성·Update Mode에서는 묻지 않고 함께 돈다.

## Prerequisites

| # | 체크 | 미충족 시 |
|---|------|-----------|
| 1 | **입력 최소 1개** (컨셉 한 줄·이미지·피그마 URL·PDF·기존 문서 중 하나) | 컨셉 한 줄이라도 있으면 진입. 전혀 없으면 "한 줄로 무엇을 만들지 말해달라" 후 진입 |
| 2 | **feature-name 확정** (패키지 폴더명) | 입력에서 유추해 제안하고 1줄 확인. 미응답 시 영문 kebab-case로 가정 |
| 3 | 대상 앱·프로파일·플랫폼 식별 | 가정 명시(디폴트 Cre8orClub/C8, 6파트: iOS·AOS·BE·FE·디자인·기획) 후 진행 |
| 4 | 피그마 URL 직접 열기 가능 여부 | 못 열면 멈추지 않고 이미지/PDF 업로드 대체 안내 + 가능한 범위로 진행 |
| 5 | references/ 폴더 접근 가능 | inline fallback |
| 6 | **앱 GNB·유틸 메뉴(`_nav.md`) 선제시** | 있으면 1줄 확인, 없으면 입력·코퍼스에서 후보 추출해 제안하고 가정. 멈추지 않음(절대규칙 9·`nav-anchor.md`) |

## ⛔ 절대 규칙 (8)

| # | 규칙 | 이유 |
|---|------|------|
| 1 | **PRD와 기능정의 리스트는 항상 함께 만든다** | 하나만 만들면 실패, 반쪽이면 핸드오프 불가. 둘을 ID로 묶어야 개발자가 헷갈리지 않는다 |
| 2 | **PRD는 구현 지시서가 아니라 개발 착수 기준 문서다** | DB 테이블 구조·API 엔드포인트·컴포넌트 세부 설계·픽셀 단위 UI 지시는 쓰지 않는다. 단 개발자가 범위를 판단할 **데이터 항목·시스템 응답은 후보 수준**으로 적는다. 목표는 긴 문서가 아니라 **개발 착수에 필요한 최소 완결성** |
| 3 | **기능정의 리스트는 개발 작업 단위를 보여주는 표다** | 행 1개 = 사용자가 실제로 하는 행동 1개 또는 시스템이 반드시 처리할 동작 1개. 버튼·텍스트·아이콘을 그대로 행으로 만들지 않는다 |
| 4 | **근거·의도·제안·미결정을 섞지 않는다** | 화면에서 본 것=근거, 컨셉에서 온 것=의도, AI 보강=제안, 사람이 정할 것=미결정. 섞이면 개발자가 "확정 기획"으로 오해해 사고 난다 |
| 5 | **정책·예외·상태는 근거 없으면 확정하지 않는다** | 화면·문서에 근거가 없으면 임의로 만들지 않는다. `미정` 또는 `결정 필요`로 둔다(`feature-column-spec.md`·`policy-exception-checklist.md`) |
| 6 | **모든 기능 F-ID는 PRD 요구사항 FR-ID와 연결한다** | features의 `연결 PRD 요구사항 ID` ↔ PRD §7 기능 요구사항 ID. 양쪽이 서로를 가리켜야 한다 |
| 7 | **미결정은 PRD 본문에 흩뿌리지 않고 decisions.md에 모은다** | 미정·갈림길은 §12와 `decisions.md`로. 결정마다 관련 F/FR ID를 붙인다 |
| 8 | **Git 코퍼스는 가능하면 읽되, 접근 불가 시 멈추지 않고 경고만 남긴다** | 코퍼스 정합은 정합성을 높이는 **선택 레이어**다. "Git 구조로/레포 기준으로/기존 기획과 맞춰줘" 요청이 있으면 우선 활성화. 못 읽어도 멈추지 않고 `decisions.md`에 미확인 경고를 남긴다 |
| 9 | **미결정은 닫힐 때까지 추적한다** | decisions.md 열린 D/I/C에 **📅 올린 날짜**를 달고, 세션 시작·갱신 때마다 `track_open.py`로 점검해 며칠 지나도 안 닫힌 건 자동으로 다시 알린다(`update-cycle.md §6`). 올렸다고 끝이 아니라 닫혀야 끝이다. 그리고 **qa.md(합격 기준)는 PRD와 함께 내는 별도 검증 문서**다 — PRD는 무엇을 만드나, qa.md는 무엇으로 통과를 판정하나(디자인 완성도·사용자 경험 포함) |

> 위 9개가 핵심 절대 규칙이다. 그 밖의 운영 규칙(입력 부족해도 멈추지 않음 · 상태값 PM 프레임 고정 · 본문 내부어 금지 · 산출물 본문 입니다체 · 피그마 fallback · QA는 별도 qa.md로 본문 7파일)은 아래 파이프라인과 references/ 문서에서 강제한다.

---

## 입력별 산출 깊이 (자동 조절)

입력 수준에 따라 깊이를 자동으로 조절한다. 멈추지 않되, 약한 입력은 미결정을 더 많이 남긴다.

| 입력 | 동작 유형 | PRD | 기능정의 | decisions |
|---|---|---|---|---|
| 컨셉만 | 기획 초안 생성형 | 구조 중심 초안(draft) | 기능 후보 중심 | 많음 |
| 와이어프레임/이미지 | 기능 역추출형 | 화면 흐름 포함 | 근거 강함 | 중간 |
| 피그마 URL | 기능 역추출형 | 화면·플로우 반영 | 가장 강함 | 중간 |
| PDF/기존 기획서 | 기획 보강형 | 기존 문서 보강 | 문서 기반 추출 | 중간 |
| 혼합(피그마+컨셉 등) | 통합 핸드오프형 | 가장 좋음(컨셉=의도, 화면=근거) | 가장 좋음 | 적음 |

---

## 웨이트·PRD 상태 (게이트 없이 자동)

고스트프로토콜은 단일 흐름으로 **멈추지 않고** 실행한다. 중간에 라이트/표준/헤비를 묻지 않는다.

- **웨이트 기본값 = Standard.** 묻지 않는다. 단 **위험 기능**(결제·개인정보·미성년·저작권/UGC·권한·삭제·신고·차단)이면 산출은 그대로 진행하되 `decisions.md`에 **"헤비(꼼꼼) 검토 권장"** 1줄을 남긴다.
- **PRD 상태(§0)는 입력·결정 수준으로 자동 판정**한다:

| 상태 | 판정 |
|---|---|
| **Draft** | 입력 부족(컨셉만 등), 미결정 많음 |
| **Review** | 골격은 섰으나 주요 정책·예외 확인 필요 |
| **Approved** | 사람이 미결정을 닫아 개발 착수 가능 (스킬이 임의로 Approved 부여 ✗ — 사람 확인 후) |

> 컨셉만 받아도 PRD는 나온다 — 단 상태는 Draft, 빈 곳은 미결정으로 `decisions.md`에. "결정이 끝나야 PRD"가 아니라 "상태를 붙여 멈추지 않는다".

---

## 실행 파이프라인 (Phase 0~7)

### Phase 0 — 입력 해석
1. 입력 유형 판별: 컨셉 / 이미지 / 피그마 URL / PDF / 기존 문서 / 혼합. → `references/input-normalization.md`
1b. **GNB·유틸 메뉴 선(先)제시(IA 앵커 — 대전제, 절대규칙 9)**: 기능 추출 전에 앱 IA부터 세운다. 상위 `mnt/outputs/planning/_nav.md`가 있으면 GNB를 한 줄로 심플 제시 + "이 메뉴 구성 맞나요? 수정할 곳?" 1줄 확인. 없으면 멈추지 말고 입력·코퍼스 `메뉴` 집합·컨셉에서 후보를 뽑아 **제안 초안**으로 제시 + 같은 확인(미응답 시 가정·진행). 모든 패키지의 `메뉴` 값은 이 GNB 항목에 매핑된다. → `references/nav-anchor.md`. (step 6처럼 있으면 띄우고 없으면 조용히, 새 작업을 막지 않음)
2. 피그마 URL은 직접 열기를 시도하되, **못 열면 멈추지 않고** 이미지/PDF 업로드 대체를 안내하고 URL·텍스트 기준으로 가능한 구조를 만든다.
3. **feature-name 확정**(패키지 폴더명, 영문 kebab-case 권장: `profile`, `portfolio`).
4. 대상 앱·프로파일·플랫폼·파트 구성 확정(디폴트 C8 6파트).
5. **Git 코퍼스 활성화 판단**(절대규칙 8): "Git에 올릴 구조로 / 기존 기획과 맞춰줘 / 레포 기준으로 정합성 봐줘" 요청이 있으면 코퍼스를 읽는다. 조직·레포·경로는 고정값(`cre8orclub` · `cre8orclub/planning` · `docs/planning`). 단 **GitHub 로그인은 사용자마다 다르므로**, Git 단계에 처음 진입하면 `scripts/personalize.py get`으로 개인 로그인을 읽고 없으면 **1회 개별화**한다(`git-personalize.md`). 요청이 없으면 로컬 패키지만 만들고 정합성 미확인 경고를 남긴다.
6. **묵힌 미결 환기(절대규칙 9)**: 대상 feature 또는 코퍼스에 기존 패키지가 있으면, 새 작업 들어가기 전 `python3 scripts/track_open.py --root mnt/outputs/planning --today {오늘}`를 돌려 **3일+ 안 닫힌 미결**을 응답 맨 위 3~6줄로 먼저 띄운다(`update-cycle.md §6`). 같은 자리에서 `python3 scripts/nav_check.py --root mnt/outputs/planning --today {오늘}`도 1회 — **월초(달 바뀜)·GNB 변경 감지**를 환기한다(`nav-anchor.md`). 없으면 조용히 건너뜀 — 새 작업을 막지 않는다.

### Phase 1 — 플랫폼 컨텍스트 로드 (요청 시)
**코퍼스 정합은 선택 레이어다(절대규칙 8).** "Git 구조로 / 레포 기준 정합성" 요청이 있을 때만 산출 전에 조직이 GitHub에 올린 최신 기획 문서들을 읽는다. → `references/platform-context.md`.
1. `scripts/corpus_tools.py fetch --repo cre8orclub/planning --out mnt/outputs/planning/.corpus` — **원격 기본 브랜치**에서 코퍼스를 가져온다(로컬 작업본이 아니라 원격이 최종본). 다른 레포면 `--repo`만 바꾼다.
2. `_index.md`·`_registry.md`·형제 `{feature}/PRD.md`·`features.md`를 **직접 읽어** 맥락 파악.
3. `scripts/corpus_tools.py scan`으로 레지스트리(F/FR ID 색인·상태) 구성. 공유 엔티티·횡단 정책·용어는 읽으며 정리.
4. **fallback**: `gh` 미설치/미인증/접근 불가 → 멈추지 말고 빈 코퍼스로 진행 + `decisions.md`에 "⚠️ Git 최신 코퍼스 미확인" 경고 + 인라인 고지.

### Phase 2 — 제품·기능 모델 정리
입력을 의미 모델로 재구성한다(`policy-exception-checklist.md` §0 참조). 채우기 전에 이해부터.
- 제품 목적 / 사용자(행위자) / 핵심 객체 / 핵심 행동 / 이 기능이 성공하면 되는 상태(불변식).
- 본질을 한 줄로 규정("경합 자원 배분 / 양면 신뢰 거래 / 공개 범위 제어"). 이후 모든 점검·사례는 이 모델 위에서 작동.

### Phase 3 — 기능 추출 (15컬럼)
화면·컨셉·문서에서 기능 후보를 역추출한다. → `references/extraction-rules.md`(3등급·화면읽기·동사/명사 변환·행 단위 기준) + `feature-column-spec.md`(15컬럼).
- **근거·의도·제안·미결정 분리**(절대규칙 4): 화면에서 본 것 → `근거`, 컨셉 의도 → 근거에 `의도:` 표기, AI 추정 → 추정으로, 정할 것 → 미정.
- **행 1개 = 행동 1개**(절대규칙 3): 사용자 행동=동사형, 기능명=명사형. 시스템이 반드시 처리할 동작(알림 발송·자동 저장·신고 접수 상태 생성 등)도 행으로 적되, 백엔드 내부 구현 단계는 행으로 만들지 않는다.
- 정책·예외·상태는 근거 없으면 `미정`/`결정 필요`(절대규칙 5). 다음 화면은 흐름 보일 때만.
- **레지스트리 대조**(코퍼스 활성 시): ID는 패키지별로 유지(새 패키지도 `F-001`부터 — 정상). 다른 패키지 기능을 가리킬 땐 **전역ID `feature#F-001`**. 이미 정의된 기능은 다시 만들지 말고 참조.

### Phase 4 — 정책·예외·상태 내부 점검
기능 후보를 정책·예외·상태 **내부 점검표**에 통과시킨다. → `references/policy-exception-checklist.md`. **이 점검표는 PRD 본문을 길게 만들기 위한 것이 아니라, 정책·예외·상태의 누락을 찾는 내부 도구다. 산출물에는 필요한 항목만 쉬운 말로 반영한다.**
1. **레드플래그 선행 스캔**(R1~R18): 치명 패턴(이중결제·권한 우회·즉시영구삭제·다크패턴 등)을 먼저 잡는다. 단 **PRD 본문에 R번호를 쓰지 않는다** — 사용자에게는 "꼭 정해야 할 것"으로 번역하고, 한 기능당 핵심 위험 1~3개만 노출, 나머지는 decisions.md에.
2. **정책·예외·경계·상태 점검**: 해피패스 각 문장을 네 갈래로 통과시켜 빠진 곳을 드러낸다(답이 없는 자리가 미결정).
3. **컴플라이언스·IP 점검**(C1~C14): 개인정보·UGC·미성년·결제·위치 등 법적 필수요구를 감지해 미결정/법무로 라우팅(생성 ✗). PRD 본문 기본 섹션이 아니라 **decisions.md의 법무 확인**으로 모은다("변호사 검토 필요" 1줄).
4. **시스템 파급**(위험·플랫폼 공통 기능일 때만): 상류·하류·미래·횡단을 본다. 코퍼스가 활성이면 기존 패키지의 횡단 정책(삭제 유예·차단 의미·세션·권한)과 **모순 금지**, 용어 통일.
- 근거 없는 정책·예외는 **확정하지 않고** 제안(A안 권장/B안)으로 두거나 미결정으로 넘긴다.

### Phase 5 — PRD 작성 (기본 12섹션 + 선택 섹션)
→ `references/prd-template.md`. **기본 12섹션**을 따른다. 위험 기능이나 복합 기능에만 선택 섹션을 더한다. 목표는 긴 문서가 아니라 개발 착수에 필요한 최소 완결성이다.

```
0 문서 정보 → 1 기능 한 줄 정의 → 2 사용자와 문제 → 3 목표 → 4 범위(포함/제외)
→ 5 진입점과 화면 흐름 → 6 핵심 사용자 시나리오 → 7 기능 요구사항(ID·요구사항·수용 기준·연결 기능 F-n)
→ 8 정책 → 9 상태와 예외 → 10 데이터/API 후보 → 11 맞물린 파트 → 12 미결정과 완료 기준
```

- **선택 섹션**(해당될 때만 붙임): 컴플라이언스/IP(원칙은 decisions.md 법무 확인) · 시스템 파급(위험·공통 기능일 때) · 추후 확장.
- §8 정책·§9 상태와 예외를 채울 때 정책·예외·상태 점검을 **검증 도구**로 쓰되(`policy-exception-checklist.md`), 근거 없는 것은 확정하지 않고 **§12 미결정**과 `decisions.md`로 분리한다.

### Phase 6 — 동시 산출 + ID 연결 + 레지스트리 갱신
7파일을 동시에 만든다(절대규칙 1·9). → `references/git-output-rules.md` · `decision-log-template.md` · `qa-template.md`.
1. **F↔FR 연결**(절대규칙 6): features의 `연결 PRD 요구사항 ID`와 PRD §7 ID를 서로 가리키게(`F-001 ↔ FR-001`). 다른 패키지 참조는 `feature#FR-001`.
2. `features.csv` ← `scripts/build_feature_csv.py` (15컬럼, UTF-8 BOM).
3. `README.md` + 상위 `_index.md` ← `scripts/build_index.py`.
3b. 상위 `_nav.md`(IA 앵커) — 없으면 GNB·유틸 후보로 생성(`status: proposed`), 있으면 이 패키지 `메뉴` 값이 GNB 항목에 매핑되는지 점검. **코퍼스 비의존·항상 존재**(`_index.md`와 동급, `nav-anchor.md`).
4. `decisions.md` — 사람이 결정할 것만(못감-정보부족 / 못감-결정부족 / 기획 확인 필요). **결정마다 관련 F/FR ID를 붙이고, 열린 D/I/C 제목 아래 `- 📅 opened: {생성일}` 줄을 박는다**(절대규칙 7·9 — track_open의 나이 계산 근거). due·owner는 위험·갈림길에만. **답안지형**으로 만든다(객관식 슬롯 + ➕ 빈칸 + ✅ 결정됨 표) — 이게 곧 갱신 입력칸(`decision-log-template.md`·`update-cycle.md`).
5. `qa.md` — 최종 PRD §7 FR·§8·§9·§12와 features에서 **합격 기준**을 도출. 8차원: 기능·정책·상태/예외·**디자인 완성도·사용자 경험/모션·접근성·성능·기능 간 정합성**. FR마다 A차원 QA 최소 1행, 화면 미확정(I-n)은 **보류**(닫히면 풀림), 코퍼스 활성 시 H차원에서 다른 패키지와 대조. PRD를 다시 쓰지 않고 "그게 됐는지 보는 법"만(`references/qa-template.md`).
6. `changelog.md` — 첫 생성 항목.
7. (코퍼스 활성 시) **`_registry.md` + `_features.csv`(통합 마스터) 갱신** ← `scripts/corpus_tools.py scan --new`. PRD는 메뉴·기능 단위로 **독립**이지만, CSV는 부분부분 올린 패키지들을 **플랫폼 전체로 통합**한다(맨 앞 `패키지`·`전역ID` 컬럼).
8. "엑셀로 줘" 요청 시에만 `scripts/build_xlsx.py`로 `features.xlsx` 추가.

### Phase 7 — 사후검토 + 패키징
1. **결정적**: `scripts/validate_planning_package.py --pkg ...` → 7파일(qa.md 포함)·12섹션·15컬럼·상태값·F↔FR·미결정 연결·금지어. 코퍼스 활성 시 `--corpus ...`로 끊긴 교차참조·_index/_registry 누락도 검사.
2. **맥락적(코퍼스 활성 시)**: 새 패키지가 기존 문서와 모순? 기능 중복? 교차참조 실재? 공유 엔티티·횡단 정책 편입? **다른 패키지가 갱신돼야 하나?**
3. 발견을 `decisions.md`의 **"플랫폼 정합성 / 다른 문서 갱신 필요"** 섹션에 + 사용자에게 **인라인 정합성 리포트**. **다른 패키지 문서는 표시만**(수정안 생성 ✗).
3b. **미결 추적**: `python3 scripts/track_open.py --root mnt/outputs/planning --today {오늘}`(코퍼스 활성 시 `.corpus` 자동 포함)로 이번·형제 패키지의 열린 미결을 날짜와 함께 리포트하고, 3일+ 묵힌 건 환기한다. 열린 D/I/C에 📅 올린 날짜가 없으면 오늘 날짜로 채운다(`update-cycle.md §6`). 미결이 남으면 산출 끝에 "일정 걸까요?"를 한 줄 제안.
4. **커밋·푸시·PR 명령어 출력만**(실행 ✗ — 사용자가 실행): 정본 레포 `cre8orclub/planning`(고정), 대상 `docs/planning/{feature}/`, 브랜치 `docs/planning-{feature}`, 커밋 `docs(planning): add PRD and feature inventory for {feature}`. **고스트프로토콜은 조직업무이므로 항상 본인 cre8orclub 계정 `{account}`로 올린다**(`{account}`·`{work_dir}`·`{repo_url}`는 개인 로그인 설정 — `personalize.py get`). 푸시 계정은 `{repo_url}` 방식이 고정한다(SSH host-alias 권장 — 전환 없이 자동, 개인 계정 안 건드림). 설정이 없으면 먼저 1회 개별화(`git-personalize.md`). 전체 명령은 `git-output-rules.md §0·§4`.

| 체크 | 내용 |
|------|------|
| 동시 산출 | PRD와 features가 둘 다 나왔나 (하나만 = 실패) |
| 최소 완결성 | PRD가 구현 지시(DB·API·픽셀)로 부풀지 않고 개발 착수 기준에 머물렀나 |
| ID 연결 | 모든 F↔FR 연결, 다른 패키지 참조는 `feature#FR-001`(끊김 없나) |
| 행 단위 | features 한 행이 행동/시스템 동작 1개인가(버튼 나열 ✗) |
| 미결정 연결 | 모든 미정이 decisions.md에 있고 관련 F/FR ID·📅 올린 날짜가 붙었나 |
| 근거 분리 | 근거·의도·제안·미결정이 섞이지 않았나 |
| QA·미결 | qa.md가 FR마다 합격 기준을 덮고 디자인 완성도·UX/모션 차원이 있나, 미확정 화면은 보류로 묶였나, 열린 미결에 날짜가 붙었나 |
| 7파일 | 7파일(qa.md 포함) + (코퍼스 활성 시) _index·_registry 갱신됐나 |
| 출력 청결 | 본문에 AI·프롬프트·엔진 용어(4축·레드플래그·R번호·디텍터·역검증 등)가 없나(qa.md 포함) |

---

## Output Path

```
mnt/outputs/planning/{feature-name}/
├── README.md       # 패키지 첫 화면 — 무엇을 보면 되는지
├── PRD.md          # 제품 요구서 기본 12섹션(+ 선택 섹션)
├── features.md     # 기능정의 리스트(15컬럼, 사람이 읽는 표)
├── features.csv    # 필터·자동화용 데이터(보조 정본)
├── decisions.md    # 결정 답안지(객관식 A/B·➕빈칸) + 열린 미결엔 📅 올린 날짜(추적 기준). "고스트프로토콜 업데이트 {feature}"
├── qa.md           # 합격 기준 — 기능별 + 디자인 완성도 + 사용자 경험(모션·전환·로딩·빈/실패). PRD·features에서 도출, 별도 문서
└── changelog.md    # 변경 이력
                    # (+ features.xlsx — "엑셀로 줘" 요청 시 옵션)
```
상위 `mnt/outputs/planning/`에는 **항상** `_nav.md`(앱 GNB·유틸 IA 앵커 — `_index.md`와 동급, 코퍼스 비의존)를 둔다. 그리고 코퍼스가 활성일 때만 `_index.md`(패키지 목록 1줄) + `_registry.md`(공유 엔티티·횡단 정책·용어·ID 색인) + `_features.csv`(**모든 패키지 features 통합 마스터**)를 둔다. Git 코퍼스는 `mnt/outputs/planning/.corpus/`에 fetch(캐시, 커밋 ✗). 각 .md 상단 frontmatter: `project / feature / version / status(draft|review|approved) / source(concept|figma|image|pdf|doc|mixed) / updated / owner`. 정본 레포 `cre8orclub/planning`의 `/docs/planning/{feature-name}/`로 옮긴다(스킬은 파일만 만들고 커밋·푸시는 사용자가 — 계정·레포 연결과 명령은 `git-output-rules.md §0·§4`).

## Reference Index

| 파일 | 내용 | 언제 |
|---|---|---|
| `references/input-normalization.md` | 입력 6유형(컨셉/이미지/피그마/PDF/문서/혼합) 판별·정규화 + 피그마 fallback + (요청 시) 레포 확인 | Phase 0 |
| `references/platform-context.md` | (요청 시) Git 최신 코퍼스 fetch·읽기 + 레지스트리 + 정합성 규칙 + 사후검토 체크리스트 | Phase 1·7 |
| `references/extraction-rules.md` | 3단계 추출 등급 · 화면 읽기 체크리스트 · 동사/명사 변환 · 행 단위 기준 · 예외 후보 (feature-inventory 흡수) | Phase 3 |
| `references/feature-column-spec.md` | 15컬럼 정의 · 행 단위 기준 · 시스템 동작 · 상태값 4종 · 맞물린 파트 · 근거 수준 · 근거/의도/제안/미결정 분리 | Phase 3·6 |
| `references/policy-exception-checklist.md` | 정책·예외·경계·상태 내부 점검표 + 레드플래그 R1~R18(출력 규칙) + 컴플라이언스 C1~C14 + 정책 디폴트 + 시스템 파급 (planning-review 흡수) | Phase 4 |
| `references/prd-template.md` | 기본 12섹션 PRD 템플릿(+ 선택 섹션·F↔FR) | Phase 5 |
| `references/decision-log-template.md` | decisions.md **답안지형** 템플릿(객관식 결정 슬롯 + ➕ 추가 빈칸 + ✅ 결정됨 이력) = 갱신 입력 surface · 관련 ID · 못감 3종 · 플랫폼 정합성 | Phase 6 · Update Mode |
| `references/git-output-rules.md` | 정본 레포(cre8orclub/planning 고정)·신원 규칙(§0) · 패키지 구조 · frontmatter · 커밋/푸시/PR(§4) · 교차참조·_registry · 금지어 | Phase 6·7 |
| `references/git-personalize.md` | **개인 로그인 개별화**(팀 공유 — 계정·이메일만 사용자별) · 고정값 vs 개인값 · 첫 실행 흐름 · 머신 설정 · preflight | Phase 0·6·7 |
| `references/update-cycle.md` | **업데이트 루프**(첫 생성 이후 갱신) · 입력 surface=decisions.md · 결정/추가/수정/범위/새입력 반영 규칙 · ID 연속·changelog·버전/상태 | Update Mode |
| `references/examples.md` | 입력 4유형별(컨셉/이미지/피그마/PDF) 워크드 예제 + 7파일 산출 | 학습·포맷 참조 |
| `scripts/personalize.py` | 개인 로그인 설정 get/detect/set (`.ghost-protocol.local.json`, gitignore — 팀 공유 제외) | Phase 0·6·7 |
| `scripts/corpus_tools.py` | (요청 시) Git 코퍼스 fetch(gh) + scan(registry.json·_registry.md·_features.csv 통합 마스터) | Phase 1·6 |
| `scripts/build_feature_csv.py` | features JSON → features.csv (15컬럼, UTF-8 BOM) | Phase 6 |
| `scripts/build_index.py` | 패키지 폴더 스캔 → README.md + 상위 _index.md 갱신 | Phase 6 |
| `scripts/validate_planning_package.py` | 7파일·12섹션·15컬럼·상태값·F↔FR·미결정 연결·금지어(qa.md 포함) + (--corpus) 끊긴 교차참조 검증 | Phase 7 |
| `references/qa-template.md` | qa.md 템플릿 — 8차원(기능·정책·상태/예외·디자인 완성도·UX/모션·접근성·성능·기능간 정합) · FR↔QA 매핑 · 미확정은 decisions I-n 연결(보류) | Phase 6 · "QA" |
| `scripts/track_open.py` | 열린 미결(D/I/C)을 📅 올린 날짜 기준 경과일로 추적 — 3일+ 묵힌 것 환기(읽기 전용, 코퍼스 시 형제 패키지까지) | Phase 0·7 · Update Mode |
| `references/nav-anchor.md` | **IA 앵커(`_nav.md`)** — 앱 GNB·유틸 메뉴 = 모든 패키지의 대전제. 스키마·`메뉴`=GNB 키 규약·점검·첫 실행·하이브리드 리마인드 | Phase 0 · "GNB" |
| `scripts/nav_check.py` | `_nav.md` 점검 — 월초(달 바뀜)·메뉴 변경 감지 환기(읽기 전용, track_open 결정성 재사용) | Phase 0 · 세션 시작 |
| `scripts/build_xlsx.py` | features JSON → 색상코딩 엑셀(상태·근거 수준) — "엑셀로 줘" 옵션 | Phase 6(옵션) |

## Next Phase

- "가는중" 기능의 정책·예외·상태를 더 깊이 → `planning-review` (이 스킬이 흡수했으나 단일 기능 정밀 설계가 필요하면)
- 패키지를 대상 레포에 커밋·동기화 → `git-sync`
- "디자인 필요" 화면 시각화 → `ui-designer` / `apple-canvas`
- 기능 → 로드맵·액션 분해 → `ceo-pipeline`
- 외부 제출용 문서화 → `shaper-skill` → `submission-cleanup`

**컴플라이언스·IP 점검이 띄운 미결정의 전문 처리(생성은 이쪽):**
- 약관·개인정보·청소년·결제·다크패턴·앱스토어 고지 → `app-and-jang`
- UGC 저작권 귀속·크리에이터 IP·크롤링/AI학습·OSS → `ip-skill`
- 파트너·크리에이터·B2B 계약 → `contract-consulting`

## Failure Modes (Gotchas)

- **반쪽 산출 함정**: PRD만 또는 기능 리스트만 만들면 핸드오프 실패. 항상 둘 다 + ID 연결(절대규칙 1·6).
- **오버스펙 함정**: PRD가 DB 테이블·API 엔드포인트·픽셀 UI까지 지시해 구현 지시서로 변질. 개발 착수 기준에 머물고 데이터·시스템 응답은 후보 수준으로만(절대규칙 2).
- **검증 엔진 욕심 함정**: 모든 기능을 위험 검토 문서처럼 길게 뽑음. 정책·예외 점검표는 내부 도구일 뿐, 산출물엔 필요한 항목만 쉬운 말로(절대규칙 2, `policy-exception-checklist.md`).
- **요소 나열 함정**: 버튼·아이콘·텍스트를 그대로 행으로 만듦. 행 1개 = 행동 또는 시스템 동작 1개(절대규칙 3).
- **멈춤 함정**: 입력이 부족하다고 "정보 더 달라"며 멈추면 안 된다. 컨셉 한 줄이라도 실행하고 미결정으로 남긴다.
- **근거 혼동 함정**: 화면에서 본 것과 AI 추정을 섞으면 개발자가 확정 기획으로 오해. 근거·의도·제안·미결정 분리(절대규칙 4).
- **정책 창작 함정**: 화면·문서에 없는 한도·과금·자격을 그럴듯하게 지어냄. 미정/제안으로(절대규칙 5).
- **R번호 누출 함정**: 레드플래그 R번호·4축 같은 엔진어가 PRD 본문에 남음. "꼭 정해야 할 것"으로 번역, 한 기능당 1~3개만(`policy-exception-checklist.md`).
- **미결정 흩뿌림 함정**: 미정을 PRD 곳곳에 단정처럼 박음. §12와 decisions.md에 모으고 관련 ID를 붙인다(절대규칙 7).
- **코퍼스 강제 함정**: 요청도 없는데 매번 GitHub 레포를 묻거나 코퍼스를 읽으려 해 실행이 무거워짐. 코퍼스는 요청 시 활성화하는 선택 레이어(절대규칙 8).

## ❌ WRONG vs ✅ CORRECT

```
❌ WRONG: PRD만 예쁘게 만들고 기능정의 리스트는 생략 → 개발자가 무엇을 만들지 표가 없음. 반쪽.
❌ WRONG: 작은 기능에도 16섹션·위험 검토·시스템 파급을 다 박음 → 기획서가 아니라 검사표.
❌ WRONG: PRD에 "users 테이블에 image_url 컬럼 추가, POST /profile/image" 같은 구현을 지시 → 구현 지시서로 변질.
❌ WRONG: 컨셉만 받았다고 "화면이 없어 진행 불가"로 멈춤 → 초안조차 안 나옴.
❌ WRONG: 화면에 안 보이는 "1인 1회·중복 차단" 정책을 확정해서 PRD에 박음 → 창작을 확정으로 오해.
❌ WRONG: 요청도 없는데 매번 GitHub 레포를 물어 코퍼스를 강제 로드 → 실행이 무거워짐.
✅ CORRECT: 입력 판별 → (요청 시 코퍼스 로드) → 의미 모델 → 기능 후보(행 1개=행동/시스템 동작 1개, 근거/의도/제안/미결정 분리) → 정책·예외·상태 내부 점검(레드플래그는 쉬운 말 1~3개) → 기본 12섹션 PRD(+ 필요 시 선택 섹션) + features(15컬럼) 동시 산출 → F↔FR 연결 → 미결정은 관련 ID·📅 올린 날짜 붙여 decisions.md → qa.md(기능 합격+디자인·UX/모션) → 7파일 Git 패키지 + 커밋 안내.
```

---
> Source: [jasonnamii/ghost-protocol](https://github.com/jasonnamii/ghost-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
