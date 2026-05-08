## obsidian-zettelkasten-setup

> 이 프로젝트는 Zettelkasten + PARA 방식의 Obsidian Vault입니다.

# Obsidian Vault Rules

이 프로젝트는 Zettelkasten + PARA 방식의 Obsidian Vault입니다.
핵심 원칙: 연결이 분류보다 중요하다. 분류에 고민하느라 기록을 미루지 않는다.

## 폴더 구조

- `00_Inbox/` — Fleeting Notes. 떠오르는 생각을 부담 없이 던져놓는 곳. 버려져도 OK.
- `01_Notes/01_Literature/` — Literature Notes. 책, 아티클, 영상 등 외부 지식의 원본 노트.
- `01_Notes/02_Permanent/` — Permanent Notes. 하나의 개념을 담은 맥락 독립적 지식 노트.
- `02_PARA/01_Projects/` — 마감일과 달성 목표가 있는 작업.
- `02_PARA/02_Areas/` — 마감일 없이 지속적으로 유지/관리하는 영역. 개인적이고 맥락 의존적인 설정/스펙.
- `02_PARA/03_Resources/` — 특정 프로젝트에 종속된 참고 자료.
- `02_PARA/04_Archives/` — 완료되거나 더 이상 활성 상태가 아닌 프로젝트/자료의 보관소.
- `03_Tracking/` — Daily Notes, 주간 회고, 습관 추적 등 시계열 데이터.
- `04_Atlas/` — MOC(Map of Content). 노트가 자연스럽게 쌓여 필요해질 때 만드는 지도.

## 메모의 세 가지 유형

- **Fleeting Note** (`00_Inbox`): 빠르게 포착한 일시적 생각. 버려져도 됨.
- **Literature Note** (`01_Literature`): 외부 자료를 읽으며 원문 발췌 + 내 반응/의문을 함께 기록.
- **Permanent Note** (`02_Permanent`): 내 언어로 재구성한 맥락 독립적 지식. 반드시 기존 노트와 연결.

## Literature Note 작성 규칙

- 파일 위치: `01_Notes/01_Literature/제목.md`
- 원문 발췌와 내 생각을 함께 기록한다. 발췌만으로 끝내지 않고 내 반응이나 의문을 덧붙인다.
- YAML frontmatter 필수 필드:
  ```yaml
  ---
  title: "제목"
  type: book/video/article/insight
  author: (선택)
  start_read_date: YYYY-MM-DD
  finish_read_date: YYYY-MM-DD
  status: done
  tags:
    - reading
    - (카테고리 태그)
  ---
  ```

## Permanent Note 작성 규칙

- 파일 위치: `01_Notes/02_Permanent/제목.md`
- 하나의 노트에는 하나의 개념만 담는다.
- 발췌문이 아닌 내 언어로 재구성한 내용만 작성한다.
- 맥락 독립적 지식이 들어간다. 단, 노트는 새로운 연결을 통해 성장할 수 있다.
- 반드시 기존의 다른 노트에 링크를 건다. 고립된 노트는 잊혀진다.
- YAML frontmatter 필수 필드:
  ```yaml
  ---
  title: "제목"
  source: "[[출처 Literature 노트]]" 또는 "출처 URL"
  page: "p00" (선택)
  tags:
    - permanent
    - (카테고리 태그)
  ---
  ```

## 워크플로우

1. 캡처: 생각이 나면 `00_Inbox`에 부담 없이 기록.
2. 발전: "이 생각을 더 발전시키고 싶은가?" → Yes면 Literature/Permanent로.
3. 연결: Permanent 노트 생성 시 기존 노트와 링크. 이 단계가 가장 중요.
4. 글쓰기: 연결된 노트들에서 패턴을 발견하면 글로 발전시킨다.

## Daily Note 업데이트 규칙

- 파일 위치: `03_Tracking/daily/YYYY/MM-Month/YYYY-MM-DD-Day.md`
- 사용자가 작업을 완료한 후 "Daily Note 업데이트" 요청 시, 오늘 날짜의 Daily Note에 작업 내용을 반영한다.
- 업데이트 대상 섹션:
  - **3. 하루 계획 > 중요 작업**: 수행한 작업을 체크리스트로 기록. 완료된 항목은 `[x]`, 미완료는 `[ ]`.
  - **4. 회고 및 로그**: 학습한 것, 잘한 것, 아쉬운 것을 간결하게 기록.
  - **5. Rolling Review**: `weekly::` 에 한 줄 요약 추가.
- 기존에 작성된 내용을 덮어쓰지 않고, 새 항목을 추가하는 방식으로 기록한다.
- YAML frontmatter와 Dataview 쿼리 블록은 수정하지 않는다.

## Project Note 작성 규칙

- 파일 위치: `02_PARA/01_Projects/프로젝트명.md`
- 템플릿: `settings/template/project.md` — 생성/업데이트 시 반드시 이 템플릿의 구조(frontmatter, 섹션 순서, callout)를 따른다.
- 프로젝트 고유 섹션(개요, 아키텍처, 기술 스택 등)은 템플릿 섹션 앞에 자유롭게 추가 가능.
- 프로젝트 완료 시:
  1. 먼저 사용자에게 회고(잘한 점, 아쉬운 점, 배운 것)를 물어본다.
  2. 응답을 프로젝트 노트의 `7. 회고` 섹션에 기록한다.
  3. `status: done`으로 변경한 뒤 `02_PARA/04_Archives/`로 이동한다.

## Resource Note 작성 규칙

- 파일 위치: `02_PARA/03_Resources/제목.md`
- 특정 프로젝트에 종속된 참고 자료. 프로젝트가 끝나면 의미가 줄어드는 정보.
- "내가 이 프로젝트를 그만둬도 남는 지식인가?" → No면 Resource.
- YAML frontmatter 필수 필드:
  ```yaml
  ---
  title: "제목"
  project: "[[연결된 프로젝트]]"
  source: "출처 URL 또는 [[출처 노트]]"
  tags:
    - resource
    - (프로젝트 태그)
  ---
  ```

## Permanent vs Area 구분 기준

- "내가 이 일을 그만둬도 남는 지식인가?" → Yes: Permanent / No: Area
- 고민이 길어지면 일단 적고 나중에 옮긴다. 분류보다 기록이 먼저.

## 노트 생성 시 기본 규칙

- Claude가 새 노트를 생성할 때 기본 위치는 `00_Inbox/`이다.
- 사용자가 특정 폴더를 지정하면 해당 위치에 생성한다.
- 노트 유형이 명확한 경우(Literature 등)에는 해당 폴더에 직접 생성한다.

## 노트 편집 시 주의사항

- 기존 노트의 YAML frontmatter 형식을 유지한다.
- 태그는 list 형식(하이픈)으로 작성한다.
- Obsidian 위키링크 형식 `[[노트명]]`을 사용한다.
- MOC는 미리 계획하지 않고, 노트가 자연스럽게 쌓여 필요해질 때 `04_Atlas/`에 생성한다.

---
> Source: [zeroam/obsidian-zettelkasten-setup](https://github.com/zeroam/obsidian-zettelkasten-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
