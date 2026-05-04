## autoimprove-cc

> > SKILL.md 자동개선 시스템 (Claude Code 네이티브).

# CLAUDE.md — autoimprove-cc

> SKILL.md 자동개선 시스템 (Claude Code 네이티브).
> 이 파일을 읽고 작업 시작할 것.

## 프로젝트 개요

**autoimprove-cc**는 Karpathy autoresearch 루프를 SKILL.md에 적용하는 Claude Code 네이티브 시스템.
Python 스크립트 없이 **agents + commands**만으로 동작한다.

- `/autoimprove <skill-path>` 로 실행
- `skill-optimizer` 에이전트가 루프를 자율 수행
- eval.json의 binary assertions로 채점
- 개선되면 git commit, 아니면 git reset

**GitHub:** https://github.com/VoidLight00/autoimprove-cc (Public)

---

## 파일 구조

```
autoimprove-cc/
├── CLAUDE.md                     이 파일
├── README.md
├── .claude/
│   ├── agents/
│   │   └── skill-optimizer.md    루프 에이전트 (핵심)
│   └── commands/
│       └── autoimprove.md        /autoimprove 슬래시 커맨드
├── eval/
│   ├── schema.json               eval.json 스키마
│   └── examples/                 eval.json 예시
├── skills/
│   └── example-skill/            테스트용 예시 스킬
│       ├── SKILL.md
│       └── eval/eval.json
└── LICENSE
```

---

## 사용법

```bash
# Claude Code 세션에서:

# 기본 실행 — eval.json 자동 생성 + 루프
/autoimprove skills/example-skill

# eval.json만 생성
/autoimprove skills/example-skill --gen-eval-only

# 최대 30회 반복
/autoimprove skills/example-skill --max-loops 30

# Dry-run (git 없이 채점만)
/autoimprove skills/example-skill --dry-run
```

---

## 핵심 컴포넌트

### skill-optimizer 에이전트

`.claude/agents/skill-optimizer.md` — 루프의 핵심.

1. eval.json 읽기 (없으면 자동 생성)
2. 각 assertion 참/거짓 판정
3. 점수 계산
4. 개선되면 git commit, 아니면 git reset
5. improve-log.md 업데이트
6. 만점 또는 max_loops까지 반복

### /autoimprove 커맨드

`.claude/commands/autoimprove.md` — 진입점.

경로 검증 → eval.json 확인 → git 상태 확인 → skill-optimizer 에이전트 실행.

---

## 코딩 규칙

- eval.json 스키마는 `eval/schema.json` 참조
- 에이전트는 한 번에 SKILL.md 변경 하나만
- 점수 동일 시 롤백 (개선 없으면 커밋 X)
- improve-log.md에 모든 변경 기록
- eval.json은 에이전트가 수정하지 않음 (테스트 기준 고정)

---

*Maintained by Hyeon · Powered by Kraken*

---
> Source: [VoidLight00/autoimprove-cc](https://github.com/VoidLight00/autoimprove-cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
