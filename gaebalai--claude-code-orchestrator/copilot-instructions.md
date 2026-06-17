## claude-code-orchestrator

> **Claude Code**가 **Codex CLI(심층 추론)**와 **Gemini CLI(대규모 리서치)**를 오케스트레이션하여 각 에이전트의 강점을 극대화하고 **개발 속도와 품질을 동시에 끌어올리는 구조**다.

# Claude Code Orchestrator

**멀티 에이전트 협업 프레임워크**

**Claude Code**가 **Codex CLI(심층 추론)**와 **Gemini CLI(대규모 리서치)**를 오케스트레이션하여 각 에이전트의 강점을 극대화하고 **개발 속도와 품질을 동시에 끌어올리는 구조**다.

---

## 왜 이 구조가 필요한가?

| 에이전트 | 강점 | 사용 목적 |
|-------|----------|---------|
| **Claude Code** | 오케스트레이션, 사용자 대화 | 전체 통합, 태스크 관리, 의사결정|
| **Codex CLI** | 깊은 추론, 설계 판단, 디버깅 | 설계 검토, 에러 분석, 트레이드오프 평가 |
| **Gemini CLI** | 1M 토큰, 멀티모달, 웹 검색 | 대규모 코드 분석, 라이브러리 조사, PDF/영상 분석 |

**IMPORTANT**: 각 에이전트는 단독으로도 강력하지만, **의도적으로 역할을 분리했을 때 성능이 폭발**한다.

---

## 컨텍스트 관리 (CRITICAL)

Claude Code의 최대 컨텍스트는 **200k 토큰**이지만,
툴 정의 / 시스템 프롬프트 등을 제외하면 **실질적으로 70~100k 수준**이다.

**YOU MUST** 👉 그래서 **출력이 큰 작업은 반드시 서브 에이전트 경유**가 원칙이다.

### 출력 크기 기준

| 출력 크기  | 사용 방식           | 이유                     |
| ------ | --------------- | ---------------------- |
| 1~2문장  | 직접 호출           | 오버헤드 없음                |
| 10줄 이상 | **서브 에이전트 경유**  | 메인 컨텍스트 보호             |
| 분석 리포트 | 서브 에이전트 → 파일 저장 | `.claude/docs/`에 영구 보존 |

### 예시
```
# MUST: 서브 에이전트 경유 (출력 큼)
Task(subagent_type="general-purpose", prompt="Codex에게 설계 검토 요청 후 요약만 반환")

# OK: 직접 호출 (아주 짧은 출력)
Bash("codex exec ... '한 문장으로 답변'")
```

---

## 빠른 사용 가이드(Quick Reference)

### Codex CLI를 써야 할 때

- 설계 판단
    - “어떤 패턴이 맞을까?”
    - “이 구조, 확장 가능할까?”
- 디버깅
    - “왜 이 에러가 나는지?”
- 비교/선택
    - “A vs B, 뭐가 나은지?”
- ➡ 깊은 사고가 필요하면 Codex

→ 참고: `.claude/rules/codex-delegation.md`

### Gemini CLI를 써야 할 때

- 리서치
    - “이거 조사해줘”
    - “요즘 트렌드 뭐임?”
- 대규모 분석
    - “이 레포 전체 구조 설명해줘”
- 멀티모달
    - “이 PDF 요약”
    - “이 강의 영상 핵심만 정리”
- ➡ 많이 읽고, 넓게 볼 땐 Gemini

→ 참고: `.claude/rules/gemini-delegation.md`

---

## Workflow

```
/startproject <기능명>
```

### 진행 순서

1. Gemini 
    - 리포지토리 전체 분석 (서브 에이전트)
2. Claude 
    - 요구사항 정리
    - 개발 계획 수립
3. Codex 
    - 설계 리뷰 및 리스크 검토 (서브에이전트)
4. Claude 
    - 실행 가능한 태스크 리스트 생성
5. (권장)
    - **구현 완료 후 별도 세션에서 리뷰**

→ 관련 커맨드: `/startproject`, `/plan`, `/tdd` skills

---

## 기술 스택(Tech Stack)

- **Python** 
- **uv** 
    - pip 직접 사용 ❌
    - 속도 + 재현성 우선
- **ruff** 
    - lint/format 통합
- **ty** 
    - type check
- **pytest**
    - 테스트 표준
- 공통 명령어
    ```
    poe lint
    poe test
    poe all
    ```

→ 참고: `.claude/rules/dev-environment.md`

---

## 문서구조(Documentation)

| 위치                             | 내용                    |
| ------------------------------ | --------------------- |
| `.claude/rules/`               | 코딩 / 보안 / 언어 규칙       |
| `.claude/docs/DESIGN.md`       | 설계 결정 기록              |
| `.claude/docs/research/`       | Gemini 조사 결과          |
| `.claude/logs/cli-tools.jsonl` | Codex / Gemini 입출력 로그 |

---

## 언어 프로토콜(Language Protocol)

- **사고/코드/로그**: 영어
- **사용자대화/설명**: 한국어

---
> Source: [gaebalai/claude-code-orchestrator](https://github.com/gaebalai/claude-code-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
