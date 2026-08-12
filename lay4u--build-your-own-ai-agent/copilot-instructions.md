## build-your-own-ai-agent

> GitHub Star 극대화를 위한 "Build Your Own AI Agent" 프로젝트.

# build-your-own-ai-agent — CLAUDE.md

## Project Overview

GitHub Star 극대화를 위한 "Build Your Own AI Agent" 프로젝트.
12개의 독립적인 AI 에이전트를 처음부터 만드는 실습 가이드.

PRD: `.omc/plans/prd-gitstar-max.md`

## Absolute Rules

### Rule 1: No Direct Code Writing

Claude는 `.py`, `.js`, `.ts`, `.html`, `.css`, `.sh` 파일을 직접 작성/수정하지 않는다.
모든 코드 구현은 반드시 omc-teams(codex/gemini)에 위임한다.

**Claude가 직접 작성 가능한 파일:**
- `CLAUDE.md`, `AGENTS.md`
- `.omc/**` (상태, 계획, 메모)
- `.github/workflows/*.yml` (CI 설정)
- `experiments.md`, `progress.md` (기록)
- git 커밋 메시지

**위반 시:** 즉시 중단하고 omc-teams로 재위임.

### Rule 2: omc-teams Delegation Pattern

```
Claude 역할:
1. 태스크 설계 (어떤 프로젝트를, 어떤 스펙으로)
2. omc-teams 호출 (codex/gemini에 병렬 위임)
3. 결과 대기 (omc_run_team_wait)
4. 결과 검증 (파일 존재 + 실행 테스트)
5. progress.md 기록 + git 커밋
6. 다음 iteration 설계 → 1번 반복
```

### Rule 3: Worker Role Assignment

| Worker | 역할 | 담당 |
|--------|------|------|
| **codex** | Python 구현 | `main.py`, `*.py`, `requirements.txt` 작성 |
| **gemini** | 문서/에셋 | `README.md`, 아키텍처 설명, 다국어 번역 |
| **claude** | 검증/통합 | 코드 리뷰, 실행 테스트, 품질 체크 |

### Rule 4: Parallel Track System

매 iteration에서 최대 효율로 병렬 실행:

```
Track A (codex x2): 프로젝트 코드 구현 (2개 동시)
Track B (gemini x1): README/문서 작성
Track C (claude x1): 이전 iteration 결과 검증
```

## Project Structure

```
build-your-own-ai-agent/
├── README.md                    # 영어 메인 (gemini)
├── README.ko.md                 # 한국어 (gemini)
├── LICENSE                      # MIT
├── CONTRIBUTING.md              # 기여 가이드
├── progress.md                  # 진행 기록 (claude)
├── assets/
│   ├── banner.png
│   └── demos/
├── 01-simple-chatbot/
│   ├── README.md
│   ├── main.py
│   └── requirements.txt
├── 02-tool-calling-agent/
│   ├── README.md
│   ├── main.py
│   ├── tools.py
│   └── requirements.txt
├── ... (03-12 동일 패턴)
└── .github/workflows/test.yml
```

## Build Pipeline

### Phase 1: Foundation (Iteration 1-2)
- 리포 구조 초기화
- 프로젝트 01-02 구현 + README

### Phase 2: Core (Iteration 3-5)
- 프로젝트 03-06 구현 + README
- 메인 README 완성 (영문/한국어)

### Phase 3: Advanced (Iteration 6-8)
- 프로젝트 07-10 구현 + README
- CI/CD 설정

### Phase 4: Expert + Polish (Iteration 9-10)
- 프로젝트 11-12 구현
- CONTRIBUTING.md, LICENSE
- 최종 품질 검증

## Quality Standards

### Code (codex 담당)
- Python 3.10+ 호환
- 각 프로젝트 self-contained (공유 의존성 없음)
- `requirements.txt`에 버전 핀닝
- Ollama 로컬 실행 옵션 필수 제공
- 30분 이내 완료 가능한 분량
- 주석은 영어, docstring 포함

### Documentation (gemini 담당)
- 각 프로젝트 README: What/Why/How/Architecture
- 메인 README: 배너, 배지, 프로젝트 테이블, Star CTA
- 다국어: 영어 우선, 한국어 필수
- 아키텍처 다이어그램 (Mermaid or ASCII)

### Verification (claude 담당)
- 파일 존재 확인 (Glob)
- requirements.txt 파싱 가능 확인
- main.py 구문 오류 없음 확인 (python -m py_compile)
- README 링크/구조 확인

## Git Convention

```
feat(01): add simple chatbot project
feat(02): add tool-calling agent project
docs: add main README with banner and project table
docs(ko): add Korean README
ci: add GitHub Actions workflow
chore: update progress tracking
```

## omc-teams Task Template

### For codex (code implementation):
```
프로젝트 디렉토리: {project_dir}/
파일 목록: main.py, {extra_files}, requirements.txt

요구사항:
1. [구체적 기능 명세]
2. Ollama 로컬 + OpenAI API 모두 지원
3. python main.py로 즉시 실행 가능
4. requirements.txt에 모든 의존성 + 버전
5. 한 파일 200줄 이내
6. docstring 포함, 영어 주석
```

### For gemini (documentation):
```
프로젝트 디렉토리: {project_dir}/
파일: README.md

요구사항:
1. 제목 + 한 줄 설명
2. What You'll Learn 섹션
3. Architecture 다이어그램 (Mermaid)
4. Quick Start (3단계 이내)
5. How It Works 설명
6. Key Concepts 섹션
7. 영어로 작성
```

## Current Status

- Phase: Not Started
- Completed Projects: 0/12
- Next Action: Phase 1 시작

---
> Source: [Lay4U/build-your-own-ai-agent](https://github.com/Lay4U/build-your-own-ai-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
