## llm-wiki

> > `CLAUDE.md` § 도메인 프레임워크의 상세 정본. LLM은 페이지를 쓰거나 갱신하거나 위키에 질문할 때 이 규약을 따른다. 규약을 바꾸면 여기서 바꾸고 `_meta/changelog.md`에 기록한다.

# 위키 페이지·인덱스·라우팅 규약 (정본)

> `CLAUDE.md` § 도메인 프레임워크의 상세 정본. LLM은 페이지를 쓰거나 갱신하거나 위키에 질문할 때 이 규약을 따른다. 규약을 바꾸면 여기서 바꾸고 `_meta/changelog.md`에 기록한다.

---

## 0. 핵심 모델 — 라우터·샤딩·정본화 (먼저 읽기)

LLM Wiki의 검색은 "전부 읽기"가 아니라 **라우팅**이다. 위키가 수천 페이지로 커져도 **질문 1개당 읽는 토큰이 거의 일정**하게 유지되는 게 목표다. 이를 위한 4개 기둥:

1. **index = 라우터(MOC)**, 카탈로그가 아니다. `index.md`는 "모든 페이지 목록"이 아니라 **"질문 의도 → 어느 타입/샤드를 펼칠지"** 정하는 얇은 진입점이다. 엔티티 줄은 index에 두지 않는다.
2. **타입별 하위 인덱스 + 샤딩.** 실제 엔티티 카탈로그는 `indexes/{type}.md`에 둔다. 한 타입이 커지면 **정본명 첫 글자로 ≤50K 토큰 샤드**로 쪼갠다 (§8).
3. **정본화(aliases).** 표기 흔들림(Parasite/기생충, 샬라메/Timothée Chalamet)을 **정본명 1개**로 모은다. 정본명이 곧 **샤드 라우팅 키**다 (§4).
4. **2단 라우팅 (Phase A / Phase B).** 질문 처리는 "어디를 열지 정하기(Route, 샤드 안 읽음)"와 "지정 샤드·페이지 읽기(Search)"를 분리한다 (§9). 라우팅이 샤드를 읽지 않아야 샤딩의 토큰 절감이 지켜진다.

> 규모가 작을 땐 이 구조가 과해 보이지만, **메커니즘을 처음부터 박아둬야** 데이터가 쌓일 때 자연스럽게 라우터→타입 인덱스→샤드로 성장한다. 소규모에선 한 단계가 다음 단계를 겸한다 (§9 성장 경로).

---

## 1. 페이지 타입

| 타입 | frontmatter `type` | 위치 | 1:1 대상 |
|------|--------------------|------|----------|
| 소스 요약 | `source` | `30-wiki/{topic}/sources/` | raw 소스 1개당 1페이지 |
| 엔티티 | `entity` | `30-wiki/{topic}/entities/` | 인물·조직·장소·제품·작품 |
| 개념 | `concept` | `30-wiki/{topic}/concepts/` | 이론·방법론·용어 |
| 타입 인덱스 | `index` | `30-wiki/{topic}/indexes/{type}.md` | 타입 1개당 1개(+샤드) |
| 주제 라우터 | `index` | `30-wiki/{topic}/index.md` | 주제 1개당 1개 |
| 루트 라우터 | `index` | `30-wiki/index.md` | 위키 전체 1개 |
| 종합 개요 | `overview` | `30-wiki/{topic}/overview.md` | 주제 1개당 1개(선택) |
| 정본 사전 | `aliases` | `30-wiki/{topic}/aliases.md` | 주제 1개당 1개 |
| query 파일백 | `query` | `50-queries/` | 가치 있는 질의 결과 |

**도메인 세분(선택):** `entity`가 많아지면 도메인에 맞게 하위 타입으로 나눌 수 있다 (예: 영화 도메인 → `people`/`works`/`series`). 이때 폴더·인덱스도 타입별로 분리한다. 기본은 `entity` 하나로 시작하고, 한 종류가 수십 개를 넘으면 분리를 검토한다.

**신뢰 등급(tier):** 모든 엔티티/개념 페이지는 `tier`를 단다 — `reviewed`(원본 추출·사람 확인) 또는 `auto`(웹/추론 lazy 생성, 미검수). `auto`는 `auto-generated.md` 대장에 등록한다 (§10).

---

## 2. frontmatter 스펙 (타입별)

### source
```yaml
type: source
title: "소스 원제목"
source_file: 20-raw/2026-06-14-article.md   # 처리완료 원본 역참조 (compile이 raw로 이동 후 경로)
topic: "주제 슬러그"
summary: "1~2문장 + 검색 키워드 5~10개"   # 이 문장이 sources/index 줄로 재사용됨
ingested: 2026-06-14
author: ""        # 있으면
url: ""           # 있으면
tags: []
provenance: extracted
```

### entity / concept
```yaml
type: entity            # 또는 concept
canonical: "정본명"      # 라우팅 키 (첫 글자가 샤드 결정)
aka: []                 # 같은 대상의 다른 표기 (aliases.md에도 등재)
topic: "주제 슬러그"
summary: "1~2문장 + 키워드 5~10개"   # ★ 이 문장이 indexes/{type}.md 의 줄 description으로 그대로 재사용됨
tags: []
sources: []             # 이 페이지를 뒷받침하는 raw/소스 id들
tier: reviewed          # reviewed | auto
provenance: extracted   # extracted | inferred | ambiguous | web-enriched
status: active          # active | stub | deprecated
updated: 2026-06-14
```

`summary`는 **인덱스의 원천**이다 — 잘 쓰면 인덱스가 자동으로 좋아진다. 1~2문장 정의 + 검색 키워드를 반드시 포함한다.

---

## 3. 페이지 본문 골격 (고정 섹션)

ingest/compile 때 "어디에 쓸지"를 결정적으로 만들기 위해 타입별 섹션을 고정한다. 빈 섹션은 `_(아직 없음)_`으로 남겨 grep 가능하게 둔다.

- **source**: `**TL;DR:**` → `## Key claims` → `## Entities & concepts` → `## How this updated the wiki` → `## Notable quotes`
- **entity/concept**: `**정의:**`(BLUF) → `## 요약` → `## Key facts` → `## 관계` → `## Open questions / 모순` → `## Sources`

템플릿 실물은 `40-templates/{source,entity,concept}.md`.

---

## 4. 네이밍 & 정본화 (aliases.md)

- 파일명 = **kebab-case 슬러그**, 안정적(한 번 정하면 안 바꿈 — 링크 깨짐 방지). 예: `napoleon-bonaparte.md`
- 한글 엔티티는 한글 슬러그 허용(공백→하이픈). 예: `기동전.md`
- 소스 슬러그는 날짜 prefix 권장: `2026-06-14-article-title.md`

### 정본명 규칙 (라우팅 키)
- 엔티티마다 **정본명 1개**를 정하고 frontmatter `canonical`에 둔다. **정본명 첫 글자가 샤드를 결정**한다 (§8).
- 같은 대상의 다른 표기는 **새 파일을 만들지 말고** frontmatter `aka` + 중앙 `aliases.md`에 등재한다.
- **정본화 = 검색 입구.** 질문이 "Parasite"로 와도 `aliases.md`에서 `기생충`으로 바꾼 뒤 라우팅한다.

### aliases.md (주제별 정본 사전)
`30-wiki/{topic}/aliases.md`에 `별칭/표기 → 정본명` 매핑을 누적한다.
```markdown
| 표기/별칭 | 정본명 | 타입 |
|-----------|--------|------|
| Parasite, 기생충, 寄生虫 | 기생충 | entity(work) |
| 샬라메, Chalamet | Timothée Chalamet | entity(person) |
```
compile이 새 엔티티를 만들 때 별칭이 보이면 여기에 추가한다. query는 라우팅 전에 이 표를 먼저 읽는다.

---

## 5. 링크 & 교차참조

- 위키 내부 참조는 `[[상대경로/슬러그]]`. 예: `[[entities/napoleon-bonaparte]]`, `[[sources/2026-06-14-article]]`
- 모든 **사실 주장**은 뒤에 provenance 링크: `나폴레옹은 코르시카 출신이다 [[sources/2026-06-14-article]]`
- 출처 없는 추론·합성은 `(추론)`/`확인 필요`로 표시.
- **링크는 대상 페이지가 실제 있을 때만** 건다. 없는 엔티티는 plain text로 두고(씨앗), 질문 시 lazy 승격한다 (§10) — 죽은 링크를 양산하지 않는다.

### 동명 disambiguation (경로 명시 링크)
같은 이름이 여러 타입/폴더에 있으면(예: `entities/Dune`이 작품 페이지와 시리즈 페이지 양쪽) **경로 명시 링크**로 가린다:
```markdown
[[works/Dune|Dune]]  (단일 작품)  vs  [[series/Dune|Dune]]  (프랜차이즈)
```
그리고 **라우터(index)의 「동명 충돌」 노트에 그 이름을 등재**해, 라우팅 시 양쪽 샤드를 모두 펼치게 한다.

---

## 6. 모순 처리

새 소스가 기존 주장과 충돌하면 **덮어쓰지 말고** 양쪽을 보존하고 명시:
```markdown
## Open questions / 모순
> ⚠️ Contradiction: [[sources/A]]는 X라 하고 [[sources/B]]는 Y라 한다. 미해결.
```
`/lint`가 `grep -rn "⚠️ Contradiction" 30-wiki/`로 전수 추적한다. (동명 disambiguation은 모순이 아니라 §5의 경로 링크로 처리 — 구분할 것.)

---

## 7. index.md = 라우터 (MOC, 카탈로그 아님)

**핵심: index는 "읽는 목록"이 아니라 "어디로 갈지 정하는 라우터"다.** 엔티티 줄은 index에 두지 않는다 — 그건 `indexes/{type}.md`의 일.

### 루트 라우터 (`30-wiki/index.md`)
- frontmatter `tags: [index, moc, router]`.
- 주제 목록 + 주제별 페이지 수 + 전역 허브(overview·aliases) 링크만.
- "질문이 어느 주제인지" → 해당 주제 라우터로 보낸다.

### 주제 라우터 (`30-wiki/{topic}/index.md`)
- **의도 → 타입 인덱스 라우팅 표**가 본체:

| 의도 | 읽을 하위 인덱스 | 분할 키 | 개수 |
|------|------------------|---------|------|
| 인물·조직 | `[[indexes/entities]]` (또는 샤드) | 정본명 첫 글자 | N |
| 개념·이론 | `[[indexes/concepts]]` | (단일) | M |
| 원본 출처 | `[[sources/index]]` | (단일) | K |

- **동명 충돌 노트** (양쪽 샤드 확인 대상 이름 목록).
- **저장된 쿼리** 링크 (`50-queries/`).
- 푸터에 타입별 합계.

라우팅 규칙(라우터 본문에 명시): "타입+첫 글자가 명확하면 샤드 1개만 / 타입 모호하면 후보 샤드 동시 / 못 찾으면 grep → lazy".

---

## 8. 타입별 하위 인덱스 + 샤딩

`30-wiki/{topic}/indexes/{type}.md` = 그 타입 엔티티의 **카탈로그**. 각 줄:
```markdown
- [[entities/napoleon-bonaparte]] — 프랑스 군인·황제. 기동전·대육군. 키워드: 코르시카, 1804 대관식, 워털루.
```
줄 description은 **페이지 frontmatter `summary`를 그대로 재사용**한다(§2). 이 줄로 "어느 페이지를 펼칠지" 1차 선별한다.

### 샤딩 (≤50K 토큰)
- 한 타입 인덱스가 커지면 **정본명 첫 글자로 토큰 균형 샤딩**: `indexes/entities-a-m.md`, `indexes/entities-n-z.md`.
- **한글 정본은 둘째 샤드**(`-n-z` 쪽)에 모은다(알파벳 뒤).
- 각 샤드 ≤50K 토큰 유지. 넘으면 경계를 다시 나눈다.
- 라우터의 분할 키 표를 샤드와 동기화한다(`/lint`가 점검).

---

## 9. 검색 성장 경로 (3단) + Phase A/B 라우팅

| 규모 | 인덱스 구조 | 라우팅 |
|------|-------------|--------|
| ~수십 페이지 | 주제 라우터가 곧 카탈로그 겸함 (`indexes/` 생략 가능) | 라우터 1개 읽고 페이지 직행 |
| ~수백 페이지 | 타입별 `indexes/{type}.md` 분리 | 라우터 → 타입 인덱스 1개 → 페이지 |
| 수천+ 페이지 | 타입 인덱스를 첫 글자 **샤딩** | 라우터 → 샤드 1개 → 페이지 (+선택: 외부 검색 `.rag`) |

> 외부 하이브리드 검색(`.rag` BM25/벡터)은 **선택적 파생 인프라**다. 있으면 query의 후보 찾기 1순위, **없으면 라우터→샤드 tiered-read로 폴백**한다 (OMC·외부 의존 0이 기본). 원본·위키는 불변이고 `.rag`는 언제든 재생성 가능한 파생물.

### 2단 라우팅 (모든 규모 공통)
- **Phase A — Route (페이지 안 읽음):** 질문에서 엔티티+타입 의도+연산(조회/비교/종합)을 뽑고, `aliases.md`로 정본화한 뒤, **라우터만 보고** 열 샤드의 최소 집합을 정한다. 샤드는 절대 읽지 않는다 (~2K 토큰).
- **Phase B — Search (샤드+페이지 읽기):** 지정 샤드만 펼쳐 후보 선별 → 본문만 읽고 `[[링크]]` 1홉 보강 → 근거로만 인용 답. 미스 시 **넓히기 사다리**: 형제 샤드 → `30-wiki/` 전체 grep → lazy 생성 또는 "없음".

**핵심 불변식:** index/샤드를 통째로 컨텍스트에 올리지 않는다. 항상 라우팅으로 **소수 후보만** 들인다 → 위키 크기와 query당 토큰 비용을 분리.

---

## 10. lazy 생성 + tier 승격

- **고빈도만 미리 페이지로.** 코퍼스 전역에서 **≥2회** 등장한 엔티티만 자체 페이지를 만든다(compile이 전역 집계로 판정). 1회 등장은 상위 페이지의 plain text **씨앗**으로 두고, 질문이 오면 그때 생성(lazy).
- **lazy 생성:** Phase B에서 grep에도 없을 때 — `20-raw/`·`sources/`에 **씨앗이 있으면** raw+웹으로 즉석 생성(`tier: auto`, `provenance: web-enriched`) → `auto-generated.md` 대장 + 해당 첫 글자 샤드에 등재(라우터는 개수·동명 노트만 갱신). **씨앗도 없으면 지어내지 말고 "위키에 없음 — ingest 필요".**
- **tier 승격:** `auto` 페이지를 사람이 확인하면 `reviewed`로 승격하고, frontmatter·`auto-generated.md` 대장·타입 인덱스 세 곳을 정합시킨다(`/lint` 점검).

---

## 11. overview.md (종합 진입)

`30-wiki/{topic}/overview.md` = 전 소스를 가로지른 큰 그림. "이 위키가 무엇을 아는가"를 한 페이지로. 거시·탐색 질문(Phase B)은 overview를 먼저 읽고 관련 샤드로 내려간다. compile이 갱신한다.

---

## 12. 파일 변환·이미지·PDF

### 문서 변환 (바이너리 → 마크다운)
Claude는 docx·pptx·xlsx 같은 바이너리를 직접 못 읽는다. `/ingest`가 확장자를 보고 마크다운으로 변환한 뒤 inbox에 넣는다. **원본 바이너리는 `20-raw/assets/`에 보관**(출처 보존), 변환된 `.md`만 compile 대상.

| 입력 | 1순위 (로컬·무료) | 폴백 (opt-in) |
|------|-------------------|----------------|
| `.md`/`.txt`/`.html` | 그대로 | — |
| `.pdf` | Claude PDF Read(텍스트형) / `markitdown` | LlamaParse (스캔·복잡 표) |
| `.docx`/`.pptx`/`.xlsx` | `markitdown <파일>` | LlamaParse (표 많은 문서) |

- **markitdown = 주력.** `pip install 'markitdown[all]'` 하나로 Office·PDF·이미지를 마크다운으로. 로컬·무료 → 자족 원칙 유지.
- **LlamaParse = 순수 opt-in.** `LLAMA_CLOUD_API_KEY`가 있을 때만 발동(표·레이아웃 복잡 문서 품질↑). 무료 크레딧 한도 내 사용, 초과 시 유료. **키 없으면 조용히 로컬(markitdown)로 폴백** — 자족성 안 깨짐.
- 어떤 도구도 없으면 막지 말고 설치 안내 또는 "텍스트로 붙여달라".

### 이미지·PDF
- 원본은 `20-raw/assets/`에 로컬 저장 (URL은 깨질 수 있으므로 다운로드 권장).
- 위키 페이지에서 이미지 참조: `![설명](../../20-raw/assets/figure.png)` + 캡션 텍스트.
- **2단계 읽기:** LLM은 마크다운 인라인 이미지를 한 번에 못 읽으므로, 텍스트를 먼저 읽고 필요한 이미지를 별도로 Read 한다.

---

## 13. provenance 등급 (설명가능성)

모든 주장·페이지에 출처 근거를 표기한다:
- `extracted` — 원본에서 직접 추출
- `inferred` — LLM이 추론·합성 (단정 금지, 사람 확인 대상)
- `ambiguous` — 정본·사실이 모호 (pending-decisions로 추적)
- `web-enriched` — lazy 생성 시 웹 보강 (`tier: auto`와 짝)

`inferred`/`ambiguous`는 단정하지 말고 §14 게이트로 사람 확인 후 확정한다.

---

## 14. 소크라테스 게이트 (인간 감독)

위키가 커질수록 어딘가 환각이 1개 섞일 확률이 1에 수렴한다. 그래서 LLM은 **무비판 수용자가 아니라 비판자**로 동작한다:
- 사람이 자료·아이디어를 던지면, 모순·약점·근거 부족을 **짚어 되묻는다**.
- compile/query의 `inferred`·`ambiguous`·모순은 단정하지 말고 사람 확인을 받은 뒤 `reviewed`로 확정한다.
- 대량 자동 생성(lazy/백필) 후엔 `/lint`로 dead-link·동명·할루·오역 스폿체크를 권한다.

---
> Source: [fivetaku/llm-wiki](https://github.com/fivetaku/llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
