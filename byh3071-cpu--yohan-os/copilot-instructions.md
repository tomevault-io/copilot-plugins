## yohan-os

> - 이 프로젝트는 "Yohan OS" — MCP 기반 개인 지식/맥락 서버 (SoT 백엔드)

# ============================================================
# Yohan OS — .cursorrules (프로젝트 헌법)
# ============================================================

## 1. 프로젝트 정체성
- 이 프로젝트는 "Yohan OS" — MCP 기반 개인 지식/맥락 서버 (SoT 백엔드)
- 스택: Node 20+, TypeScript, MCP stdio, Zod 4, YAML SoT
- 사용자: Yohan (timezone: Asia/Seoul, lang: ko)

## 2. SoT(Source of Truth) 원칙
- SoT = `memory/` + Git. 이것이 최종 진실.
- 노션은 미러·입력 큐일 뿐, SoT를 덮지 않는다.
- 동기 규칙: `memory/rules/notion-sync.md` 참조.
- 충돌 시 항상 `memory/` 파일이 우선.

## 3. 문서 작성 규칙
- 모든 `.md` 파일은 **YAML 프론트매터 필수**.
- 구분선: 블록 **시작·끝** 모두 **`---`** (하이픈 **3개**). `--` 는 무효.
- `tags`·`related`: **YAML 배열** — 예: `tags: [a, b, c]`, `related: [other-id]`.
- 본문: **첫 줄은 `#` 레벨1 제목** → 이후 `##` + 불릿(`-`). 산문 최소화.
- 한 파일 = 한 주제 (원자적 노트). 날짜: `YYYY-MM-DD`.

복사용 예시:

```yaml
---
id: kebab-case-id
date: YYYY-MM-DD
domain: 도메인명
tags: [tag-one, tag-two]
related: [other-note-id]
status: draft
---
```
(`status`: `draft` | `insight` | `active` | `archived`)

## 4. 코딩 규칙
- 들여쓰기: spaces 2
- 주석: 간결하게, 불필요한 주석 금지
- 변수·함수명: camelCase (TypeScript 컨벤션)
- 에러 처리: 임의 수정 금지 → 원인과 해결 방안을 먼저 제시

## 5. 절대 금지 (must_not)
- 시크릿·API 키를 SoT 평문에 저장 금지 (.env 사용)
- 확인 없이 프로덕 배포·삭제·대금 처리 금지
- 출처 없는 "최신" 단정 금지
- 정확도 100% 무분별 약속 금지
- `memory/` 구조(폴더·파일명)를 임의로 변경 금지

## 6. 작업 흐름
- 세션 시작 시: MCP `get_context` 호출하여 맥락 로드
- 복잡 작업: P/G/E (Planner → Generator → Evaluator) 파이프라인 적용
- Evaluator pass 전까지 SoT에 영구 반영 금지 (draft만 허용)
- 아키텍처·비전 충돌 결정은 `append_decision`으로 로그기록

## 7. 참조 우선순위
- 비전·요구: `docs/VISION-AND-REQUIREMENTS.md`
- 하네스 규칙: `memory/rules/agent-harness.md`
- 평가 체크리스트: `memory/rules/evaluator-checklist.md`
- GitHub 레포 정리(URL + why-how 인사이트): `memory/rules/github-repo-why-how.md`, 템플릿 `memory/templates/github-repo-why-how-template.md`
- 선별·맥락·파인만 2차 정제(URL/OCR/RSS 등): `memory/rules/archiving-appraisal-feynman.md`
- 프로필·금지: `memory/profile.yaml`
- 활성 프로젝트: `memory/active-project.yaml`

## 8. 컨텍스트 주입 규칙
- 한 번에 모든 문서 주입 금지 → Attention 분산·할루시네이션 유발
- 해당 작업에 필요한 최소 맥락만 `@파일`/`@폴더`로 주입
- 문서가 방대할 경우 AGENTS.md(목차) 먼저 읽고 세부 문서 탐색

## 9. 커뮤니케이션
- 언어: 한국어 기본, 코드·기술 용어는 영어 혼용
- 스타일: 결론·액션 우선, 미사여구 금지, 팩트 기반
- 불확실한 정보: 추측하지 않고 명시적으로 "확인 필요" 표기

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/byh3071-cpu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
